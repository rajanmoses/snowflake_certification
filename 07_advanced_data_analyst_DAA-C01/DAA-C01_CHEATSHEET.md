# DAA-C01 Cheatsheet

Dense syntactic reference. Open this during mock exam review and whenever you need to confirm an argument order, parameter name, or default behavior. Covers everything that commonly appears as a one-liner trap on the exam.

---

## 1. Window Function Syntax

### Full OVER clause

```sql
function() OVER (
  [PARTITION BY expr_list]
  [ORDER BY expr_list [ASC|DESC] [NULLS FIRST|LAST]]
  [ROWS|RANGE BETWEEN <frame_start> AND <frame_end>]
)
```

### Frame boundary options

| Keyword | Meaning |
|---------|---------|
| `UNBOUNDED PRECEDING` | start of partition |
| `n PRECEDING` | n rows (ROWS) or n units (RANGE) before current |
| `CURRENT ROW` | current row |
| `n FOLLOWING` | n rows/units after current |
| `UNBOUNDED FOLLOWING` | end of partition |

### Default frames (CRITICAL)

| Clause state | Default frame |
|--------------|---------------|
| No `ORDER BY` | `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` (whole partition) |
| `ORDER BY` present, no explicit frame | `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` |

The default being RANGE (not ROWS) is the single most trap-worthy fact in this domain. `LAST_VALUE` without an explicit frame returns the current row's value.

### ROWS vs RANGE

| | ROWS | RANGE |
|-|------|-------|
| Counting basis | position (physical rows) | ORDER BY expression value |
| With ties | ties processed separately | ties produce identical frames |
| Interval support | integers only (`2 PRECEDING`) | interval literals (`INTERVAL '7 DAYS' PRECEDING`) |
| Use for | moving averages / rolling counts | time-aware running totals; dense/sparse-safe windows |

### Named windows

```sql
SELECT
  SUM(amt) OVER w,
  AVG(amt) OVER w
FROM t
WINDOW w AS (PARTITION BY cust ORDER BY ts ROWS 6 PRECEDING);
```

---

## 2. Common Analytical Patterns

### Running total

```sql
SUM(amt) OVER (ORDER BY ts ROWS UNBOUNDED PRECEDING)
```

### Moving average (trailing 7 rows)

```sql
AVG(amt) OVER (ORDER BY ts ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
```

### Centered moving average (7 rows)

```sql
AVG(amt) OVER (ORDER BY ts ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING)
```

### Rank with ties (keep ties, gaps allowed)

```sql
RANK() OVER (PARTITION BY grp ORDER BY val DESC)
```

### Dense rank (keep ties, no gaps)

```sql
DENSE_RANK() OVER (PARTITION BY grp ORDER BY val DESC)
```

### Row number (arbitrary tie-breaking)

```sql
ROW_NUMBER() OVER (PARTITION BY grp ORDER BY val DESC)
```

### Percent rank [0, 1]

```sql
PERCENT_RANK() OVER (ORDER BY val)
-- (rank - 1) / (total_rows - 1)
```

### Cume dist (fraction ≤ current)

```sql
CUME_DIST() OVER (ORDER BY val)
```

### LAG / LEAD with default

```sql
LAG(col, 1, 0) OVER (PARTITION BY cust ORDER BY ts)  -- 1 row back, default 0
LEAD(col, 7)   OVER (PARTITION BY cust ORDER BY ts)  -- 7 rows forward
```

### YoY change (dense monthly data)

```sql
rev - LAG(rev, 12) OVER (ORDER BY month)
-- Breaks on gaps; use calendar join if gaps possible.
```

### Gaps and islands (consecutive dates per user)

```sql
SELECT user_id, MIN(dt) run_start, MAX(dt) run_end, COUNT(*) run_length
FROM (
  SELECT user_id, dt,
    DATEADD('day', -ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY dt), dt) AS grp
  FROM activity)
GROUP BY user_id, grp;
```

### Sessionization (30-min gap)

```sql
SELECT * FROM clicks
MATCH_RECOGNIZE (
  PARTITION BY user_id ORDER BY ts
  MEASURES MATCH_NUMBER() session_id, FIRST(ts) start_, LAST(ts) end_, COUNT(*) n
  ONE ROW PER MATCH
  PATTERN (S+)
  DEFINE S AS DATEDIFF('minute', PREV(ts), ts) <= 30 OR PREV(ts) IS NULL
);
```

### Row-level z-score

```sql
(val - AVG(val) OVER (PARTITION BY grp))
/ NULLIF(STDDEV(val) OVER (PARTITION BY grp), 0)
```

### Ratio to group total

```sql
RATIO_TO_REPORT(amt) OVER (PARTITION BY cust)
-- equivalent to: amt / SUM(amt) OVER (PARTITION BY cust)
```

### First/last in partition (correct frame!)

```sql
FIRST_VALUE(col) OVER (
  PARTITION BY grp ORDER BY ts
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING)
```

### Top N per group (ties kept)

```sql
SELECT * FROM t
QUALIFY RANK() OVER (PARTITION BY grp ORDER BY val DESC) <= 3;
```

---

