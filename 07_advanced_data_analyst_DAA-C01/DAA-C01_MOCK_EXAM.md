# DAA-C01 Mock Exam

65 questions covering all DAA-C01 blueprint domains. Timed target: 115 minutes (same as real exam, ~105 seconds per question).

**Format:** Choose the single best answer unless marked `[multi-select]`. Answers and explanations follow in the answer key at the bottom.

---

## Questions

### Section 1 — Data Analytics Fundamentals (Q1-10)

**Q1.** You need to profile a 10 TB table and get a representative sample for quick exploration. Which clause produces the lowest-latency sample at the cost of some bias on partition-correlated columns?

A. `SAMPLE BERNOULLI (1)`
B. `SAMPLE SYSTEM (1)`
C. `TABLESAMPLE (1 ROWS)`
D. `LIMIT 10000`

---

**Q2.** `APPROX_COUNT_DISTINCT(user_id)` returns 50,000,003 on a table where the true distinct count is 50,000,000. Why does the function tolerate this error?

A. HyperLogLog's memory is constant (~16KB), so it cannot store every distinct value exactly.
B. The function samples a fraction of rows and extrapolates.
C. The function reads only micro-partition metadata.
D. Approximation only applies to numeric columns.

---

**Q3.** You run `COUNT_IF(email IS NULL OR TRIM(email) = '') / COUNT(*)::FLOAT` to measure which data quality dimension?

A. Uniqueness
B. Accuracy
C. Completeness
D. Timeliness

---

**Q4.** `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY val)` on the sorted set {10, 20} returns what?

A. 10
B. 15
C. 20
D. NULL

---

**Q5.** You find that `KURTOSIS(amount)` returns 4.2 on your dataset. Which Snowflake-specific interpretation is correct?

A. Heavier tails than a normal distribution (which Snowflake defines as kurtosis = 0).
B. Lighter tails than a normal distribution.
C. Right-skewed asymmetry.
D. Perfect symmetry.

---

**Q6.** Which approach prevents partition pruning when filtering on a DATE column named `order_date`?

A. `WHERE order_date = '2024-01-15'`
B. `WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31'`
C. `WHERE DATE_TRUNC('day', order_date) = '2024-01-15'`
D. `WHERE order_date >= '2024-01-15' AND order_date < '2024-01-16'`

---

**Q7.** `APPROX_TOP_K(col, 5, 10)` is suspected of returning inaccurate counts. Which single adjustment would most improve accuracy?

A. Increase the first argument (k).
B. Increase the third argument (counters), keeping it well above k.
C. Switch to `COUNT(DISTINCT col)`.
D. Add a `GROUP BY col` to the query.

---

**Q8.** You want to retrieve `INFORMATION_SCHEMA.TABLES.ROW_COUNT` for live profiling. Which statement is TRUE?

A. ROW_COUNT is accurate for views.
B. ROW_COUNT is maintained metadata for permanent and transient tables.
C. ROW_COUNT lags by up to 3 hours.
D. ROW_COUNT requires a full table scan to compute.

---

**Q9.** Which function returns a VARIANT containing the TYPE string of a VARIANT value?

A. `TYPEOF(v)`
B. `IS_VARIANT(v)`
C. `GET_TYPE(v)`
D. `CAST(v AS TEXT)`

---

**Q10.** You run `SELECT AVG(revenue) OVER ()` and `SELECT AVG(revenue) FROM t`. On a non-null column of 1000 rows, how do the outputs differ?

A. First returns 1 row; second returns 1000 rows.
B. First returns 1000 rows (each with the overall average); second returns 1 row with the average.
C. Outputs are identical.
D. First errors because `OVER ()` requires PARTITION BY.

---

### Section 2 — Advanced SQL Analytics (Q11-25)

**Q11.** The following query returns unexpected results — `last_order_date` equals `current_order_date` for every row:
```sql
SELECT
  customer_id,
  order_date AS current_order_date,
  LAST_VALUE(order_date) OVER (
    PARTITION BY customer_id ORDER BY order_date
  ) AS last_order_date
FROM orders;
```
Why?

A. `LAST_VALUE` is not a Snowflake-supported function.
B. The default window frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, so "last" = current row.
C. `LAST_VALUE` requires a `QUALIFY` clause.
D. `PARTITION BY customer_id` forces each partition to have only one row.

