# Snowflake Advanced Data Analyst (DAA-C01) — Study Pack Index

## Certification Overview

**Exam Code:** DAA-C01
**Full Title:** SnowPro Advanced: Data Analyst
**Audience:** Analysts who use Snowflake SQL (and lightly, Python/Streamlit) to profile, transform, visualize, forecast, and share data. Assumes SnowPro Core-level comfort with warehouses, roles, time travel, and ingestion.
**Format:** ~65 multiple-choice / multi-select questions, 115 minutes, proctored.
**Passing score:** scaled; aim for 75%+ on the mock here to feel comfortable.
**Prerequisites:** SnowPro Core certification is strongly recommended (not strictly required for DAA, but assumed knowledge).
**Study time:** 40-60 hours if you already use Snowflake daily; 80-120 hours otherwise.

This pack covers the exam's five technical blueprint domains plus analyst-focused data preparation and sharing. The emphasis is heavily SQL-first — expect to read, debug, and author queries involving window functions, QUALIFY, PIVOT, MATCH_RECOGNIZE, FLATTEN, and the Cortex ML Function family.

---

## Blueprint Weights (as of the current DAA-C01 syllabus)

| Domain | Weight | Pack Module |
|--------|--------|-------------|
| Data Analytics Fundamentals | 15-20% | `01_data_analytics_fundamentals_QA.md` |
| Advanced SQL for Analytics | 25-30% | `02_advanced_sql_analytics_QA.md` |
| Semi-Structured and Unstructured Data | 15-20% | `03_semi_structured_and_unstructured_QA.md` |
| Data Visualization and Apps | 15-20% | `04_data_visualization_and_apps_QA.md` |
| Statistical and ML Analytics | 20-25% | `05_statistical_and_ml_analytics_QA.md` |
| Data Preparation and Sharing | 15-20% | `06_data_preparation_and_sharing_QA.md` |

Weights sum to >100% because Snowflake lists overlapping sub-topics across modules; treat these as emphasis signals rather than exact proportions. The two biggest-weight domains (Advanced SQL and Cortex ML) are collectively ~50% of the exam — spend proportional time there.

---

## Module Summaries

### 01 — Data Analytics Fundamentals
Workflow discipline: discovery (`SHOW/DESCRIBE/GET_DDL`), profiling (`INFORMATION_SCHEMA`, `ACCOUNT_USAGE`), quality dimensions, descriptive stats (`PERCENTILE_CONT/DISC`, `CORR`, `COVAR`, `SKEW`, `KURTOSIS`), outlier methods (z-score, IQR, MAD), sampling (`BERNOULLI` vs `SYSTEM`), approximate functions (`APPROX_COUNT_DISTINCT`, `APPROX_TOP_K`, `APPROX_PERCENTILE`), null semantics, and the fact/dimension distinction. The "when to use which function" domain.

### 02 — Advanced SQL for Analytics (highest weight)
Window functions in depth: ranking (`ROW_NUMBER/RANK/DENSE_RANK/PERCENT_RANK`), offset (`LAG/LEAD`), aggregation with frames (`ROWS` vs `RANGE BETWEEN`), navigation (`FIRST_VALUE/LAST_VALUE/NTH_VALUE`), `RATIO_TO_REPORT`, `NTILE`, `CUME_DIST`. Grouping extensions (`GROUPING SETS/ROLLUP/CUBE/GROUPING_ID`). `PIVOT/UNPIVOT` syntax. `QUALIFY` as a filter on window results. `MATCH_RECOGNIZE` for sessionization, run detection, funnel analytics. The gaps-and-islands pattern. Aggregates: `BOOLOR_AGG/BOOLAND_AGG`, `LISTAGG`, `ARRAY_AGG`, `OBJECT_AGG`.

### 03 — Semi-Structured and Unstructured
`VARIANT/OBJECT/ARRAY` type system and shredded storage. Access syntax: colon-dot, bracket, cast-after-navigate. `FLATTEN` with `OUTER/RECURSIVE/MODE/PATH` parameters paired with `LATERAL`. XML via `PARSE_XML`/`XMLGET`. Parquet/JSON/Avro from stages. Directory tables for unstructured files, `GET_PRESIGNED_URL`, `BUILD_SCOPED_FILE_URL`. Cortex AI functions on files (`AI_CLASSIFY`, `PARSE_DOCUMENT`). `OBJECT_INSERT/DELETE/PICK/KEYS`, `ARRAY_*` family, `TYPEOF/IS_ARRAY/IS_OBJECT`. External tables vs. directory tables.

### 04 — Data Visualization and Apps
Snowsight chart types (bar, line, scatter, scorecard, heatgrid). Dashboard filters (`:param` substitution), picklists, cascading filters, date ranges. Scheduled refresh and sharing. Streamlit in Snowflake: `get_active_session()`, `st.session_state`, `@st.cache_data`/`@st.cache_resource`, Snowpark DataFrame rendering, Plotly integration. Native Apps. Row Access Policies and Masking in a dashboard context. Query tagging for dashboard performance tracking.

### 05 — Statistical and ML Analytics
Cortex ML Functions: `FORECAST`, `ANOMALY_DETECTION`, `CLASSIFICATION`, `TOP_INSIGHTS`, `CONTRIBUTION_EXPLORER`. Signatures (`SYSTEM$REFERENCE`, `TIMESTAMP_COLNAME`, `TARGET_COLNAME`, `SERIES_COLNAME`), `CONFIG_OBJECT` tuning (`prediction_interval`), multi-series forecasting, exogenous variables, output interpretation (`FORECAST/LOWER_BOUND/UPPER_BOUND/IS_ANOMALY`). Snowpark ML modeling. Statistical SQL (`REGR_*`, `CORR`, `COVAR`, `APPROX_PERCENTILE_ACCUMULATE/COMBINE/ESTIMATE`). Model productionization via Dynamic Tables + Tasks. Confusion matrix in SQL. AI_CLASSIFY vs CLASSIFICATION.