## 3. QUALIFY vs Subquery

### QUALIFY (Snowflake-idiomatic)

```sql
SELECT * FROM orders
QUALIFY ROW_NUMBER() OVER (PARTITION BY cust ORDER BY ts DESC) = 1;
```

### Subquery equivalent (standard SQL)

```sql
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY cust ORDER BY ts DESC) rn
  FROM orders
) WHERE rn = 1;
```

### Rules

- QUALIFY requires at least one window function in the query (in SELECT or in the QUALIFY clause itself).
- Logical execution: FROM → WHERE → GROUP BY → HAVING → WINDOW → **QUALIFY** → SELECT → DISTINCT → ORDER BY → LIMIT.
- QUALIFY can reference SELECT-list aliases (lateral-ish alias resolution).

---

## 4. GROUPING SETS / ROLLUP / CUBE

### Explicit grouping sets

```sql
GROUP BY GROUPING SETS ((a,b), (a), (b), ())
-- 4 grouping levels in one scan
```

### ROLLUP — hierarchical subtotals

```sql
GROUP BY ROLLUP(year, quarter, month)
-- Produces: (year,quarter,month), (year,quarter), (year), ()
-- n+1 groupings for n columns
```

### CUBE — all combinations

```sql
GROUP BY CUBE(product, region)
-- Produces: (product,region), (product), (region), ()
-- 2^n groupings for n columns
```

### GROUPING_ID — distinguish real NULL from subtotal NULL

```sql
SELECT
  COALESCE(product, 'ALL') product,
  COALESCE(region,  'ALL') region,
  SUM(amount),
  GROUPING_ID(product, region) grp_id,
  CASE GROUPING_ID(product, region)
    WHEN 0 THEN 'leaf'
    WHEN 1 THEN 'product subtotal'    -- region aggregated-to-all
    WHEN 2 THEN 'region subtotal'     -- product aggregated-to-all
    WHEN 3 THEN 'grand total'
  END level
FROM sales
GROUP BY CUBE(product, region);
```

Bit encoding (leftmost column = highest bit): bit = 1 means that column is aggregated to ALL in this grouping.

---

## 5. PIVOT / UNPIVOT

### PIVOT (long → wide)

```sql
SELECT * FROM src
PIVOT (SUM(amount) FOR category IN ('A','B','C'))
AS p (orig_col, a_sum, b_sum, c_sum);
```

### Dynamic PIVOT (2024+)

```sql
PIVOT (SUM(amount) FOR category IN (ANY ORDER BY category))
-- or
PIVOT (SUM(amount) FOR category IN (<subquery>))
```

### UNPIVOT (wide → long)

```sql
SELECT * FROM src
UNPIVOT [INCLUDE NULLS] (metric_val FOR metric_name IN (col1, col2, col3));
```

### Gotchas

- Pivoted column names: quoted uppercase strings. Reference as `"A"`, not `A`.
- Non-pivot, non-aggregate columns in the inner SELECT become implicit grouping keys — select only what you need.
- UNPIVOT default drops NULL rows; add `INCLUDE NULLS` to retain.

---

## 6. MATCH_RECOGNIZE Skeleton

```sql
SELECT * FROM src
MATCH_RECOGNIZE (
  PARTITION BY <partition_cols>
  ORDER BY <sort_cols>
  MEASURES
    MATCH_NUMBER()      AS match_id,
    CLASSIFIER()        AS classifier,
    FIRST(col)          AS first_val,
    LAST(col)           AS last_val,
    COUNT(*)            AS n_rows
  { ONE ROW PER MATCH | ALL ROWS PER MATCH [SHOW EMPTY MATCHES] }
  [AFTER MATCH SKIP {PAST LAST ROW | TO NEXT ROW | TO LAST var | TO FIRST var}]
  PATTERN ( <pattern_expression> )
  DEFINE
    var1 AS <boolean condition>,
    var2 AS <boolean condition>
);
```

### Pattern operators

| Operator | Meaning |
|----------|---------|
| `A B C`  | sequence |
| `A*`     | zero or more |
| `A+`     | one or more |
| `A?`     | zero or one |
| `A{3}`   | exactly 3 |
| `A{3,5}` | 3 to 5 |
| `A{3,}`  | 3 or more |
| `A|B`    | alternation |
| `(A B)`  | grouping |

### Navigation functions (pattern context)

- `PREV(col [, n])` — previous row in pattern.
- `NEXT(col [, n])` — next row.
- `FIRST(col [, n])` — first row of current match.
- `LAST(col [, n])` — last row.

### Example — 3+ consecutive price ups

```sql
MATCH_RECOGNIZE (
  PARTITION BY symbol ORDER BY dt
  MEASURES MATCH_NUMBER() mn, FIRST(dt) start_dt, LAST(dt) end_dt, COUNT(*) run_len
  ONE ROW PER MATCH
  PATTERN (UP{3,})
  DEFINE UP AS price > PREV(price)
)
```

---

## 7. VARIANT / OBJECT / ARRAY Access

### Navigation operators