---

**Q12.** Which clause correctly selects "the single most recent order per customer, keeping ties"?

A. `QUALIFY ROW_NUMBER() OVER (PARTITION BY c ORDER BY d DESC) = 1`
B. `QUALIFY RANK() OVER (PARTITION BY c ORDER BY d DESC) = 1`
C. `QUALIFY DENSE_RANK() OVER (PARTITION BY c ORDER BY d DESC) = 1`
D. `WHERE ROW_NUMBER() OVER (PARTITION BY c ORDER BY d DESC) = 1`

---

**Q13.** Given the following, how many groupings are produced?
```sql
SELECT a, b, c, SUM(amount)
FROM t
GROUP BY CUBE(a, b, c);
```

A. 3
B. 4
C. 7
D. 8

---

**Q14.** Choose the correct MATCH_RECOGNIZE pattern that detects 3 or more consecutive price increases:

A. `PATTERN (UP{3})`
B. `PATTERN (UP{3,})`
C. `PATTERN (UP*)`
D. `PATTERN (UP+)`

---

**Q15.** In a MATCH_RECOGNIZE clause, what does the `DEFINE` clause specify?

A. Which rows to partition by.
B. Which measures (output columns) to emit.
C. The boolean condition for each pattern variable.
D. The ordering of rows within a partition.

---

**Q16.** You pivot a long-format table into wide format:
```sql
SELECT * FROM src
PIVOT (SUM(amount) FOR category IN ('A','B','C'));
```
Downstream, you reference the pivoted column A. Which syntax is correct?

A. `A`
B. `[A]`
C. `"A"`
D. `` `A` ``

---

**Q17.** Which Snowflake-only clause lets you filter on a window function result without a subquery?

A. `HAVING`
B. `QUALIFY`
C. `WINDOW FILTER`
D. `WHERE`

---

**Q18.** The query `SELECT COUNT_IF(status = 'active') OVER (PARTITION BY segment) FROM customers` fails. Why?

A. `COUNT_IF` is not valid in Snowflake.
B. `OVER` requires an `ORDER BY`.
C. `COUNT_IF` is not a window function in Snowflake.
D. `PARTITION BY segment` requires an index.

---

**Q19.** You need to compute a 30-day moving average on a daily revenue table. Which frame is correct?

A. `ROWS BETWEEN 30 PRECEDING AND CURRENT ROW`
B. `ROWS BETWEEN 29 PRECEDING AND CURRENT ROW`
C. `ROWS BETWEEN UNBOUNDED PRECEDING AND 30 FOLLOWING`
D. `RANGE INTERVAL '30 DAYS' FOLLOWING`

---

**Q20.** Gaps and islands on consecutive dates per user — which SQL pattern groups rows into islands using the `date - row_number` trick?

A. `DATEADD('day', ROW_NUMBER() OVER (PARTITION BY user ORDER BY dt), dt)`
B. `DATEADD('day', -ROW_NUMBER() OVER (PARTITION BY user ORDER BY dt), dt)`
C. `dt - ROW_NUMBER() OVER (ORDER BY dt)`
D. `MATCH_RECOGNIZE (ORDER BY dt PATTERN (ISLAND+))`

---

**Q21.** You unpivot wide monthly data:
```sql
SELECT * FROM src UNPIVOT (sales FOR month IN (jan, feb, mar));
```
Rows where all of jan/feb/mar are NULL are dropped. How do you keep them?

A. Use `OUTER UNPIVOT`.
B. Use `UNPIVOT INCLUDE NULLS`.
C. Add `WHERE sales IS NOT NULL`.
D. Switch to PIVOT.

---

**Q22.** What is `GROUPING_ID(a, b)` equal to when a row represents the "grouped by a only, b is a subtotal" level?

A. 0
B. 1
C. 2
D. 3

---

**Q23.** `LISTAGG(product_name, ', ')` returns `"Error: output exceeds 16 MB"`. Which alternative preserves the full list?

A. `MAX(product_name)`
B. `COUNT(product_name)`
C. `ARRAY_AGG(product_name)`
D. `OBJECT_AGG(product_name, NULL)`

---

**Q24.** `AFTER MATCH SKIP PAST LAST ROW` in a MATCH_RECOGNIZE does what?