### 06 — Data Preparation and Sharing
View vs. Materialized View vs. Dynamic Table decision tree. `TARGET_LAG` and `DOWNSTREAM`. Secure views. Zero-copy clones. Time Travel for recovery. Marketplace, private listings, data clean rooms. Secure Data Sharing producer/consumer. Query Profile reading. Micro-partition pruning. Clustering keys and `SYSTEM$CLUSTERING_INFORMATION`. Search Optimization Service. Warehouse sizing (scale up vs scale out). Query Acceleration Service. `COPY INTO` vs `INSERT INTO SELECT`. Tags and comments.

---

## Suggested Study Order

Order below reflects dependency and highest-weight-first for the second half:

1. **Week 1 — Fundamentals (Module 01).** Build muscle memory for `INFORMATION_SCHEMA` queries, approximate functions, and NULL semantics. These assumptions underlie every other module.
2. **Week 2-3 — Advanced SQL (Module 02).** The heaviest module. Spend extra time on `MATCH_RECOGNIZE` and window frames (`ROWS` vs `RANGE`) — these are the hardest syntax to reconstruct from memory.
3. **Week 4 — Semi/Unstructured (Module 03).** Focus on FLATTEN parameters and `LATERAL` — exam questions commonly ask you to pick the right FLATTEN flag combo.
4. **Week 5 — Statistical and ML (Module 05).** Memorize Cortex ML signatures. Build a few `FORECAST` / `ANOMALY_DETECTION` / `CLASSIFICATION` models in a sandbox so the parameter names feel natural.
5. **Week 6 — Data Prep and Sharing (Module 06).** View/MV/DT decision table and Query Profile interpretation. These show up as scenario questions.
6. **Week 7 — Visualization and Apps (Module 04).** Lightest conceptual load; blitz through dashboards and Streamlit specifics. Memorize filter syntax (`:param`), caching decorators, and `get_active_session`.
7. **Week 8 — Review week.** Work the mock exam, revisit flashcards, retry weak items. Re-read the cheatsheet daily.

---

## Companion Documents in This Pack

- **`README.md`** (this file) — orientation and study plan.
- **`01_data_analytics_fundamentals_QA.md`** through **`06_data_preparation_and_sharing_QA.md`** — long-form Q&A per domain. Primary learning material.
- **`DAA-C01_FLASHCARDS.md`** — ~100 front/back cards for spaced-repetition drilling. Front-loaded with window function and FLATTEN syntax and Cortex ML signatures.
- **`DAA-C01_MOCK_EXAM.md`** — 65 multi-domain questions in the exam's style, with answer key and explanations.
- **`DAA-C01_CHEATSHEET.md`** — dense syntax reference. Keep next to you while working the mocks.

---

## Tips for a SQL-Heavy Analyst Exam

- **Memorize syntax, not just intent.** The exam will show you SQL fragments and ask what the output is or whether it's correct. Reading SQL is different from writing it.
- **Default frames bite.** When ORDER BY is present but no frame is specified, the frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This changes the meaning of `LAST_VALUE`, `MAX`, `SUM` over ordered windows. The exam loves this trap.
- **`QUALIFY` requires a window function in the query.** Not just in the QUALIFY clause — anywhere in SELECT, WHERE equivalents, etc. If a question has no window, QUALIFY is wrong.
- **FLATTEN parameters are named arguments.** `FLATTEN(input => arr, outer => TRUE, recursive => FALSE, path => 'a.b', mode => 'ARRAY')`. Know the four modifier flags cold.
- **Cortex ML signatures use `SYSTEM$REFERENCE`.** It is NOT just a view name; it is `SYSTEM$REFERENCE('VIEW','my_view')` or `SYSTEM$REFERENCE('TABLE','my_table')`.
- **`CONFIG_OBJECT` is a JSON-ish OBJECT literal.** `{'prediction_interval': 0.95}` with single quotes around keys.
- **Pruning requires un-wrapped predicates.** `WHERE date_col = '2024-01-01'` prunes; `WHERE DATE_TRUNC('day', date_col) = '2024-01-01'` does not.
- **Dashboard filters are case-sensitive.** `:segment` != `:Segment`.
- **PIVOT column names come out quoted.** Use `"BUILDING"` not `BUILDING` to reference them downstream.
- **MATCH_RECOGNIZE has 6 required parts** (minus optional): `PARTITION BY`, `ORDER BY`, `MEASURES`, `ONE/ALL ROWS PER MATCH`, `PATTERN`, `DEFINE`. Missing any of the required ones is a syntax error.
- **Three truths of NULL:** aggregates skip NULLs; `NULL = NULL` is NULL; `NOT IN` with any NULL in the subquery returns NULL (not FALSE). The exam tests all three.
- **Scale up vs. scale out.** Scale up (bigger warehouse) = faster single query. Scale out (multi-cluster) = more concurrent queries. Mixing these up on a scenario question is a common wrong answer.
- **Dynamic Tables are the "modern view that stores data."** When a question offers View / MV / DT, remember: MVs have strict query shape restrictions (no joins, no windows). DTs accept arbitrary SQL. Views don't store data.
- **Reader accounts are legacy.** Private listings and secure shares supersede them for most new scenarios.
- **Use the mock questions to calibrate pacing.** You have ~105 seconds per question in the real exam. If a mock question takes longer than 2 minutes to settle, mark it as a weak spot.
- **Build it in a sandbox.** For every unfamiliar function, run at least one query in a free Snowflake trial. Reading syntax is 50% retention; typing it is 90%.