| Syntax | Example | Use |
|--------|---------|-----|
| `col:field` | `v:user` | top-level field |
| `col:field.sub` | `v:user.name` | nested object (dot) |
| `col:field:sub` | `v:user:name` | nested object (colon — equivalent) |
| `col:arr[0]` | `v:events[0]` | array element by index |
| `col['key']` | `v['x-y-z']` | special characters in key |
| `col:field::TYPE` | `v:id::INT` | cast after navigation |

### Type-check functions

```sql
IS_ARRAY(v)        -- TRUE if VARIANT holds array
IS_OBJECT(v)       -- TRUE if VARIANT holds object
IS_NULL_VALUE(v)   -- TRUE if VARIANT holds JSON null (not SQL NULL)
IS_INTEGER(v) / IS_DOUBLE(v) / IS_VARCHAR(v) / IS_BOOLEAN(v)
TYPEOF(v)          -- returns 'ARRAY', 'OBJECT', 'INTEGER', ...
```

### JSON parsing

```sql
PARSE_JSON('...')       -- strict, errors on invalid
TRY_PARSE_JSON('...')   -- NULL on invalid
CHECK_JSON('...')       -- NULL if valid, error-message string if invalid
```

### OBJECT manipulation

```sql
OBJECT_CONSTRUCT('a',1,'b',2)                 -- {"a":1,"b":2}; NULL keys dropped
OBJECT_CONSTRUCT_KEEP_NULL('a',NULL,'b',2)    -- retains NULL-valued keys
OBJECT_INSERT(obj, 'c', 3, TRUE)              -- adds/updates key
OBJECT_DELETE(obj, 'a', 'b')                  -- removes keys
OBJECT_PICK(obj, 'a', 'b')                    -- keeps only listed keys
OBJECT_KEYS(obj)                              -- VARIANT array of top-level keys
```

### ARRAY functions

```sql
ARRAY_CONSTRUCT(1,2,3)
ARRAY_SIZE(a)                 -- length
ARRAY_AGG(col) WITHIN GROUP (ORDER BY ...)
ARRAY_CAT(a, b)               -- concatenate
ARRAY_SLICE(a, from, to)      -- [from, to) 0-indexed
ARRAY_CONTAINS(v::VARIANT, a) -- membership (NOTE: cast search value)
ARRAY_POSITION(v::VARIANT, a) -- index of first match
ARRAY_DISTINCT(a)             -- dedupe
ARRAY_SORT(a)                 -- ascending
```

### FLATTEN parameters

```sql
FLATTEN(
  input      => <variant>,
  path       => '<nested.path>',          -- optional, start at nested location
  outer      => TRUE|FALSE,               -- FALSE default; TRUE keeps parent on empty
  recursive  => TRUE|FALSE,               -- FALSE default; TRUE walks all levels
  mode       => 'OBJECT'|'ARRAY'|'BOTH'   -- BOTH default
)
```

### FLATTEN output columns

| Column | Meaning |
|--------|---------|
| `SEQ`   | global sequence number |
| `KEY`   | object key (NULL for array elements) |
| `PATH`  | string path to this leaf |
| `INDEX` | array position (NULL for object entries) |
| `VALUE` | the element value |
| `THIS`  | the original VARIANT input |

### LATERAL + FLATTEN

```sql
SELECT parent.id, f.index, f.value:sku::STRING
FROM parent, LATERAL FLATTEN(input => parent.items) f;
-- LATERAL lets f see parent.items (correlated per row).
```

### XML parsing

```sql
PARSE_XML('<root>...</root>')
XMLGET(xml, 'tagname' [, instance_index])    -- returns VARIANT
GET(XMLGET(xml, 'tag'), '$')::TYPE           -- extract text content
v:"@attribute_name"::STRING                  -- attribute access
v:"$"::STRING                                -- text content access
```

---

## 8. Cortex ML Function Signatures

### FORECAST

```sql
-- Create model
CREATE OR REPLACE SNOWFLAKE.ML.FORECAST model_name(
  INPUT_DATA         => SYSTEM$REFERENCE('VIEW', 'training_view'),
  SERIES_COLNAME     => '<optional, for multi-series>',
  TIMESTAMP_COLNAME  => 'ts',
  TARGET_COLNAME     => 'y'
);

-- Generate forecasts
CALL model_name!FORECAST(
  FORECASTING_PERIODS => 30,
  CONFIG_OBJECT       => {'prediction_interval': 0.95}
);

-- With exogenous future values
CALL model_name!FORECAST(
  INPUT_DATA          => SYSTEM$REFERENCE('VIEW', 'future_exog'),
  FORECASTING_PERIODS => 30
);
```

Output columns: `TS`, `FORECAST`, `LOWER_BOUND`, `UPPER_BOUND` (plus series key if multi-series).

### ANOMALY_DETECTION

```sql
-- Create model
CREATE OR REPLACE SNOWFLAKE.ML.ANOMALY_DETECTION model_name(
  INPUT_DATA         => SYSTEM$REFERENCE('VIEW', 'training_view'),
  SERIES_COLNAME     => '<optional>',
  TIMESTAMP_COLNAME  => 'ts',
  TARGET_COLNAME     => 'y'
);

-- Score new data
CALL model_name!DETECT_ANOMALIES(
  INPUT_DATA         => SYSTEM$REFERENCE('VIEW', 'scoring_view'),
  TIMESTAMP_COLNAME  => 'ts',
  TARGET_COLNAME     => 'y',
  CONFIG_OBJECT      => {'prediction_interval': 0.95}
);
```