A. Discards the entire partition on first match.
B. Resumes matching at the row immediately after the last row of the current match.
C. Skips any row with a NULL ORDER BY value.
D. Matches only the last row in each partition.

---

**Q25.** Which `OVER` configuration is most appropriate for "customer order value as a share of their total spend"?

A. `OVER (ORDER BY order_date ROWS UNBOUNDED PRECEDING)`
B. `OVER (PARTITION BY customer_id)`
C. `OVER ()`
D. `OVER (PARTITION BY customer_id ORDER BY order_date)`

---

### Section 3 — Semi-Structured and Unstructured (Q26-35)

**Q26.** You want to flatten an array `v:items` and keep the parent row even when the array is empty. Which FLATTEN flag is required?

A. `recursive => TRUE`
B. `mode => 'ARRAY'`
C. `outer => TRUE`
D. `path => 'items'`

---

**Q27.** Which syntax correctly navigates to the first element of a nested array and casts to INT?

A. `v.events[0].id::INT`
B. `v:events[0].id::INT`
C. `v::events[0]::id::INT`
D. `v['events']<0>:id::INT`

---

**Q28.** `TYPEOF(data:tags)` returns `'VARCHAR'` on some rows and `'ARRAY'` on others. Which strategy normalizes this to always be an array?

A. `data:tags::ARRAY`
B. `IFF(IS_ARRAY(data:tags), data:tags, ARRAY_CONSTRUCT(data:tags))`
C. `OBJECT_CONSTRUCT(data:tags)`
D. `TRY_CAST(data:tags AS ARRAY)`

---

**Q29.** You have a directory table on a stage of PDF files. Which function returns a time-limited HTTP URL for a file?

A. `GET_URL(stage, path)`
B. `GET_PRESIGNED_URL(stage, relative_path, expiration_seconds)`
C. `TO_FILE(stage, path)`
D. `BUILD_SCOPED_FILE_URL(stage, relative_path)`

---

**Q30.** `OBJECT_CONSTRUCT('name', NULL, 'age', 30)` returns:

A. `{"name": null, "age": 30}`
B. `{"age": 30}`
C. NULL
D. Error

---

**Q31.** Which function parses a JSON string and returns NULL on invalid input instead of raising an error?

A. `PARSE_JSON`
B. `TRY_PARSE_JSON`
C. `CHECK_JSON`
D. `TO_JSON`

---

**Q32.** You want to query a Parquet file in a stage and get a typed table without ingestion. Which object do you create?

A. Materialized view
B. Dynamic Table
C. External table
D. Directory table

---

**Q33.** The query `SELECT * FROM events WHERE data:region = 'US'` scans the full table despite indexing. Which single change is most likely to enable partition pruning on this VARIANT field?

A. Add a clustering key on `data:region`.
B. Add an explicit cast: `WHERE data:region::STRING = 'US'`.
C. Use `WHERE IS_OBJECT(data:region)`.
D. Convert to a MATERIALIZED VIEW.

---

**Q34.** Which FLATTEN output column gives you the array index of each element?

A. `KEY`
B. `PATH`
C. `INDEX`
D. `SEQ`

---

**Q35.** `OBJECT_PICK(o, 'a', 'b')` on an OBJECT lacking key `'b'` does what?

A. Raises an error.
B. Returns an empty object.
C. Returns an object with only key `'a'` (silently drops the missing key).
D. Returns NULL.

---

### Section 4 — Data Visualization and Apps (Q36-45)

**Q36.** You create a dashboard filter `:segment` and reference it in a tile. The query runs but ignores the filter value. Likely cause?

A. Snowsight only supports date filters.
B. Filter name is case-sensitive — maybe `:Segment` vs `:segment`.
C. Dashboards cannot use parameterized filters.
D. Filters only apply to Streamlit apps.

---

**Q37.** The best chart for comparing a single KPI (e.g., total revenue this quarter) with a prior-period delta in a Snowsight dashboard is:

A. Line chart
B. Scatter plot
C. Scorecard
D. Heatgrid

---

**Q38.** A line chart in Snowsight needs multiple series (one per segment). What SQL shape is correct?

