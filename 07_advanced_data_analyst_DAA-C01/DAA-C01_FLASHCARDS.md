# DAA-C01 Flashcards

Front/back cards for spaced-repetition drilling. Front-loaded with the highest-density syntax: window function frames, QUALIFY, PIVOT/UNPIVOT, MATCH_RECOGNIZE, VARIANT navigation, FLATTEN parameters, and Cortex ML Function signatures. ~105 cards total.

Format: `**Q:**` then `**A:**`. Study by covering the answer with your hand or a card.

---

## Section A — Window Function Syntax (cards 1-20)

**Q1:** What is the full OVER clause skeleton for a window function?
**A1:** `function() OVER (PARTITION BY ... ORDER BY ... ROWS|RANGE BETWEEN <start> AND <end>)`.

**Q2:** What is the default window frame when ORDER BY is present but no explicit frame is specified?
**A2:** `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. (Not ROWS — critical distinction.)

**Q3:** What is the default frame when ORDER BY is omitted?
**A3:** The entire partition (`ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` effectively).

**Q4:** Difference between `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW` and `RANGE BETWEEN INTERVAL '2 DAYS' PRECEDING AND CURRENT ROW`?
**A4:** ROWS is positional (always 3 rows regardless of values). RANGE is value-based (all rows whose ORDER BY value falls in the interval; ties produce identical frames).

**Q5:** What does `LAG(col, 2, 0)` return?
**A5:** The value of `col` 2 rows prior in the partition, with default `0` instead of NULL when no prior row exists.

**Q6:** Running total syntax?
**A6:** `SUM(col) OVER (ORDER BY date ROWS UNBOUNDED PRECEDING)` or equivalently `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`.

**Q7:** 7-row trailing moving average syntax?
**A7:** `AVG(col) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` — 7 rows = current + 6 preceding.

**Q8:** Centered 7-row moving average syntax?
**A8:** `AVG(col) OVER (ORDER BY date ROWS BETWEEN 3 PRECEDING AND 3 FOLLOWING)`.

**Q9:** Difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` for the ordered values `100, 90, 90, 80`?
**A9:** ROW_NUMBER: 1,2,3,4. RANK: 1,2,2,4 (gap). DENSE_RANK: 1,2,2,3 (no gap).