Output columns: `TS`, `Y`, `FORECAST`, `LOWER_BOUND`, `UPPER_BOUND`, `IS_ANOMALY`, `DISTANCE`.

Higher `prediction_interval` = wider band = fewer anomalies flagged.

### CLASSIFICATION

```sql
-- Create model
CREATE OR REPLACE SNOWFLAKE.ML.CLASSIFICATION model_name(
  INPUT_DATA     => SYSTEM$REFERENCE('VIEW', 'training_view'),
  TARGET_COLNAME => 'label'
);

-- Predict
SELECT
  id,
  model_name!PREDICT(INPUT_DATA => OBJECT_CONSTRUCT(*)) AS p,
  p:class::STRING                       AS predicted_class,
  p:probability.churned::NUMBER         AS churn_probability,
  p:feature_importance                  AS per_row_importance
FROM features_view;
```

### TOP_INSIGHTS

```sql
CALL SNOWFLAKE.ML.TOP_INSIGHTS!GET_DRIVERS(
  INPUT_DATA      => SYSTEM$REFERENCE('VIEW','v'),
  LABEL_COLNAME   => 'is_high_value',
  METRIC_COLNAME  => 'revenue'
);
```

Finds "interesting subgroups" where the metric differs from the overall average.

### CONTRIBUTION_EXPLORER

```sql
CALL SNOWFLAKE.ML.CONTRIBUTION_EXPLORER!EXPLAIN_CHANGE(
  INPUT_DATA             => SYSTEM$REFERENCE('VIEW','v'),
  METRIC_COLNAME         => 'revenue',
  CATEGORICAL_DIMENSIONS => ['product','region','channel'],
  BASELINE_FILTER        => 'period = ''2024-Q1''',
  COMPARISON_FILTER      => 'period = ''2024-Q2'''
);
```

Decomposes metric change between two filtered subsets by dimension contribution.

### CONFIG_OBJECT examples

```sql
CONFIG_OBJECT => {'prediction_interval': 0.95}                              -- default
CONFIG_OBJECT => {'prediction_interval': 0.80}                              -- tighter (more sensitive)
CONFIG_OBJECT => {'prediction_interval': 0.99}                              -- looser (less sensitive)
CONFIG_OBJECT => {'prediction_interval': 0.95, 'on_error': 'skip'}          -- skip errored series
```

### Cortex AI vs ML

| Function | Family | Use |
|----------|--------|-----|
| `SNOWFLAKE.ML.FORECAST` | ML | Time-series forecasting |
| `SNOWFLAKE.ML.ANOMALY_DETECTION` | ML | Unsupervised outlier flagging |
| `SNOWFLAKE.ML.CLASSIFICATION` | ML | Supervised classification |
| `SNOWFLAKE.ML.TOP_INSIGHTS` | ML | Rule mining |
| `SNOWFLAKE.ML.CONTRIBUTION_EXPLORER` | ML | Driver attribution |
| `SNOWFLAKE.CORTEX.COMPLETE` | LLM | Text generation |
| `SNOWFLAKE.CORTEX.SUMMARIZE` | LLM | Summarization |
| `SNOWFLAKE.CORTEX.AI_CLASSIFY` | LLM | Zero-shot classification |
| `SNOWFLAKE.CORTEX.PARSE_DOCUMENT` | LLM | PDF/image OCR+layout |

---

## 9. Snowsight Dashboard Syntax

### Filter reference in tile queries

```sql
SELECT * FROM orders
WHERE order_date BETWEEN :date_from AND :date_to
  AND segment = :segment
  AND status IN (:status_multi);
```

### Filter types

| Type | Produces | Usage |
|------|----------|-------|
| Date range | `:date_from`, `:date_to` | `BETWEEN :date_from AND :date_to` |
| Picklist (single) | scalar | `col = :filter` |
| Picklist (multi) | array | `col IN (:filter)` |
| Text | scalar string | `col = :filter` |

### Picklist dynamic options query

```sql
-- Picklist filter query must return exactly ONE column
SELECT DISTINCT C_MKTSEGMENT FROM customers ORDER BY 1;
```

### Cascading filter (child depends on parent)

```sql
-- Child filter query references parent filter :region
SELECT DISTINCT nation
FROM nations JOIN regions USING (region_key)
WHERE region = :region
  OR :region IS NULL   -- handle unset parent
ORDER BY 1;
```

### Scheduling

- Minimum schedule interval: 1 minute (varies).
- Executes as the dashboard **owner's** role.
- Email delivery (PDF/CSV) is optional.
- Query tag is set via session preamble: `ALTER SESSION SET QUERY_TAG = 'dash_<name>'`.

### Filter rules

- Filter names are **case-sensitive**: `:Segment` ≠ `:segment`.
- Date values are strings; cast with `::DATE` for safety.
- Multi-select filters produce arrays; use `IN (:name)`.
- Cannot inject filter values into DDL (SELECT only).