A. Pivot the segments into separate columns.
B. Long format: one row per (time, segment) with a Series binding.
C. One row per time with a comma-separated segment string.
D. Separate tiles per segment.

---

**Q39.** Streamlit-in-Snowflake code uses `get_active_session()` from which module?

A. `snowflake.connector`
B. `snowflake.snowpark.context`
C. `snowflake.streamlit`
D. `pandas`

---

**Q40.** Which Streamlit decorator caches a Snowpark session object (not its serializable results)?

A. `@st.cache_data`
B. `@st.cache_resource`
C. `@st.singleton`
D. `@functools.cache`

---

**Q41.** `st.session_state` is best used for:

A. Caching expensive query results across users.
B. Persistent storage of DataFrames to Snowflake.
C. Maintaining user-specific state across widget re-runs within one session.
D. Replacing the warehouse configuration.

---

**Q42.** A scheduled dashboard refresh runs under whose privileges?

A. The viewer's primary role.
B. The viewer's secondary roles.
C. The dashboard owner's role.
D. The ACCOUNTADMIN role.

---

**Q43.** Which chart type is most appropriate for visualizing the density of (day_of_week × hour) activity?

A. Line chart
B. Scatter plot
C. Scorecard
D. Heatgrid

---

**Q44.** For a Streamlit app that must render 500k rows efficiently, which approach is best?

A. `st.dataframe(session.table("t").to_pandas())`
B. Use pagination or `.limit(N).to_pandas()` and render in chunks.
C. Use `st.table(session.table("t"))`.
D. Convert to a Snowsight dashboard — Streamlit cannot render that many rows.

---

**Q45.** You want all queries from a dashboard tagged for subsequent cost analysis. Where do you set the tag?

A. In the dashboard JSON manifest.
B. `ALTER SESSION SET QUERY_TAG = 'dash_x'` within the dashboard's warehouse preamble.
C. `ALTER WAREHOUSE SET DEFAULT_TAG`.
D. You cannot tag dashboard queries.

---

### Section 5 — Statistical and ML Analytics (Q46-55)

**Q46.** Which Cortex function creates a model for univariate time-series forecasting?

A. `SNOWFLAKE.ML.FORECAST`
B. `SNOWFLAKE.ML.ANOMALY_DETECTION`
C. `SNOWFLAKE.CORTEX.COMPLETE`
D. `SNOWFLAKE.ML.CLASSIFICATION`

---

**Q47.** To forecast 30 periods ahead with a 90% prediction interval, the correct call is:

A. `CALL m!FORECAST(FORECASTING_PERIODS => 30, prediction_interval => 0.9);`
B. `CALL m!FORECAST(FORECASTING_PERIODS => 30, CONFIG_OBJECT => {'prediction_interval': 0.9});`
C. `CALL m!FORECAST(PERIODS => 30, CI => 0.9);`
D. `SELECT m.FORECAST(30, 0.9);`

---

**Q48.** To build a FORECAST model that produces per-product forecasts, which parameter do you add?

A. `SERIES_COLNAME => 'product_id'`
B. `GROUP_COLNAME => 'product_id'`
C. `PARTITION_COLNAME => 'product_id'`
D. `MULTISERIES => TRUE`

---

**Q49.** Cortex ANOMALY_DETECTION — raising `prediction_interval` from 0.90 to 0.99 has what effect?

A. Flags more anomalies.
B. Flags fewer anomalies.
C. Does not affect sensitivity.
D. Causes retraining.

---

**Q50.** `SNOWFLAKE.ML.CLASSIFICATION!PREDICT` returns a VARIANT with fields including:

A. `class`, `probability`, `feature_importance`
B. `label`, `score`, `confidence`
C. `prediction`, `error`, `support`
D. `is_match`, `distance`, `cluster_id`

---

**Q51.** `SNOWFLAKE.ML.TOP_INSIGHTS` is primarily designed for:

A. Point-in-time forecasting of a KPI.
B. Finding interesting subgroups where a metric differs from the overall population.
C. Classifying individual records.
D. Decomposing variance into components.

---

**Q52.** `SNOWFLAKE.ML.CONTRIBUTION_EXPLORER` is used when:

A. You want to decompose the change in a metric between two time periods or groups by dimension.
B. You want zero-shot classification of text.
C. You need a running forecast.
D. You need an anomaly flag per row.

---