**Q10:** Why does `LAST_VALUE(col) OVER (ORDER BY ts)` without an explicit frame return the current row's value?
**A10:** Default frame is `RANGE UNBOUNDED PRECEDING TO CURRENT ROW` — so "last" = current. Fix: specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`.

**Q11:** How do you get the first and last value across the entire partition?
**A11:** Add `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` to both window specs.

**Q12:** What does `NTILE(10) OVER (ORDER BY col)` do?
**A12:** Splits rows into 10 approximately-equal-sized buckets based on ordering; returns the bucket number 1-10 per row.

**Q13:** What range of values does `PERCENT_RANK()` return?
**A13:** [0.0, 1.0] inclusive. 0.0 for the smallest value in the partition, 1.0 for the largest. Formula: (rank - 1) / (total - 1).

**Q14:** What does `CUME_DIST()` return?
**A14:** Cumulative distribution: fraction of partition rows with value ≤ current (0 < val ≤ 1.0, inclusive). Tied rows share the maximum cum_dist.

**Q15:** What is `RATIO_TO_REPORT(col) OVER (PARTITION BY grp)` equivalent to?
**A15:** `col / SUM(col) OVER (PARTITION BY grp)` — row's share of its partition total.

**Q16:** What are `FIRST_VALUE`, `LAST_VALUE`, and `NTH_VALUE`?
**A16:** FIRST_VALUE: first row's value in the frame. LAST_VALUE: last row's value in the frame. NTH_VALUE(col, n): nth row's value (NULL if frame has fewer than n rows). `FROM FIRST`/`FROM LAST` controls direction.

**Q17:** Is `ORDER BY` required for `LAG`/`LEAD`?
**A17:** Yes — without ORDER BY, result is non-deterministic (Snowflake assigns arbitrary order).

**Q18:** How do you compute YoY change on dense monthly data with a window function?
**A18:** `LAG(rev, 12) OVER (ORDER BY month)` — jumps back 12 positions. Breaks on gaps.

**Q19:** Named window clause syntax?
**A19:** `SELECT ..., SUM(x) OVER w, AVG(x) OVER w FROM t WINDOW w AS (PARTITION BY grp ORDER BY ts ROWS 7 PRECEDING)`. Promotes reuse.

**Q20:** Can you reference a window alias in `WHERE`?
**A20:** No. Windows execute AFTER WHERE. Use `QUALIFY` or wrap in a subquery.

---

## Section B — QUALIFY and Execution Order (cards 21-28)

**Q21:** What does QUALIFY do?
**A21:** Filters rows based on window function output — same role as HAVING for aggregates. Snowflake-specific.

**Q22:** Logical SQL execution order?
**A22:** FROM/JOIN → WHERE → GROUP BY → HAVING → WINDOW → QUALIFY → SELECT → DISTINCT → ORDER BY → LIMIT.

**Q23:** When is QUALIFY valid?
**A23:** When the query contains at least one window function (in SELECT or QUALIFY itself).

**Q24:** QUALIFY equivalent of `SELECT * FROM (SELECT *, ROW_NUMBER() OVER(...) rn FROM t) WHERE rn=1`?
**A24:** `SELECT * FROM t QUALIFY ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...) = 1`.

**Q25:** Why does `SELECT ROW_NUMBER() OVER (...) AS rn FROM t WHERE rn = 1` fail?
**A25:** WHERE evaluates before window functions. The alias doesn't exist yet. Use QUALIFY.

**Q26:** Can QUALIFY be used alongside a GROUP BY?
**A26:** Yes, as long as a window function is present. QUALIFY runs after HAVING.

**Q27:** Does QUALIFY support referencing a SELECT-list alias?
**A27:** Yes — QUALIFY runs after SELECT alias resolution logically.

**Q28:** Top-N-per-group pattern: ties kept vs. ties dropped?
**A28:** `QUALIFY ROW_NUMBER() OVER (...) <= N` drops ties arbitrarily. `QUALIFY RANK() OVER (...) <= N` keeps all tied rows.

---

## Section C — Grouping Extensions (cards 29-34)

**Q29:** GROUPING SETS syntax to compute (a,b), (a), () groupings in one scan?
**A29:** `GROUP BY GROUPING SETS ((a,b),(a),())`.

**Q30:** ROLLUP(a,b,c) produces which grouping sets?
**A30:** `(a,b,c), (a,b), (a), ()` — hierarchical with grand total.

**Q31:** CUBE(a,b) produces which grouping sets?
**A31:** `(a,b), (a), (b), ()` — all 2^n combinations.

**Q32:** What does `GROUPING_ID(a,b)` return and how is it interpreted?
**A32:** Bit-encoded integer: bit=1 for each column aggregated to ALL. For (a,b): 0=grouped by both, 1=grouped by a only (b is subtotal), 2=grouped by b only, 3=grand total.

**Q33:** Why is `GROUPING_ID` critical with CUBE/ROLLUP?
**A33:** Distinguishes a real NULL in data from a NULL introduced by subtotaling.

**Q34:** Is `ROLLUP(a,b)` the same as `ROLLUP(b,a)`?
**A34:** No — order matters. ROLLUP produces hierarchical subtotals in the specified column order.

---

## Section D — PIVOT and UNPIVOT (cards 35-42)

**Q35:** Basic PIVOT syntax?
**A35:** `SELECT * FROM src PIVOT (SUM(amount) FOR category IN ('A','B','C'))`.

**Q36:** How do you alias pivoted column names?
**A36:** `PIVOT (...) AS p (orig_col, new_a, new_b, new_c)`.

**Q37:** Dynamic PIVOT syntax (2024+)?
**A37:** `PIVOT (SUM(amount) FOR category IN (ANY ORDER BY category))`.

**Q38:** What is the trap with "implicit grouping" in PIVOT?
**A38:** Any non-pivot, non-aggregate column in the inner SELECT becomes a grouping key. Project only the columns you need, or you'll get unexpected multiple rows.

**Q39:** Basic UNPIVOT syntax?
**A39:** `src UNPIVOT (metric_val FOR metric_name IN (col1, col2, col3))`.

**Q40:** By default, UNPIVOT drops rows where unpivoted values are NULL. How do you keep them?
**A40:** Use `INCLUDE NULLS`: `UNPIVOT INCLUDE NULLS (val FOR name IN (c1,c2,c3))`.

**Q41:** How are pivoted column names referenced downstream?
**A41:** As quoted string identifiers: `"BUILDING"` not `BUILDING`.

**Q42:** What data types must UNPIVOTed columns share?
**A42:** All columns in the IN list must have compatible data types (same type or implicitly convertible).

---

## Section E — MATCH_RECOGNIZE (cards 43-54)

**Q43:** Six required components of MATCH_RECOGNIZE?
**A43:** PARTITION BY, ORDER BY, MEASURES, ONE ROW PER MATCH or ALL ROWS PER MATCH, PATTERN, DEFINE.

**Q44:** MATCH_RECOGNIZE skeleton for session detection with 30-min gap?
**A44:**
```sql
MATCH_RECOGNIZE (
  PARTITION BY user_id
  ORDER BY ts
  MEASURES MATCH_NUMBER() session_id, FIRST(ts) start, LAST(ts) end_
  ONE ROW PER MATCH
  PATTERN (S+)
  DEFINE S AS DATEDIFF('minute', PREV(ts), ts) <= 30 OR PREV(ts) IS NULL
)
```

**Q45:** PATTERN quantifiers?
**A45:** `*` zero-or-more, `+` one-or-more, `?` optional, `{n}` exactly n, `{n,m}` range, `|` alternation, `()` grouping.

**Q46:** Difference between `ONE ROW PER MATCH` and `ALL ROWS PER MATCH`?
**A46:** ONE: single summary row per match (default). ALL: emit every matched row annotated with `MATCH_NUMBER()` and `CLASSIFIER()`.

**Q47:** How do you handle the first row (no PREV) in a MATCH_RECOGNIZE DEFINE clause?
**A47:** `DEFINE X AS <condition> OR PREV(ts) IS NULL`.

**Q48:** What is `AFTER MATCH SKIP` for?
**A48:** Controls where the matching engine resumes after a match: `PAST LAST ROW` (next unmatched row), `TO NEXT ROW` (immediately next row), `TO LAST var`, `TO FIRST var`.

**Q49:** `PREV(col)` and `NEXT(col)` inside MATCH_RECOGNIZE — what are they?
**A49:** Pattern navigation functions. They return the previous/next value in the pattern context (not window functions, not SQL `LAG`/`LEAD`).

**Q50:** What does `CLASSIFIER()` return?
**A50:** The name of the pattern variable (e.g., 'DOWN', 'UP') that matched the current row. Available in MEASURES and in ALL ROWS PER MATCH output.

**Q51:** What does `MATCH_NUMBER()` return?
**A51:** Integer identifying which match the row belongs to (1, 2, 3, ...). Great for sessions and run IDs.

**Q52:** Pattern for "3 or more consecutive increases"?
**A52:** `PATTERN (UP{3,})` with `DEFINE UP AS price > PREV(price)`.

**Q53:** How is MATCH_RECOGNIZE implemented under the hood?
**A53:** NFA (nondeterministic finite automaton). Processes the ordered partition in a single pass.

**Q54:** Can PATTERN use regex `|` for alternation?
**A54:** Yes: `PATTERN ((A B+) | (C D))` matches either sequence.

---

## Section F — VARIANT Access (cards 55-67)

**Q55:** Three equivalent ways to access `v.user.name`?
**A55:** `v:user.name`, `v:user:name`, `v['user']['name']`. All valid.

**Q56:** How do you access an array element?
**A56:** `v:events[0]` or `v['events'][0]`.

**Q57:** How do you access a key containing a hyphen (like `x-forwarded-for`)?
**A57:** Bracket notation: `v["x-forwarded-for"]` (dot/colon fails on special chars).

**Q58:** Why cast with `::INT` or `::STRING` after VARIANT navigation?
**A58:** For strongly typed comparisons, correct pruning behavior, and to avoid VARIANT-boolean returns. `v:x::INT = 5` prunes better than `v:x = 5`.

**Q59:** What are the five functions that detect VARIANT type?
**A59:** `IS_ARRAY`, `IS_OBJECT`, `IS_NULL_VALUE` (JSON null, not SQL NULL), `IS_INTEGER`/`IS_DOUBLE`/`IS_VARCHAR`/`IS_BOOLEAN`, `TYPEOF(v)` (returns string).

**Q60:** Difference between JSON null and SQL NULL?
**A60:** JSON null is a VARIANT value `null`. SQL NULL is the absence of value. `IS_NULL_VALUE(v)` returns TRUE for the former; `IS NULL` detects the latter.

**Q61:** How do you build a JSON object in SQL?
**A61:** `OBJECT_CONSTRUCT('key1', val1, 'key2', val2)` or `OBJECT_CONSTRUCT(*)` for current row.

**Q62:** Difference between `OBJECT_CONSTRUCT` and `OBJECT_CONSTRUCT_KEEP_NULL`?
**A62:** Default drops keys whose values are NULL. KEEP_NULL preserves them.

**Q63:** Three main OBJECT manipulation functions?
**A63:** `OBJECT_INSERT(obj, key, value [, update_flag])`, `OBJECT_DELETE(obj, key1[, k2...])`, `OBJECT_PICK(obj, key1[, k2...])`.

**Q64:** `OBJECT_KEYS(v)` returns what?
**A64:** VARIANT ARRAY of the top-level keys. Flatten again to iterate.

**Q65:** Three JSON parsing functions and their failure behavior?
**A65:** `PARSE_JSON` (errors), `TRY_PARSE_JSON` (returns NULL), `CHECK_JSON` (returns NULL if valid, error-message string if invalid).

**Q66:** How is `PARSE_JSON('null')` different from `PARSE_JSON(NULL)`?
**A66:** `PARSE_JSON('null')` returns VARIANT JSON null; `PARSE_JSON(NULL)` returns SQL NULL.

**Q67:** XML attribute vs. text content keys?
**A67:** Attributes under `@` (e.g., `v:"@sku"`); text content under `$` (e.g., `GET(xml, '$')`).

---

## Section G — FLATTEN Parameters (cards 68-77)

**Q68:** FLATTEN full named-parameter signature?
**A68:** `FLATTEN(input => <variant>, path => '<path>', outer => TRUE|FALSE, recursive => TRUE|FALSE, mode => 'OBJECT'|'ARRAY'|'BOTH')`.

**Q69:** What columns does FLATTEN emit?
**A69:** `SEQ`, `KEY`, `PATH`, `INDEX`, `VALUE`, `THIS`.

**Q70:** Why do you usually prefix FLATTEN with LATERAL?
**A70:** LATERAL lets the FLATTEN subquery see the outer row's columns (correlated). Without LATERAL, FLATTEN can't reference the parent's VARIANT column.

**Q71:** What does `outer => TRUE` do?
**A71:** Keeps the parent row even when the array/object is empty or NULL — LEFT JOIN semantics. Without it, parent rows with empty collections are dropped.

**Q72:** What does `recursive => TRUE` do?
**A72:** FLATTEN descends into nested arrays/objects at every level. Each nested leaf becomes a row.

**Q73:** What does `mode => 'ARRAY'` restrict?
**A73:** Flatten only the array parts; ignore objects. `mode => 'OBJECT'` is the mirror. Default `'BOTH'`.

**Q74:** What does `path => 'orders.items'` do?
**A74:** Starts FLATTEN at the nested path rather than the root — equivalent to `FLATTEN(input => v:orders.items)` but older syntax.

**Q75:** Pattern to flatten a nested array per row and keep rows with empty arrays?
**A75:** `FROM parent, LATERAL FLATTEN(input => parent.items, outer => TRUE) f`.

**Q76:** When should you NOT use `recursive => TRUE`?
**A76:** When documents are deeply nested and you only care about one level — recursive emits a row for every intermediate node and leaf, exploding output.

**Q77:** How do you aggregate over flattened array elements?
**A77:**
```sql
SELECT f.value:sku::STRING sku, SUM(f.value:qty::NUMBER) total
FROM orders, LATERAL FLATTEN(input => items) f
GROUP BY sku;
```

---

## Section H — Cortex ML Function Signatures (cards 78-93)

**Q78:** Cortex FORECAST creation signature?
**A78:**
```sql
CREATE SNOWFLAKE.ML.FORECAST model_name(
  INPUT_DATA => SYSTEM$REFERENCE('VIEW','my_view'),
  TIMESTAMP_COLNAME => 'ts',
  TARGET_COLNAME => 'y'
);
```

**Q79:** How do you call the forecaster to generate predictions?
**A79:** `CALL model_name!FORECAST(FORECASTING_PERIODS => 30, CONFIG_OBJECT => {'prediction_interval': 0.95});`

**Q80:** Required parameter to turn single-series into multi-series FORECAST?
**A80:** `SERIES_COLNAME => 'product_id'`. Internally trains one model per series.

**Q81:** What columns does FORECAST output?
**A81:** `TS`, `FORECAST`, `LOWER_BOUND`, `UPPER_BOUND` (and series key if multi-series). Lower/Upper come from `prediction_interval`.

**Q82:** How do you include exogenous variables in FORECAST?
**A82:** Add feature columns to the training view (they're used automatically). At forecast time, pass `INPUT_DATA => SYSTEM$REFERENCE('VIEW','future_exog')` containing those feature values for the horizon.

**Q83:** Cortex ANOMALY_DETECTION creation signature?
**A83:**
```sql
CREATE SNOWFLAKE.ML.ANOMALY_DETECTION ad_model(
  INPUT_DATA => SYSTEM$REFERENCE('VIEW','train'),
  TIMESTAMP_COLNAME => 'ts',
  TARGET_COLNAME => 'y'
);
```

**Q84:** How do you call the trained anomaly detector?
**A84:**
```sql
CALL ad_model!DETECT_ANOMALIES(
  INPUT_DATA => SYSTEM$REFERENCE('VIEW','new_data'),
  TIMESTAMP_COLNAME => 'ts',
  TARGET_COLNAME => 'y',
  CONFIG_OBJECT => {'prediction_interval': 0.95}
);
```

**Q85:** ANOMALY_DETECTION output columns?
**A85:** `TS`, `Y` (actual), `FORECAST` (expected), `LOWER_BOUND`, `UPPER_BOUND`, `IS_ANOMALY`, `DISTANCE`.

**Q86:** How do you make anomaly detection more sensitive (flag more anomalies)?
**A86:** Lower the `prediction_interval` (e.g., 0.80). Tighter bands = more flags. Higher (0.99) = fewer flags.

**Q87:** Cortex CLASSIFICATION creation signature?
**A87:**
```sql
CREATE SNOWFLAKE.ML.CLASSIFICATION model_name(
  INPUT_DATA => SYSTEM$REFERENCE('VIEW','train'),
  TARGET_COLNAME => 'label'
);
```

**Q88:** How do you call predict on a classification model?
**A88:** `SELECT model_name!PREDICT(INPUT_DATA => OBJECT_CONSTRUCT(*)) FROM features;`

**Q89:** CLASSIFICATION PREDICT output shape?
**A89:** VARIANT object with `class`, `probability` (per-class), `feature_importance` (per-row/local).

**Q90:** TOP_INSIGHTS signature?
**A90:**
```sql
CALL SNOWFLAKE.ML.TOP_INSIGHTS!GET_DRIVERS(
  INPUT_DATA => SYSTEM$REFERENCE('VIEW','v'),
  LABEL_COLNAME => 'is_high',
  METRIC_COLNAME => 'revenue'
);
```

**Q91:** CONTRIBUTION_EXPLORER signature?
**A91:**
```sql
CALL SNOWFLAKE.ML.CONTRIBUTION_EXPLORER!EXPLAIN_CHANGE(
  INPUT_DATA => SYSTEM$REFERENCE('VIEW','v'),
  METRIC_COLNAME => 'revenue',
  CATEGORICAL_DIMENSIONS => ['product','region'],
  BASELINE_FILTER => 'period = ''Q1''',
  COMPARISON_FILTER => 'period = ''Q2'''
);
```

**Q92:** `CONFIG_OBJECT` syntax — key quoting?
**A92:** `{'prediction_interval': 0.95}` — single quotes around keys, JSON-like structure.

**Q93:** What is the quickest way to interpret CLASSIFICATION's per-row explanations?
**A93:** `prediction:feature_importance` is a VARIANT object mapping feature name → importance for that specific row.

---

## Section I — Dashboards and Streamlit (cards 94-100)

**Q94:** Snowsight dashboard filter syntax in a tile query?
**A94:** `WHERE col = :filter_name` — Snowsight substitutes the current filter value at execution.

**Q95:** How do you build a cascading child filter dependent on `:region`?
**A95:** Child picklist query references `:region`:
```sql
SELECT DISTINCT N_NAME FROM NATION
JOIN REGION USING (R_REGIONKEY)
WHERE R_NAME = :region ORDER BY 1;
```

**Q96:** Streamlit-in-Snowflake connection pattern?
**A96:**
```python
from snowflake.snowpark.context import get_active_session
session = get_active_session()
```

**Q97:** Difference between `@st.cache_data` and `@st.cache_resource`?
**A97:** `cache_data` caches hashable serializable results (DataFrames). `cache_resource` caches non-serialized singletons (sessions, connections, models).

**Q98:** What is `st.session_state`?
**A98:** Dict-like store that survives widget-triggered reruns within one user session. Essential for multi-step wizard flows.

**Q99:** Scheduled dashboard refresh — whose role does it run under?
**A99:** The dashboard owner's role, not the viewer's. Row Access Policies apply to the owner's context.

**Q100:** How do you tag queries emitted by a dashboard for performance tracking?
**A100:** `ALTER SESSION SET QUERY_TAG='dash_name'` as a preamble. Then filter `QUERY_HISTORY` by `query_tag`.

---

## Section J — Data Prep and Sharing (cards 101-110)

**Q101:** View vs Materialized View vs Dynamic Table — which supports arbitrary joins and windows?
**A101:** View (yes, no storage) and Dynamic Table (yes, with storage and declarative refresh). MV has strict shape restrictions: no joins (except simple), no window functions, no non-deterministic functions.

**Q102:** Dynamic Table minimum TARGET_LAG?
**A102:** 1 minute.

**Q103:** What does `TARGET_LAG = DOWNSTREAM` mean?
**A103:** Refresh only when a downstream Dynamic Table needs this upstream data — avoids wasted work.

**Q104:** Why create a SECURE VIEW rather than a regular VIEW?
**A104:** Hides the view's SQL definition from consumers. Required for views exposed through secure data shares. Also disables some pushdown optimizations.

**Q105:** Zero-copy clone syntax?
**A105:** `CREATE TABLE t2 CLONE t1;` or `CREATE TABLE t2 CLONE t1 AT (TIMESTAMP => '...')` for point-in-time.

**Q106:** Query Profile metrics most important to analysts?
**A106:** `partitions_scanned / partitions_total` (pruning efficiency), bytes_scanned, spill to local/remote storage, join row counts, compilation time.

**Q107:** Micro-partition pruning requirement for a `WHERE date_col = '...'` filter?
**A107:** The predicate must be on the raw column, not a function-wrapped version. `DATE_TRUNC('day', date_col) = ...` prevents direct min/max pruning on `date_col`.

**Q108:** Clustering key vs. Search Optimization Service?
**A108:** Clustering optimizes range scans (date ranges). SOS optimizes point lookups (equality, IN). Different access patterns.

**Q109:** Scale up vs. scale out?
**A109:** Scale up = larger warehouse size → single-query speed. Scale out = multi-cluster → concurrent query capacity. Not interchangeable.

**Q110:** Share creation (producer side)?
**A110:**
```sql
CREATE SHARE s;
GRANT USAGE ON DATABASE d TO SHARE s;
GRANT USAGE ON SCHEMA d.sch TO SHARE s;
GRANT SELECT ON ALL TABLES IN SCHEMA d.sch TO SHARE s;
ALTER SHARE s ADD ACCOUNTS = ('consumer_acct');
```

---

## Section K — Approximate Functions and Stats (cards 111-120)

**Q111:** `APPROX_COUNT_DISTINCT` under the hood?
**A111:** HyperLogLog sketch; ~16 KB per group; ~1.6% relative error; mergeable via `HLL_COMBINE`.

**Q112:** `APPROX_TOP_K(col, k, counters)` — what does `counters` control?
**A112:** Size of the SpaceSaving sketch. Must be ≥ k. 10-100× k gives good accuracy. Lower → faster but less accurate.

**Q113:** `APPROX_PERCENTILE_ACCUMULATE/ESTIMATE/COMBINE`?
**A113:** ACCUMULATE builds a t-digest sketch per group. ESTIMATE extracts percentile from a sketch. COMBINE merges sketches for incremental percentile tracking.

**Q114:** `PERCENTILE_CONT` vs `PERCENTILE_DISC`?
**A114:** CONT interpolates linearly between adjacent values (returns value that may not exist in data). DISC returns the first actual value whose cumulative distribution ≥ p. MEDIAN = PERCENTILE_CONT(0.5).

**Q115:** `REGR_SLOPE(y, x)` argument order?
**A115:** y FIRST, x SECOND — opposite of mathematical convention. Always double-check.

**Q116:** `SKEW` interpretation?
**A116:** 0 = symmetric, > 0 = right-tailed (positive skew), < 0 = left-tailed.

**Q117:** `KURTOSIS` interpretation in Snowflake?
**A117:** Returns EXCESS kurtosis (normal distribution = 0, not 3). Positive = heavier tails than normal.

**Q118:** `MODE(col)` returns what on ties?
**A118:** One arbitrary mode (not all tied values, not NULL). For all tied modes, use `GROUP BY col ... QUALIFY RANK() OVER (ORDER BY COUNT(*) DESC) = 1`.

**Q119:** `WIDTH_BUCKET(expr, min, max, buckets)` — what happens for values outside [min, max]?
**A119:** Returns 0 for values below min, `buckets + 1` for values above max. Never NULL.

**Q120:** How is `BOOLOR_AGG(expr)` different from `MAX(IFF(expr, 1, 0)) = 1`?
**A120:** `BOOLOR_AGG` is a native boolean aggregate returning BOOLEAN. The IFF version returns INT=1 or 0. Semantically identical for non-NULL inputs; BOOLOR_AGG is clearer.

---

## Review Tips

- Cycle through sections, not top-to-bottom. Each section covers one syntactic family.
- Rate each card confident / fuzzy / unknown. Drill unknowns daily; fuzzies every 2 days; confidents weekly.
- When a card feels "memorized," hide the Q, write the A from scratch on paper — if you can produce it fresh, it's truly learned.
- 20 cards per 30-minute session is a healthy pace for retention.