---

## 10. Streamlit in Snowflake Essentials

### Session acquisition

```python
# In Snowsight-hosted Streamlit (SiS)
from snowflake.snowpark.context import get_active_session
session = get_active_session()

# External/local
from snowflake.snowpark import Session
session = Session.builder.configs({
    "account":"...","user":"...","password":"...",
    "role":"ANALYST","warehouse":"WH","database":"DB","schema":"PUBLIC"
}).create()
```

### Caching decorators

```python
import streamlit as st

@st.cache_data(ttl=300, max_entries=50)   # cached serializable results
def load_orders(segment: str):
    return session.sql(f"SELECT * FROM orders WHERE seg='{segment}'").to_pandas()

@st.cache_resource                        # cached non-serializable singletons
def get_session():
    return get_active_session()
```

| Decorator | Caches | Example |
|-----------|--------|---------|
| `@st.cache_data` | serializable results (DataFrames, dicts, strings) | query results |
| `@st.cache_resource` | non-serializable singletons | sessions, models |

### Session state

```python
# Dict-like; survives widget-triggered re-runs
if 'step' not in st.session_state:
    st.session_state.step = 1

st.session_state.user_segment = 'BUILDING'
st.rerun()  # forces immediate re-execution
```

### Common widgets

```python
st.selectbox("Segment", ["A","B","C"])
st.multiselect(...)
st.slider("Threshold", 0, 100, 50)
st.date_input("From", date(2024,1,1))
st.text_input(...)
st.button("Run")
st.file_uploader("Upload CSV")        # ~200 MB limit
```

### Chart options

```python
st.line_chart(df, x="month", y="revenue")
st.bar_chart(df)
st.scatter_chart(df)
st.dataframe(df)                       # tabular; max ~50k rows cleanly
st.plotly_chart(fig, use_container_width=True)
st.altair_chart(chart)
```

### Rendering tips

- `.limit(N).to_pandas()` before display — never ship full tables.
- `cache_data` the query function to avoid re-runs on every widget change.
- `st.rerun()` after modifying session state to apply changes immediately.

### Limits (as of 2024-2026)

- Package allowlist (Anaconda channel).
- No `subprocess`, no outbound HTTP without External Access Integration + Secret.
- One session per user per app.
- File upload limit ~200 MB.

---

## 11. View vs MV vs Dynamic Table Decision Table

**View**
- Storage: None (SQL only)
- Refresh: On every query (full recompute)
- Supports joins: Yes (any)
- Supports window functions: Yes
- Supports non-deterministic funcs: Yes
- Lag control: n/a (always current)
- Cost model: Query-time compute
- Minimum lag: 0 (live)
- Use case: Reusable logic, lightweight transforms

**Materialized View**
- Storage: Pre-computed, auto-maintained
- Refresh: Automatic on base-table changes
- Supports joins: Only simple joins (sometimes)
- Supports window functions: No
- Supports non-deterministic funcs: No
- Lag control: Fully automatic
- Cost model: Background maintenance credits
- Minimum lag: ~seconds
- Use case: Hot aggregations with simple shape

**Dynamic Table**
- Storage: Pre-computed, auto-maintained
- Refresh: Declarative via TARGET_LAG
- Supports joins: Yes (any)
- Supports window functions: Yes
- Supports non-deterministic funcs: Yes
- Lag control: `TARGET_LAG = '5 minutes'` or `DOWNSTREAM`
- Cost model: Background refresh credits (dedicated warehouse)
- Minimum lag: 1 minute
- Use case: Multi-step ELT pipelines, declarative analyst tables

### Decision tree

```
Do you need the data stored?
├── NO → VIEW
│
└── YES. Is it a simple single-table aggregation with no window functions, no joins beyond basic dimension?
    ├── YES. Is cost-of-maintenance worth it?
    │    ├── YES → MATERIALIZED VIEW
    │    └── NO  → VIEW with explicit CTAS snapshots
    │
    └── NO → DYNAMIC TABLE with TARGET_LAG
```

### Dynamic Table syntax

```sql
CREATE OR REPLACE DYNAMIC TABLE dt_name
  TARGET_LAG = '5 minutes'   -- or 'DOWNSTREAM'
  WAREHOUSE  = analytics_wh
  AS
SELECT ... FROM base;

-- Monitor refresh
SELECT * FROM TABLE(INFORMATION_SCHEMA.DYNAMIC_TABLE_REFRESH_HISTORY(
  NAME => 'DT_NAME'
));
```

### Materialized View restrictions (common ones)

- No `JOIN` (except specific dimension-like simple cases).
- No window functions (`OVER`).
- No non-deterministic functions (`CURRENT_TIMESTAMP`, `UUID_STRING`, `RANDOM`).
- No ORDER BY except within aggregates.
- No LIMIT / OFFSET.
- No sub-queries that cannot be flattened.

---

## 12. Micro-Partition Pruning Quick Reference

### Pruning works