**Q53.** `REGR_SLOPE(y, x)` — which value is the dependent variable in this signature?

A. The first argument (`y`).
B. The second argument (`x`).
C. Either, depending on which has more variance.
D. Neither — SNOWFLAKE's REGR_SLOPE is non-standard.

---

**Q54.** Which pattern productionizes Cortex FORECAST with automatic retraining?

A. Create the model once; it refreshes automatically.
B. Use a Dynamic Table for training data + a Task to run `CREATE OR REPLACE` on the model on a schedule.
C. Call `!RETRAIN` nightly from a Streamlit app.
D. Cortex models cannot be scheduled.

---

**Q55.** To compute a precision/recall/F1 score for a classifier in SQL, you'd compute from the confusion matrix:

A. precision = TP / (TP + FN); recall = TP / (TP + FP)
B. precision = TP / (TP + FP); recall = TP / (TP + FN); F1 = 2*TP / (2*TP + FP + FN)
C. accuracy = (TP + TN) / N only
D. Snowflake has `SNOWFLAKE.ML.METRICS!EVAL` built-in that returns all three.

---

### Section 6 — Data Preparation and Sharing (Q56-65)

**Q56.** You need a transformation that supports joins and window functions and auto-refreshes within 5 minutes of base-table changes. Choose:

A. View
B. Materialized View
C. Dynamic Table with `TARGET_LAG = '5 minutes'`
D. Scheduled Task writing to a regular table

---

**Q57.** Materialized Views in Snowflake have which restriction?

A. No aggregations allowed.
B. No window functions, no joins to multiple tables, no non-deterministic functions.
C. Only Enterprise+ accounts can create them.
D. Cannot query VARIANT columns.

---

**Q58.** Which clause creates a VIEW that hides its SQL definition from consumers?

A. `CREATE VIEW ... HIDE DEFINITION`
B. `CREATE SECURE VIEW`
C. `CREATE INVISIBLE VIEW`
D. `CREATE VIEW ... WITH ENCRYPTION`

---

**Q59.** You accidentally dropped a column and committed. How do you recover without a backup?

A. `UNDROP COLUMN`
B. Create a clone `AT (OFFSET => -300)` from before the change and promote.
C. Submit a support ticket; data is lost.
D. `ALTER TABLE ... RESTORE COLUMN`.

---

**Q60.** On a 10 TB clustered table, which Query Profile metric indicates good pruning?

A. `bytes_scanned` close to table size.
B. `partitions_scanned / partitions_total` close to 1.0.
C. `partitions_scanned / partitions_total` close to 0.0 (very few partitions read).
D. Spill-to-remote-storage > 0.

---

**Q61.** For a Snowflake Marketplace listing you've accepted, which statement is TRUE?

A. You can create tables in the shared database.
B. You pay for the data at the listing's listed GB price.
C. The provider's row access policies apply to your queries.
D. You must manually ETL the shared data to query it.

---

**Q62.** To add many concurrent viewers without making individual queries faster, you should:

A. Scale UP the warehouse (XS → L).
B. Scale OUT (multi-cluster warehouse, increase MAX_CLUSTER_COUNT).
C. Enable Query Acceleration Service.
D. Add a clustering key.

---

**Q63.** Search Optimization Service is best suited for:

A. Date range scans on time-series.
B. Highly selective point-lookup queries (equality, IN).
C. Large aggregations across the full table.
D. Semi-structured recursive flattens.

---

**Q64.** `CREATE TABLE t2 CLONE t1 AT (OFFSET => -3600)` does what?

A. Clones t1 as it was 1 hour ago.
B. Clones t1 and offsets its timestamps by -3600 seconds.
C. Clones only the last 3600 rows.
D. Errors — OFFSET is not supported.

---

**Q65.** You want to share aggregated analytics with a partner without exposing raw rows. The producer-side pattern is:

A. Create a regular view and grant SELECT to a share.
B. Grant SELECT on the underlying raw tables to the share.
C. Create a SECURE VIEW aggregating raw tables and grant SELECT on the view to a share.
D. Copy data into a new database and share that database.

---

---

## Answer Key with Explanations

### Section 1