- `WHERE date_col = '2024-01-15'`
- `WHERE date_col BETWEEN '2024-01-01' AND '2024-01-31'`
- `WHERE date_col >= '2024-01-15' AND date_col < '2024-01-16'`
- `WHERE id IN (1, 5, 100)`
- `WHERE data:region::STRING = 'US'` (shred-aware)

### Pruning blocked

- `WHERE DATE_TRUNC('day', date_col) = '2024-01-15'`  — function wrap
- `WHERE date_col = '2024-01-15'::VARCHAR` — type mismatch
- `WHERE SUBSTR(name, 1, 3) = 'FOO'` — function wrap
- `WHERE data:region = 'US'` (no cast) — VARIANT-boolean comparison
- `WHERE IS_ARRAY(data:tags)` — predicate on VARIANT shape

### Fixes

- Rewrite function-wrapped predicates: `date_col >= '2024-01-15' AND date_col < '2024-01-16'`.
- Add explicit casts on VARIANT paths: `::STRING`, `::INT`, `::TIMESTAMP`.
- Materialize hot paths to typed columns via a Dynamic Table.
- Add clustering key on common filter columns.

---

## 13. Approximate Functions

- `APPROX_COUNT_DISTINCT(col)` → integer estimate; memory ~16KB; use: cardinality on large groups
- `HLL(col)` → sketch (binary); memory ~16KB; use: mergeable count-distinct
- `HLL_COMBINE(sketch)` → merged sketch; use: rollup across sketches
- `HLL_ESTIMATE(sketch)` → integer; use: extract count from sketch
- `APPROX_TOP_K(col, k, counters)` → ARRAY of {value, estimate}; memory ~counters × 32B; use: top-k heavy hitters
- `APPROX_PERCENTILE(col, p)` → numeric; memory small; use: one-shot percentile
- `APPROX_PERCENTILE_ACCUMULATE(col)` → sketch; use: per-group sketch
- `APPROX_PERCENTILE_ESTIMATE(sk, p)` → numeric; use: extract from sketch
- `APPROX_PERCENTILE_COMBINE(sketches)` → merged sketch; use: roll up sketches

Rules of thumb:
- Use approximate when cardinality is in the millions+ and 1-2% error is acceptable.
- `counters` in `APPROX_TOP_K` should be 10-100× k.
- Sketch-based approximate functions enable incremental percentile tracking without full re-scan.

---

## 14. Statistical SQL Functions

### Centrality

```sql
AVG(col) / MEAN(col)                  -- arithmetic mean (identical in Snowflake)
MEDIAN(col)                           -- = PERCENTILE_CONT(0.5)
MODE(col)                             -- most frequent value (ties → arbitrary)
```

### Spread

```sql
STDDEV(col) / STDDEV_SAMP(col)        -- sample stddev (default)
STDDEV_POP(col)                       -- population stddev
VAR_SAMP(col) / VARIANCE(col)         -- sample variance
VAR_POP(col)                          -- population variance
```

### Shape

```sql
SKEW(col)                             -- 0 symmetric; >0 right-tail; <0 left-tail
KURTOSIS(col)                         -- EXCESS kurtosis (normal = 0, not 3)
```

### Percentile

```sql
PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY col)   -- interpolated (continuous)
PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY col)   -- first value with cum >= p (discrete)
```

### Association

```sql
CORR(x, y)                            -- Pearson r in [-1, 1]
COVAR_SAMP(x, y) / COVAR_POP(x, y)    -- covariance
```

### Regression

```sql
REGR_SLOPE(y, x)                      -- y first, x second (reversed from math!)
REGR_INTERCEPT(y, x)
REGR_R2(y, x)
REGR_COUNT(y, x)                      -- rows with both non-null
REGR_SXX(y, x)                        -- sum of (x - avg(x))^2
REGR_SYY(y, x)
REGR_SXY(y, x)
```

### Boolean aggregates

```sql
BOOLOR_AGG(expr)                      -- TRUE if any TRUE
BOOLAND_AGG(expr)                     -- TRUE only if all TRUE
BOOLXOR_AGG(expr)                     -- TRUE if exactly one TRUE
```

### Count variants

```sql
COUNT(*)                              -- all rows including NULL columns
COUNT(col)                            -- non-NULL rows
COUNT(DISTINCT col)                   -- exact (expensive on large N)
COUNT_IF(predicate)                   -- Snowflake shortcut
```

### Confidence interval (95%, mean)

```sql
WITH s AS (
  SELECT COUNT(*) n, AVG(x) mu, STDDEV(x) sd FROM data)
SELECT mu, mu - 1.96*sd/SQRT(n) lo, mu + 1.96*sd/SQRT(n) hi
FROM s;
```

### Two-sample t-test (Welch's)

```sql
WITH g AS (SELECT grp, AVG(v) mu, STDDEV(v) sd, COUNT(*) n FROM t GROUP BY grp)
SELECT
  (SELECT mu FROM g WHERE grp='A') - (SELECT mu FROM g WHERE grp='B') AS mean_diff,
  ((SELECT mu FROM g WHERE grp='A') - (SELECT mu FROM g WHERE grp='B'))
  / SQRT(
    POWER((SELECT sd FROM g WHERE grp='A'),2) / (SELECT n FROM g WHERE grp='A')
    + POWER((SELECT sd FROM g WHERE grp='B'),2) / (SELECT n FROM g WHERE grp='B')
  ) AS t_stat;
```

### Confusion matrix

```sql
SELECT
  COUNT_IF(actual=1 AND predicted=1)              AS tp,
  COUNT_IF(actual=0 AND predicted=1)              AS fp,
  COUNT_IF(actual=1 AND predicted=0)              AS fn,
  COUNT_IF(actual=0 AND predicted=0)              AS tn,
  tp*1.0 / NULLIF(tp+fp, 0)                       AS precision,
  tp*1.0 / NULLIF(tp+fn, 0)                       AS recall,
  2.0*tp / NULLIF(2*tp+fp+fn, 0)                  AS f1
FROM predictions;
```

---

## 15. Discovery and Profiling

### Metadata commands (never scan data)

```sql
SHOW TABLES LIKE '%customer%' IN SCHEMA db.schema;
DESCRIBE TABLE t;
GET_DDL('TABLE', 'db.schema.t');
SELECT * FROM TABLE(RESULT_SCAN(LAST_QUERY_ID()));   -- post-process SHOW output
```

### INFORMATION_SCHEMA (real-time, 7-day retention)

```sql
INFORMATION_SCHEMA.TABLES          -- row_count accurate for permanent/transient
INFORMATION_SCHEMA.COLUMNS
INFORMATION_SCHEMA.VIEWS
INFORMATION_SCHEMA.QUERY_HISTORY_BY_SESSION()
```

### ACCOUNT_USAGE (365-day retention, 45m-3h lag, Enterprise+)

```sql
SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY
SNOWFLAKE.ACCOUNT_USAGE.OBJECT_DEPENDENCIES
SNOWFLAKE.ACCOUNT_USAGE.TAG_REFERENCES
SNOWFLAKE.ACCOUNT_USAGE.TABLE_STORAGE_METRICS
SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTIONS_USAGE_HISTORY
```

### Query profiling

```sql
ALTER SESSION SET QUERY_TAG = 'my_dash';
-- run queries
SELECT query_text, execution_time, bytes_scanned,
       partitions_scanned, partitions_total
FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE query_tag = 'my_dash'
ORDER BY execution_time DESC;
```

---

## 16. Data Sharing Quick Reference

### Producer side (create share)

```sql
CREATE SHARE my_share;
GRANT USAGE ON DATABASE prod TO SHARE my_share;
GRANT USAGE ON SCHEMA prod.analytics TO SHARE my_share;
GRANT SELECT ON ALL TABLES IN SCHEMA prod.analytics TO SHARE my_share;
GRANT SELECT ON VIEW prod.analytics.secure_v TO SHARE my_share;
ALTER SHARE my_share ADD ACCOUNTS = ('consumer_acct_id');
```

### Consumer side (mount share)

```sql
SHOW SHARES;
CREATE DATABASE shared_prod FROM SHARE producer_acct.my_share;
SELECT * FROM shared_prod.analytics.table_x;
```

### Zero-copy clone

```sql
CREATE TABLE t2 CLONE t1;
CREATE TABLE t_hist CLONE t1 AT (TIMESTAMP => '2024-01-15 10:00');
CREATE TABLE t_ago CLONE t1 AT (OFFSET => -3600);
CREATE SCHEMA s2 CLONE s1;
```

### Time Travel recovery

```sql
SELECT * FROM t AT (OFFSET => -300);                          -- 5 min ago
SELECT * FROM t AT (TIMESTAMP => '2024-01-15 10:00:00');
SELECT * FROM t BEFORE (STATEMENT => '<query_id>');

-- Restore
CREATE OR REPLACE TABLE t CLONE t AT (OFFSET => -300);
```

Retention: Standard = 1 day; Enterprise+ = up to 90. Fail-safe = 7 days (admin-only).

### Marketplace

- Browse → Request → Accept → Shared DB appears in your account.
- Read-only. No CREATE in shared DB.
- Provider's row/column security applies.
- Consumer pays warehouse credits only; some listings have additional data charges.

---

## 17. Warehouse Sizing

| Size | Credits/hour | Server count | Best for |
|------|--------------|--------------|----------|
| XS | 1 | 1 | Small queries, dashboards |
| S | 2 | 2 | Small-medium analytics |
| M | 4 | 4 | Standard ad-hoc |
| L | 8 | 8 | Large aggregations |
| XL | 16 | 16 | TB-scale queries |
| 2XL | 32 | 32 | Complex multi-TB joins |
| 3XL+ | 64, 128, 256, 512 | 64+ | Extreme-scale, rare |

### Scale UP vs OUT

| | Scale UP (bigger size) | Scale OUT (multi-cluster) |
|-|------------------------|----------------------------|
| Fixes | slow single queries | query queueing / concurrency |
| Mechanism | more cores/memory per cluster | more parallel clusters |
| Config | `WAREHOUSE_SIZE = 'LARGE'` | `MIN/MAX_CLUSTER_COUNT`, `SCALING_POLICY` |
| Credit impact | linear with size | linear with active clusters |