**A1. B.** `SAMPLE SYSTEM` (also written `SAMPLE BLOCK`) samples whole micro-partitions. Faster than `BERNOULLI` which scans every row, but biased when partitions correlate with the column of interest. `LIMIT` is not random. See Module 01 Q7.

**A2. A.** HyperLogLog uses a fixed-size sketch (~16KB per group) regardless of cardinality, trading ~1.6% relative error for constant memory. It does not sample. Module 01 Q2.

**A3. C.** The ratio of NULL/blank emails to total rows measures completeness — "are required fields populated?" Module 01 Q3.

**A4. B.** `PERCENTILE_CONT` interpolates between adjacent values. On {10,20} at p=0.5, that's 10 + 0.5*(20-10) = 15. `PERCENTILE_DISC(0.5)` would return 10. Module 01 Q4.

**A5. A.** Snowflake's `KURTOSIS` returns **excess** kurtosis (normal distribution = 0, not 3). 4.2 means tails are heavier than normal. Module 01 Q13.

**A6. C.** Wrapping the column in `DATE_TRUNC` prevents the engine from using column min/max metadata for pruning. All other options allow direct comparison. Module 06 Q8.

**A7. B.** `counters` must be >> k for SpaceSaving accuracy. 5 counters for top-5 is near-worst-case. Aim for 10-100× k. Module 01 Q9.