### Multi-cluster syntax

```sql
ALTER WAREHOUSE analytics_wh SET
  WAREHOUSE_SIZE     = 'LARGE'
  MIN_CLUSTER_COUNT  = 1
  MAX_CLUSTER_COUNT  = 4
  SCALING_POLICY     = 'STANDARD';   -- or 'ECONOMY'
```

### Query Acceleration Service

```sql
ALTER WAREHOUSE analytics_wh SET
  ENABLE_QUERY_ACCELERATION           = TRUE
  QUERY_ACCELERATION_MAX_SCALE_FACTOR = 8;
```

Serverless compute augments the warehouse for queries with large scans — pay-as-used.

---

## 18. Clustering and Search Optimization

### Clustering key

```sql
ALTER TABLE orders CLUSTER BY (order_date);
SELECT SYSTEM$CLUSTERING_INFORMATION('ORDERS', '(ORDER_DATE)');
```

- Automatic clustering runs in background (serverless credits).
- One clustering key per table (can be multi-column).
- Optimizes **range** scans.

### Search Optimization Service

```sql
ALTER TABLE events ADD SEARCH OPTIMIZATION ON EQUALITY(user_id, event_id);
```

- Optimizes **point** lookups (equality, IN).
- Minimum effective table size: ~2 TB range.

### Choosing between them

| Query shape | Choose |
|-------------|--------|
| `WHERE date BETWEEN ...` | Clustering |
| `WHERE id = 42` | SOS |
| `WHERE email LIKE 'a%'` | SOS (SUBSTRING equality) |
| `WHERE date > ... AND segment IN (...)` | Clustering on date, maybe SOS on segment |

---

## 19. Query Profile Checklist

When a query is slow, check the Profile for:

1. **Partition pruning ratio.** `partitions_scanned / partitions_total` — aim for low (<10% on filtered queries).
2. **Spill to disk.** Local spill is bad; remote spill is 100× worse. Fix by scaling up.
3. **Bytes exchange between operators.** Large cross-stage movement signals exploding joins.
4. **TableScan with high bytes_scanned and low rows_produced.** Missing predicates or no pruning.
5. **Join row explosion.** Compare input vs. output rows per join operator — many-to-many joins blow up.
6. **Compilation time.** Spikes if the query has 100+ joins or ambiguous aliases.
7. **Queue time.** Warehouse under-provisioned for concurrency — scale out.

---

## 20. Final Trap List

Memorize these — they are the most common exam traps.

1. Default window frame with ORDER BY is **RANGE**, not ROWS. `LAST_VALUE` without explicit frame = current row.
2. `REGR_SLOPE(y, x)` — y first, x second (opposite of math convention).
3. `KURTOSIS` in Snowflake = EXCESS kurtosis (normal = 0, not 3).
4. `COUNT_IF` is an aggregate, NOT a window function.
5. QUALIFY requires a window function in the query.
6. WHERE cannot reference window function aliases; use QUALIFY.
7. FLATTEN parameters are named: `input`, `path`, `outer`, `recursive`, `mode`.
8. `outer => TRUE` keeps parent on empty/NULL (LEFT JOIN semantics).
9. `OBJECT_CONSTRUCT` drops NULL-valued keys; use `_KEEP_NULL` to retain.
10. `ARRAY_CONTAINS(v::VARIANT, arr)` — must cast search value to VARIANT.
11. Pivoted column names are quoted strings: `"BUILDING"` not `BUILDING`.
12. UNPIVOT drops all-NULL rows by default; use `INCLUDE NULLS`.
13. `GROUPING_ID` bit ordering: leftmost column = high-order bit.
14. MATCH_RECOGNIZE requires: PARTITION BY, ORDER BY, MEASURES, ONE/ALL ROWS, PATTERN, DEFINE.
15. `AFTER MATCH SKIP PAST LAST ROW` resumes at row after match's last.
16. Cortex FORECAST uses `SYSTEM$REFERENCE('VIEW','name')`, not a bare name.
17. `CONFIG_OBJECT => {'prediction_interval': 0.95}` — single quotes around keys.
18. Higher `prediction_interval` = wider band = fewer anomalies flagged.
19. CLASSIFICATION `!PREDICT` returns VARIANT: `class`, `probability`, `feature_importance`.
20. Materialized Views don't support joins or window functions — use Dynamic Tables.
21. Dynamic Table minimum TARGET_LAG = 1 minute.
22. Scheduled dashboards run under the owner's role, not the viewer's.
23. Filter names in Snowsight are case-sensitive: `:Segment` ≠ `:segment`.
24. `get_active_session()` from `snowflake.snowpark.context`.
25. `@st.cache_resource` for sessions and non-serializable singletons; `@st.cache_data` for DataFrames.
26. Function-wrapped predicates block micro-partition pruning.
27. VARIANT path with explicit cast enables shred-aware pruning.
28. Secure views required for secure data sharing.
29. Reader accounts are legacy; private listings are the modern pattern.
30. ACCOUNT_USAGE lags 45m-3h; use INFORMATION_SCHEMA for real-time.