**A8. B.** `INFORMATION_SCHEMA.TABLES.ROW_COUNT` is maintained metadata, accurate for permanent/transient tables but **not** for views. It is real-time (no 3h lag — that's `ACCOUNT_USAGE`). Module 01 Q1.

**A9. A.** `TYPEOF(v)` returns a string like `'INTEGER'`, `'VARCHAR'`, `'ARRAY'`, `'OBJECT'`, `'NULL_VALUE'`. Module 03 Q13.

**A10. B.** `OVER ()` is a valid window spec (whole table as one partition). It attaches the overall average to each row, so output has 1000 rows. `SELECT AVG(revenue) FROM t` without GROUP BY aggregates to 1 row. Module 02 Q1.

### Section 2

**A11. B.** The default frame when ORDER BY is present is `RANGE UNBOUNDED PRECEDING TO CURRENT ROW`. So `LAST_VALUE` reads up to the current row only. Fix: `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`. Module 02 Q16.

**A12. B.** `RANK()` gives the same rank to tied rows, so `rank = 1` keeps all tied most-recent orders. `ROW_NUMBER` picks one arbitrarily. `DENSE_RANK = 1` also works but `RANK` is the canonical answer for "ties kept." D is wrong because WHERE cannot reference window functions. Module 02 Q3.

**A13. D.** `CUBE(a,b,c)` produces 2^n groupings = 2^3 = 8: (a,b,c),(a,b),(a,c),(b,c),(a),(b),(c),(). Module 02 Q7.

**A14. B.** `{3,}` means "3 or more." `{3}` is exactly 3. `*` is 0+. `+` is 1+. Module 02 Q11.

**A15. C.** `DEFINE` associates boolean predicates with pattern variables. `PATTERN` orders them, `MEASURES` emits outputs, `PARTITION BY/ORDER BY` scope the data. Module 02 Q11.

**A16. C.** Pivoted column names come out as quoted uppercase identifiers. Reference as `"A"`. Module 02 Q9.

**A17. B.** `QUALIFY` is Snowflake's filter on window function results. Module 02 Q4.

**A18. C.** `COUNT_IF` is an aggregate, not a window function. Use `SUM(IFF(status='active',1,0)) OVER (...)` or `COUNT(CASE WHEN ... THEN 1 END) OVER (...)`. Module 02 Q20.

**A19. B.** 30-row window = current + 29 preceding. `BETWEEN 29 PRECEDING AND CURRENT ROW`. Module 02 Q6.

**A20. B.** Negating the row number before adding to date keeps the result constant for a contiguous run. Subtracting from DATE is standard; use `DATEADD('day', -row_num, dt)`. Module 02 Q15.

**A21. B.** `UNPIVOT INCLUDE NULLS` retains rows where unpivoted values are NULL. Default is `EXCLUDE NULLS`. Module 02 Q10.

**A22. C.** Bit encoding: leftmost column is the HIGH-order bit. For `GROUPING_ID(a,b)`: 00=both grouped, 01=b aggregated (grouped by a only), 10=a aggregated (grouped by b only), 11=grand total. "Grouped by a only" = b is subtotal = bit for b is 1, bit for a is 0 → binary 01 = decimal 1. **Wait**: this depends on interpretation. Snowflake treats leftmost as highest bit, so GROUPING_ID(a,b) has a as bit 1, b as bit 0. "Grouped by a only, b aggregated" means b is at ALL level, so b's bit = 1 → 01 = 1. **Correct answer: B.**

Correction: **A22 answer is B (1).** The question asked "grouped by a only, b is subtotal" — b is aggregated-to-all, so b's bit is set. With a as high-order and b as low-order: 01 = 1. Module 02 Q8.

**A23. C.** `LISTAGG` output capped at 16 MB VARCHAR. `ARRAY_AGG` returns a VARIANT array which can accommodate larger payloads and preserves types. Module 02 Q21-22.

**A24. B.** `AFTER MATCH SKIP PAST LAST ROW` resumes matching at the row after the current match's last row. Alternatives: `TO NEXT ROW`, `TO LAST var`. Module 02 Q11.

**A25. B.** Partitioning by customer_id, no ORDER BY needed (whole partition). This computes `SUM(amount) OVER (PARTITION BY customer_id)` on the denominator. Module 02 Q1.

### Section 3

**A26. C.** `outer => TRUE` keeps parent rows with empty/NULL arrays (LEFT JOIN semantics). Module 03 Q5.

**A27. B.** Colon then dot is the standard VARIANT path. Brackets for arrays, cast at the end. Module 03 Q3.

**A28. B.** `IFF(IS_ARRAY(v), v, ARRAY_CONSTRUCT(v))` normalizes scalar-or-array polymorphism. Module 03 Q13.

**A29. B.** `GET_PRESIGNED_URL(stage, relative_path, expiration_seconds)` returns a time-bounded signed URL. `BUILD_SCOPED_FILE_URL` returns a Snowflake-internal URL for AI function calls. Module 03 Q8.

**A30. B.** `OBJECT_CONSTRUCT` drops NULL-valued keys by default. Use `OBJECT_CONSTRUCT_KEEP_NULL` to retain them. Module 03 Q2.

**A31. B.** `TRY_PARSE_JSON` returns NULL on invalid input; `PARSE_JSON` errors. `CHECK_JSON` returns NULL if valid, error message string if invalid. Module 03 Q20.

**A32. C.** External tables expose stage files as queryable tables without ingestion. Directory tables are for file metadata only. Module 03 Q22.

**A33. B.** Explicit `::STRING` cast enables shred-aware predicate pushdown. Without it, the engine compares VARIANT to STRING and may not use shred metadata. Module 03 Q21.

**A34. C.** `INDEX` is the array position (0-based). `KEY` is the object key. `PATH` is the string path. `SEQ` is a global sequence number. Module 03 Q4.

**A35. C.** `OBJECT_PICK` silently drops missing keys without error. `OBJECT_INSERT(update=FALSE)` on existing key errors, but PICK does not. Module 03 Q10.

### Section 4

**A36. B.** Filter names are case-sensitive. `:Segment` and `:segment` are different parameters. Module 04 Q3.

**A37. C.** Scorecard is designed for single KPIs with optional trend delta. Module 04 Q8.

**A38. B.** Long format with a Series binding is the standard pattern. Pivoted/wide formats force manual column mapping. Module 04 Q9.

**A39. B.** `snowflake.snowpark.context.get_active_session`. Module 04 Q6.

**A40. B.** `@st.cache_resource` for non-serializable singletons (sessions, model handles). `@st.cache_data` for serializable results (DataFrames, dicts). Module 04 Q14.

**A41. C.** `st.session_state` maintains dict-like state across widget re-runs within one user session. Module 04 Q15.

**A42. C.** Scheduled dashboards run under the owner's role, not the viewer's. Row Access Policies apply to the owner's context. Module 04 Q4.

**A43. D.** Heatgrid shows 2D density (dow × hour) with color intensity for the metric. Module 04 Q1.

**A44. B.** Pagination or `.limit(N)` avoids shipping gigabytes to the Streamlit server. `st.dataframe` renders ~50k rows cleanly. Module 04 Q16.

**A45. B.** Set `QUERY_TAG` at session start. The dashboard's warehouse session preamble is the appropriate place. Module 04 Q21.

### Section 5

**A46. A.** `SNOWFLAKE.ML.FORECAST`. ANOMALY_DETECTION is for outlier flagging, COMPLETE is an LLM, CLASSIFICATION is for supervised categorical outputs. Module 05 Q2.

**A47. B.** `CONFIG_OBJECT => {'prediction_interval': 0.9}` is the correct syntax. The prediction_interval goes INSIDE CONFIG_OBJECT. Module 05 Q5.

**A48. A.** `SERIES_COLNAME` turns a single-model forecast into per-series forecasting. Module 05 Q4.

**A49. B.** Wider interval (0.99) = looser bounds = fewer deviations beyond them = fewer anomalies flagged. Module 05 Q7.

**A50. A.** `class`, `probability` (per-class dict), `feature_importance` (per-row/local). Module 05 Q9.

**A51. B.** TOP_INSIGHTS finds interesting subgroups where a metric differs from the overall average. Module 05 Q10.

**A52. A.** CONTRIBUTION_EXPLORER decomposes metric change between a baseline and a comparison period/group. Module 05 Q11.

**A53. A.** The first argument of `REGR_SLOPE(y, x)` is y (dependent). This is the opposite of math convention — a common exam trap. Module 05 Q13.

**A54. B.** Dynamic Table for the training view (auto-refreshing), Task running `CREATE OR REPLACE` on the Cortex model on cron schedule. Serving via a view calling `!FORECAST`. Module 05 Q21.

**A55. B.** Standard definitions. Precision = TP / predicted positive. Recall = TP / actual positive. F1 harmonic mean. Snowflake has no built-in `METRICS!EVAL` for this. Module 05 Q19.

### Section 6

**A56. C.** Dynamic Table with 5-minute TARGET_LAG. Materialized Views can't do joins or windows. Views don't auto-refresh (they re-query every time). Tasks require manual pipeline management. Module 06 Q1-2.

**A57. B.** MVs restrict: no joins (beyond simple dimension lookups), no window functions, no non-deterministic functions. Module 06 Q1.

**A58. B.** `CREATE SECURE VIEW` hides the SQL text. Essential for secure data sharing. Module 06 Q3.

**A59. B.** Time Travel + CLONE: `CREATE OR REPLACE TABLE t CLONE t AT (OFFSET => -300)` or similar. Module 06 Q16.

**A60. C.** Few partitions scanned relative to total = good pruning. 90%+ pruning is a common target. Module 06 Q8.

**A61. C.** Row Access Policies and masking from the provider still apply to consumer queries. Consumers cannot create objects (read-only). Marketplace data pricing varies but you don't necessarily pay listed GB price — many listings are free. Module 06 Q5.

**A62. B.** Multi-cluster (scale out) adds parallel clusters for concurrent queries. Scaling UP increases single-query speed, not concurrency. Module 06 Q11.

**A63. B.** SOS = inverted-index-like structure for point lookups. Clustering is for range scans. Module 06 Q10.

**A64. A.** `OFFSET => -3600` is Time Travel: clone from 3600 seconds (1 hour) ago. Module 06 Q16.

**A65. C.** Secure View required to hide definition; aggregated only; grant SELECT on the view to the share. Module 06 Q22.

---

## Scoring

| Score | Interpretation |
|-------|----------------|
| 55-65 (85%+) | Ready — go sit the exam |
| 48-54 (74-83%) | Review the specific weak sections; retake in a week |
| 40-47 (62-73%) | Spend another 1-2 weeks reviewing modules and flashcards |
| < 40 | Review all six Q&A modules systematically; the fundamentals need reinforcement |

### Weak-Spot Grid

Count misses by section. The exam weights ~25% to Advanced SQL and ~22% to Statistical/ML — spend proportional review time if misses cluster there.

| Section | Questions | Your # correct | Weight |
|---------|-----------|----------------|--------|
| Fundamentals | 1-10 | | 17% |
| Advanced SQL | 11-25 | | 27% |
| Semi/Unstructured | 26-35 | | 17% |
| Visualization/Apps | 36-45 | | 17% |
| Statistical/ML | 46-55 | | 22% |
| Prep/Sharing | 56-65 | | 17% |
