# DEA-C02 Mock Exam (65 Questions)

Mirrors the real exam distribution: 18 Data Movement, 14 Performance, 12 Storage, 12 Transformation, 9 Security. Mix of single-select and multi-select; "Choose TWO/THREE" tagged explicitly. Target: 115 minutes, scoring ≥70%.

Difficulty legend: `[F]` Foundation · `[I]` Intermediate · `[A]` Advanced

Instructions: Work straight through without peeking. Record your answers (e.g., 1=B, 2=A,C...). Score against the key at the bottom; use the pointer back to the source Q&A file to review wrong answers.

---

## QUESTIONS

### 1. [F] Data Movement
A data engineer needs to ingest 50 MB JSON files that land in S3 every 2 minutes, with a <2-minute end-to-end latency SLA and at-least-once semantics acceptable. Which service is the best fit?

A. `COPY INTO` on a cron schedule every minute
B. Snowpipe auto-ingest with event notification
C. Snowpipe Streaming SDK
D. External table with scheduled refresh

---

### 2. [I] Data Movement
A pipe has been suspended via `ALTER PIPE sales_pipe SUSPEND` for 3 days. Which statement is correct?

A. The SQS queue has stopped receiving events; recent files must be reloaded via REFRESH.
B. The SQS queue continues collecting events; resume drains the backlog, limited by the 4-day SQS retention.
C. The pipe state is `STOPPED_CLONED_OR_DROPPED` and must be recreated.
D. Resume will fail because the owner role has expired.

---

### 3. [I] Data Movement — Choose TWO
Which statements about Snowpipe Streaming are correct?

A. Supports exactly-once semantics per channel via offset tokens.
B. Requires password authentication for client connections.
C. Writes bypass stage files and go directly into micro-partitions.
D. Has a dedup window of 14 days keyed on filename.
E. Latency is comparable to classic Snowpipe auto-ingest (~1 min).

---

### 4. [A] Data Movement
In a Stream+Task CDC pipeline, which pattern correctly handles late-arriving records with `event_time` to avoid clobbering newer target rows with older updates?

A. Use `MERGE` without any event_time guard; last write wins.
B. Use `MERGE ... WHEN MATCHED AND s.event_time > t.event_time THEN UPDATE ...`
C. Use `INSERT ON DUPLICATE KEY UPDATE`.
D. Create a BEFORE trigger to reject old events.

---

### 5. [F] Data Movement
A Snowpipe loaded a file 10 days ago. The file was re-uploaded with the same name today. What happens?

A. The file is deduped and skipped (within 14-day history).
B. The file is loaded again, creating duplicates.
C. The pipe transitions to STOPPED_CLONED_OR_DROPPED.
D. Snowflake raises an error for duplicate filename.

---

### 6. [I] Data Movement
Which Snowpipe REFRESH behavior is correct?

A. Enqueues files uploaded in the last 14 days.
B. Enqueues files uploaded in the last 7 days matching the pipe's PATTERN.
C. Reloads all files in the stage regardless of age.
D. Only works on internal stages, not external.

---

### 7. [A] Data Movement — Choose TWO
A Dynamic Table joins `orders` and `customers`. Which conditions keep refreshes as INCREMENTAL rather than falling back to FULL?

A. The query uses `SELECT *` from `orders`.
B. Both source tables have change tracking enabled.
C. The query uses `OVER (ORDER BY ts ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)`.
D. Join predicate is a deterministic equi-join on business keys.
E. The query references `CURRENT_TIMESTAMP()`.

---

### 8. [I] Data Movement
Which privilege must the loading role hold for `ENABLE_SCHEMA_EVOLUTION = TRUE` to add new columns during a `COPY INTO`?

A. INSERT only
B. OWNERSHIP
C. EVOLVE SCHEMA
D. MANAGE GRANTS

---

### 9. [I] Data Movement
A task graph has 1 root and 3 dependent tasks. Developer attempts `ALTER TASK t_root RESUME` while children are suspended. Expected behavior?

A. Snowflake resumes root and all children automatically.
B. Snowflake errors — children must be resumed first.
C. Root resumes but silently skips children.
D. Children remain suspended; root runs and dependents are no-ops.

---

### 10. [A] Data Movement
A fact table receives 10,000 files per day via Snowpipe auto-ingest. Average file size is 500 KB. Which recommendation reduces cost most effectively?

A. Switch to Snowpipe Streaming for exactly-once semantics.
B. Enlarge files upstream to 100–250 MB compressed and switch to batch COPY on a schedule.
C. Enable `MATCH_BY_COLUMN_NAME = CASE_INSENSITIVE`.
D. Reduce Snowpipe warehouse size.

---

### 11. [I] Data Movement
Which statement about `METADATA$FILENAME` and `METADATA$FILE_ROW_NUMBER` is correct?

A. They're columns added automatically to every table.
B. They're pseudo-columns used inside the SELECT list of a COPY to project source filename + line.
C. They only work with Parquet files.
D. They're stored in INFORMATION_SCHEMA.LOAD_HISTORY, not accessible in queries.

---

### 12. [I] Data Movement
A Kafka connector in classic (file) mode produces latency of ~60s. The team wants <10s latency and exactly-once semantics. Which single change achieves this?

A. Reduce `buffer.flush.time` to 1 second.
B. Set `snowflake.ingestion.method=SNOWPIPE_STREAMING` and use key-pair auth.
C. Increase Kafka Connect `tasks.max`.
D. Switch target tables to transient to reduce storage overhead.

---

### 13. [F] Data Movement
A `COPY INTO` with `ON_ERROR = 'CONTINUE'` finishes with 500 rejected rows. How do you reconcile?

A. The failed rows are automatically retried on the next COPY.
B. Query `TABLE(VALIDATE(target_table, JOB_ID => '_last'))` to list per-file errors.
C. Check `AUTOMATIC_CLUSTERING_HISTORY`.
D. Use `SYSTEM$PIPE_STATUS('pipe_name')`.

---

### 14. [I] Data Movement
Which object exposes `relative_path`, `size`, `last_modified`, `md5`, and `file_url` for files on a named stage?

A. External table
B. Directory table
C. Iceberg table
D. Materialized view

---

### 15. [A] Data Movement — Choose TWO
Which statements about Streams are correct?

A. A `CREATE OR REPLACE TABLE` on the source makes the stream stale.
B. `SHOW_INITIAL_ROWS = TRUE` returns all current rows on every read.
C. Each DML consuming a stream advances its offset atomically on commit.
D. A single table can have at most one stream.
E. `SYSTEM$STREAM_HAS_DATA` is free to invoke in a Task `WHEN` clause.

---

### 16. [A] Data Movement
A storage integration is created pointing to an S3 bucket. The S3 role trust policy references the correct external ID but Snowflake still fails to list files. Most likely cause?

A. The storage integration was created with wrong `TYPE`.
B. The S3 bucket IAM role is missing `s3:ListBucket` and `s3:GetObject` for the Snowflake-generated external ID.
C. The pipe is in state `PAUSED_BY_SYSTEM`.
D. Account-level network policy blocks AWS endpoints.

---

### 17. [I] Data Movement
Which continuous ingestion method provides exactly-once semantics?

A. `COPY INTO` from external stage
B. Snowpipe auto-ingest
C. Snowpipe Streaming via SDK
D. External Table auto-refresh

---

### 18. [F] Data Movement
Which COPY INTO option performs a dry-run that returns parse errors without inserting rows?

A. `ON_ERROR = 'ABORT_STATEMENT'`
B. `VALIDATION_MODE = 'RETURN_ERRORS'`
C. `PATTERN = '.*error.*'`
D. `MATCH_BY_COLUMN_NAME = CASE_SENSITIVE`

---

### 19. [F] Performance
A `SELECT COUNT(*) FROM huge_table` with no WHERE clause completes in milliseconds and shows `bytes_scanned = 0`. Which cache served this?

A. Result cache
B. Warehouse SSD cache
C. Metadata cache
D. Query acceleration service

---

### 20. [I] Performance
Query A ran yesterday; Query B is byte-for-byte identical, same user and role, run today. The underlying table had an INSERT 10 minutes ago. Which statement is correct?

A. Query B returns the cached result from Query A instantly.
B. The result cache is invalidated by the INSERT; Query B re-executes.
C. The warehouse SSD cache returns results without re-reading storage.
D. Query B fails because the cache is stale.

---

### 21. [I] Performance — Choose TWO
Which conditions defeat micro-partition pruning on a column `dt`?

A. `WHERE dt = '2026-10-30'::DATE`
B. `WHERE DATE_TRUNC('day', dt) = '2026-10-30'`
C. `WHERE dt BETWEEN '2026-10-01' AND '2026-10-30'`
D. `WHERE dt::VARCHAR LIKE '2026-10%'`
E. `WHERE dt >= DATEADD(day, -7, CURRENT_DATE)`

---

### 22. [A] Performance
A table clustered by `(dt, region)` has `average_depth = 25`. Query filters are `WHERE dt=? AND region=?`. Clustering credit usage last 30 days has been stable. What's the most likely cause of degraded performance?

A. The clustering key is wrong — reverse to `(region, dt)`.
B. Auto-clustering was suspended; reclustering is not catching up.
C. The warehouse is too small for the join size.
D. Result cache is disabled.

---

### 23. [I] Performance
Search Optimization Service is enabled on `EQUALITY(order_id)`. A query filters `WHERE DATE(order_date) = '2026-10-01'`. Will SOS accelerate this query?

A. Yes — SOS indexes all equality predicates.
B. No — SOS only accelerates columns explicitly indexed, and `order_date` is not in the SOS definition.
C. Yes, after a 24-hour SOS warmup.
D. Only if the warehouse is Snowpark-optimized.

---

### 24. [I] Performance
A Materialized View is defined over `sales` with `SELECT dt, region, SUM(amt) GROUP BY dt, region`. A query runs `SELECT region, SUM(amt) FROM sales WHERE dt='2026-10-30' GROUP BY region`. Which is correct?

A. The optimizer always ignores the MV because the query doesn't name it.
B. The optimizer may rewrite the query to use the MV if it proves equivalence.
C. The MV must be referenced explicitly in FROM.
D. MVs cannot have GROUP BY clauses.

---

### 25. [A] Performance — Choose TWO
Which Query Profile observations most strongly indicate the need to resize the warehouse up?

A. `bytes_spilled_to_local_storage > 0`
B. `bytes_spilled_to_remote_storage > 1 GB`
C. `percentage_scanned_from_cache = 0`
D. Synchronization time > 30% of total
E. `partitions_scanned = partitions_total`

---

### 26. [F] Performance
A warehouse resumes, runs a 10-second query, and auto-suspends. How many seconds are billed?

A. 10 (per-second after resume)
B. 60 (60-second minimum per resume)
C. 0 (metadata-only)
D. 120 (minimum 2 minutes)

---

### 27. [I] Performance
Which setting most directly controls how aggressively a multi-cluster warehouse adds clusters when queries queue?

A. `WAREHOUSE_SIZE`
B. `AUTO_SUSPEND`
C. `SCALING_POLICY` (`STANDARD` vs `ECONOMY`)
D. `MAX_CONCURRENCY_LEVEL`

---

### 28. [A] Performance
A Snowpark Python stored procedure trains a scikit-learn model on a 50M-row pandas DataFrame. Executions on a Standard L warehouse fail with OOM. Best fix?

A. Reduce DataFrame size by sampling.
B. Switch to a Snowpark-optimized warehouse of size M or larger.
C. Enable Query Acceleration Service.
D. Enable Search Optimization Service.

---

### 29. [I] Performance
When does `RESULT_SCAN(LAST_QUERY_ID())` return cached results?

A. Within 24 hours, if the query result is still cached and user has access to underlying objects.
B. Only for queries returning < 1000 rows.
C. Only for aggregation queries.
D. Always, regardless of time.

---

### 30. [I] Performance
A warehouse has `ENABLE_QUERY_ACCELERATION = TRUE` and `QUERY_ACCELERATION_MAX_SCALE_FACTOR = 8`. For eligible queries, what's the effect?

A. All queries run on a dedicated 8x cluster.
B. Scan-heavy portions of eligible queries can borrow up to 8x additional compute temporarily.
C. The warehouse size is permanently increased 8x.
D. Multi-cluster scales out to 8 clusters.

---

### 31. [A] Performance
Join between a 10 TB `sales` table and a 10 MB `dim_region` table runs slowly. Query Profile shows a HashJoin node with build side as `sales`. What should the optimizer have done differently, and how do you nudge it?

A. The optimizer should build on the small side (`dim_region`); add an explicit JOIN hint.
B. The optimizer should build on the small side; Snowflake doesn't expose hints, but check stats via INFER_SCHEMA.
C. The optimizer should build on the small side; refresh stats manually with `ANALYZE TABLE`.
D. The optimizer should build on the small side; Snowflake has no `ANALYZE TABLE` — rewrite the query (CTE on small side, filter early) to influence CBO cardinality estimates.

---

### 32. [I] Performance
A query tag `{"team":"analytics","job":"nightly"}` is set at session start. Where do you query credit attribution by team?

A. `SNOWFLAKE.INFORMATION_SCHEMA.QUERY_HISTORY_BY_SESSION`
B. `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY` with `query_tag` JSON parsing
C. `SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY`
D. `SYSTEM$QUERY_TAG_USAGE()`

---

### 33. [F] Storage
What is the retention of Fail-safe on a permanent table?

A. 1 day
B. 7 days, non-configurable
C. 90 days configurable
D. 14 days

---

### 34. [I] Storage
A clone is created of a 10 TB table. Immediately after creation, how much additional storage is consumed?

A. 10 TB (deep copy)
B. ~0 (metadata-only pointer sharing)
C. 1 TB (10% delta copy)
D. Depends on `DATA_RETENTION_TIME_IN_DAYS`

---

### 35. [I] Storage — Choose TWO
Which statements about transient tables are correct?

A. Max Time Travel is 1 day.
B. Have 7-day Fail-safe.
C. Can be converted in place to permanent via `ALTER TABLE`.
D. No Fail-safe tier (reduces storage cost ~30% vs permanent).
E. Dropped automatically at session end.

---

### 36. [A] Storage
`ALTER TABLE huge_table SET DATA_RETENTION_TIME_IN_DAYS = 1` is executed on a permanent table that previously had retention=90 days. Which is correct?

A. Storage drops immediately as old MPs are deleted.
B. MPs older than 1 day move to Fail-safe immediately and drain over the following 7 days.
C. The change takes effect only after the next DML.
D. The MPs are permanently deleted at the moment of the ALTER.

---

### 37. [F] Storage
`UNDROP TABLE sales` fails. Which situation is a likely cause?

A. A new table named `sales` was created after the drop.
B. The warehouse was suspended at the moment of UNDROP.
C. The current role has `OWNERSHIP` on the parent schema.
D. The database is being replicated.

---

### 38. [A] Storage — Choose TWO
Which objects **cannot** be fully functionally cloned via `CREATE ... CLONE`?

A. Permanent tables
B. External tables
C. Materialized views
D. Pipes (metadata only, SQS endpoint not cloned)
E. Schemas

---

### 39. [I] Storage
Cross-cloud replication (e.g., AWS → Azure) requires which Snowflake edition minimum?

A. Standard
B. Enterprise
C. Business Critical
D. VPS

---

### 40. [A] Storage
A table has a stream whose offset is 20 days old. Base `DATA_RETENTION_TIME_IN_DAYS = 1`, `MAX_DATA_EXTENSION_TIME_IN_DAYS = 14`. Current state of the stream?

A. Stream is current; retention is extended to 20 days.
B. Stream is stale; data for the stream's required window has aged out past the extension cap.
C. Stream automatically recreates with `SHOW_INITIAL_ROWS = TRUE`.
D. Snowflake raises an error on the next read.

---

### 41. [I] Storage
A Hybrid Table is best suited for which workload?

A. Full-table scans for BI dashboards.
B. OLTP-style single-row writes and sub-millisecond PK lookups.
C. Historical reporting across many years.
D. Semi-structured JSON ingestion.

---

### 42. [A] Storage
A consumer account tries `CREATE TABLE t_copy CLONE provider_db.public.shared_sales`. Expected behavior?

A. Succeeds — clone creates a zero-copy pointer to the shared data.
B. Fails — shared tables cannot be cloned; use CTAS instead.
C. Succeeds, but creates a shallow copy that becomes read-only.
D. Succeeds, but incurs cross-account replication charges.

---

### 43. [I] Storage
Which Account Usage view shows per-table storage breakdown (active/TT/FS/retained_for_clone)?

A. `STORAGE_USAGE`
B. `TABLE_STORAGE_METRICS`
C. `DATABASE_STORAGE_USAGE_HISTORY`
D. `WAREHOUSE_METERING_HISTORY`

---

### 44. [F] Storage
A one-row UPDATE on a table whose MP contains 1 million rows causes what physically?

A. Only the single row's bytes are rewritten in place.
B. The entire MP is rewritten (new MP created, old MP retained for TT).
C. The row is appended to a journal and consolidated nightly.
D. Nothing — Snowflake uses row-level MVCC.

---

### 45. [F] Transformation
Which UDF type offers the lowest per-row overhead?

A. Scalar Python UDF
B. Vectorized Python UDF
C. Scalar SQL UDF (inlined by optimizer)
D. UDTF

---

### 46. [I] Transformation
Snowpark DataFrame operations are evaluated when?

A. On each operator call (eager).
B. On action methods like `.collect()`, `.show()`, `.save_as_table()`, `.write.save()` (lazy).
C. On session close.
D. Hourly, via a background task.

---

### 47. [I] Transformation — Choose TWO
Which features are specific to Dynamic Tables (not shared with Materialized Views)?

A. Supports joins between multiple source tables.
B. Transparent query rewrite by the optimizer.
C. Explicit `TARGET_LAG` freshness SLA.
D. Automatic maintenance by Snowflake.
E. Must be a single-table projection.

---

### 48. [A] Transformation
A Snowpark Python stored procedure needs to read from one table, call an external REST API with per-row data, and write to another table. Which configuration is required for the API call?

A. `EXECUTE AS OWNER` only.
B. Create an EXTERNAL ACCESS INTEGRATION with a NETWORK RULE allowing the API host and a SECRET for credentials, then reference it in the procedure.
C. Set a `NETWORK_POLICY` on the user calling the procedure.
D. Grant `MANAGE GRANTS` to the procedure's owner role.

---

### 49. [I] Transformation
A dbt incremental model uses `unique_key='id'` and `on_schema_change='append_new_columns'`. A new column `region` appears in the source. What happens on next `dbt run`?

A. The model fails due to schema mismatch.
B. The new column is added to the target table; existing rows get NULL for `region`.
C. The entire table is dropped and rebuilt.
D. dbt silently drops the new column.

---

### 50. [A] Transformation
Which SCD Type 2 implementation detail is required to prevent "no-op" updates when source timestamps change but dimension attributes don't?

A. Use `MERGE INTO` with a hash check on tracked columns (`WHERE d.hash <> s.hash`).
B. Enable `ENABLE_SCHEMA_EVOLUTION = TRUE`.
C. Use `COPY INTO` instead of MERGE.
D. Set `TARGET_LAG = DOWNSTREAM` on the dimension.

---

### 51. [I] Transformation
A Snowpark Python UDF depends on `pandas==1.5.3`. How do you pin the version?

A. `PACKAGES = ('pandas==1.5.3')` in the CREATE FUNCTION statement.
B. `pip install pandas==1.5.3` in a @stage.
C. Set a session variable `PANDAS_VERSION`.
D. Pinning is not supported; latest version is used.

---

### 52. [I] Transformation
In Snowflake Scripting, how do you retrieve the row count affected by the most recent DML inside a block?

A. `SQLCODE`
B. `SQLSTATE`
C. `SQLROWCOUNT`
D. `ROW_COUNT()`

---

### 53. [A] Transformation
A Snowpark Python stored proc with `EXECUTE AS OWNER` is granted USAGE to role `DEV`. The procedure body runs `GRANT SELECT ON TABLE x TO ROLE y`. The owner role has MANAGE GRANTS; DEV does not. What happens when DEV calls the procedure?

A. Call fails because DEV lacks MANAGE GRANTS.
B. Call succeeds; the GRANT runs with the owner's privileges.
C. Call succeeds but the GRANT is a no-op.
D. Snowflake issues a warning and auto-escalates DEV.

---

### 54. [I] Transformation
Which Snowpark DataFrame action pulls all rows to the client Python process (OOM risk)?

A. `.show()`
B. `.save_as_table()`
C. `.collect()` / `.to_pandas()`
D. `.filter()`

---

### 55. [A] Transformation — Choose TWO
Which statements about Iceberg Tables in Snowflake are correct?

A. Snowflake-managed catalog mode supports full DML and time travel.
B. Externally-managed catalog mode is always read-only from Snowflake.
C. Iceberg Tables are stored in your cloud storage, not Snowflake's.
D. Time travel on Iceberg uses Snowflake's TT bytes, same as FDN tables.
E. Iceberg format allows interop with Spark/Trino/Athena.

---

### 56. [I] Transformation
Which stored procedure language has native access to Snowpark DataFrames + Python packages like scikit-learn?

A. SQL Scripting
B. JavaScript
C. Python
D. Snowflake Native SQL

---

### 57. [F] Security
Which role is responsible for creating users and roles in a least-privilege RBAC design?

A. ACCOUNTADMIN
B. SECURITYADMIN
C. USERADMIN
D. SYSADMIN

---

### 58. [I] Security
Which combination correctly implements per-column dynamic masking that reveals data only to the `PII_ADMIN` role?

A. `CREATE SECURE VIEW` with CASE in SELECT.
B. `CREATE MASKING POLICY mp ... AS (val STRING) RETURNS STRING -> CASE WHEN CURRENT_ROLE()='PII_ADMIN' THEN val ELSE '***' END` + `ALTER TABLE ... MODIFY COLUMN SET MASKING POLICY mp`.
C. `GRANT SELECT ... TO PII_ADMIN; REVOKE SELECT FROM PUBLIC`.
D. Row access policy on the sensitive column.

---

### 59. [A] Security — Choose TWO
Which statements about Row Access Policies are correct?

A. One table can have multiple row access policies simultaneously.
B. Policies return BOOLEAN per row — TRUE makes the row visible.
C. Policies can reference entitlement tables for user-to-row mapping.
D. Policies modify column values on return.
E. Owner of the table bypasses the policy automatically.

---

### 60. [I] Security
Which statement about `USE SECONDARY ROLES ALL` is correct?

A. Changes the primary role, affecting object ownership on CREATE.
B. Adds all granted roles' privileges to the session as a union; object creation still uses the primary role.
C. Disables all masking policies.
D. Requires ACCOUNTADMIN privilege.

---

### 61. [I] Security
A tag `CLASSIFICATION = 'PII_EMAIL'` has a masking policy attached. A new column with that tag is added to a different table. What happens?

A. Nothing — tag-based policies apply only to initially tagged columns.
B. The masking policy automatically applies to the new column.
C. The tag has to be re-applied by SECURITYADMIN.
D. An error is raised because masking policies cannot be shared.

---

### 62. [A] Security
A service account needs to call Snowpipe REST API. What's the required authentication?

A. Password + MFA.
B. OAuth token with scope `pipe:insert`.
C. Key-pair auth (RSA public key registered on user, client signs JWT).
D. SAML assertion from IdP.

---

### 63. [A] Security
`ACCESS_HISTORY` is used to answer: which users accessed the `customers.email` column in the last 30 days? Which Account Usage view directly provides this with column-level resolution (including view-base-table resolution)?

A. `QUERY_HISTORY`
B. `GRANTS_TO_ROLES`
C. `ACCESS_HISTORY` with FLATTEN on `base_objects_accessed`
D. `LOGIN_HISTORY`

---

### 64. [I] Security
Which combination hides a view's definition from external data share consumers?

A. `CREATE VIEW` + `GRANT SELECT`
B. `CREATE SECURE VIEW` + `GRANT SELECT ... TO SHARE`
C. `CREATE VIEW` with `COMMENT = 'HIDDEN'`
D. `REVOKE DESC`

---

### 65. [A] Security
A pipe owner role is dropped. What state does the pipe transition to, and what's the fix?

A. State remains RUNNING; no action needed.
B. State becomes PAUSED_BY_SYSTEM; re-grant ownership to a valid role and resume.
C. State becomes STOPPED_CLONED_OR_DROPPED; recreate the pipe.
D. State becomes SUSPENDED; call `ALTER PIPE RESUME`.

---

## ANSWER KEY

### 1. **B** — Snowpipe auto-ingest with event notification
Latency ~1 min matches the <2-minute SLA. Snowpipe Streaming (<10s) is overkill and more complex when at-least-once is acceptable. COPY cron doesn't scale cost-efficiently for frequent small files. External tables don't ingest.
See `01_data_movement_QA.md` Q1 and Q21.

### 2. **B** — SQS queue continues collecting events; resume drains the backlog, limited by 4-day SQS retention
`SUSPEND` on the pipe stops the pipe from consuming the queue but SQS keeps enqueuing events for its 4-day retention. Compare to `PIPE_EXECUTION_PAUSED` which pauses differently.
See `01_data_movement_QA.md` Q16.

### 3. **A, C** — Exactly-once + direct micro-partition writes
Snowpipe Streaming provides exactly-once via offset tokens and writes directly to micro-partitions. It requires key-pair auth (not password), has sub-10s latency, and uses offset-token dedup (not 14-day file dedup).
See `01_data_movement_QA.md` Q2.

### 4. **B** — MERGE with `WHEN MATCHED AND s.event_time > t.event_time`
Prevents older events from overwriting newer target rows. Pair with `QUALIFY ROW_NUMBER()` on source for intra-batch dedup.
See `01_data_movement_QA.md` Q22.

### 5. **A** — Deduped and skipped (14-day file history)
Snowpipe tracks loaded filenames for 14 days. A 10-day-old re-upload is within that window and gets skipped. If re-uploaded at day 15+, it would reload.
See `01_data_movement_QA.md` Q1 and Q15.

### 6. **B** — Enqueues files from last 7 days matching the PATTERN
`ALTER PIPE REFRESH` is a one-time catch-up limited to the last 7 days. Dedup (14-day file history) catches already-loaded files.
See `01_data_movement_QA.md` Q15.

### 7. **B, D** — Change tracking + deterministic equi-join
Change tracking is prerequisite for incremental refresh. Deterministic equi-joins are incrementalizable. `SELECT *`, window functions with ORDER BY, and `CURRENT_TIMESTAMP()` force FULL refresh.
See `01_data_movement_QA.md` Q14 and `05_data_transformation_QA.md` Q4.

### 8. **C** — EVOLVE SCHEMA
`EVOLVE SCHEMA` is a distinct privilege, not implied by INSERT. Must be explicitly granted to the loading role.
See `01_data_movement_QA.md` Q7 and Q24.

### 9. **B** — Snowflake errors — children must be resumed first
Resume order must be children → root; suspend order is reverse. Snowflake enforces to prevent orphan dependents.
See `01_data_movement_QA.md` Q4.

### 10. **B** — Enlarge files to 100–250 MB and switch to batch COPY
10,000 × 500 KB files/day = ~0.06 credits/1000 files overhead × 10 = ~0.6 credits/day just in Snowpipe per-file charges, plus compute. Larger files + scheduled COPY is dramatically cheaper for this volume/latency.
See `01_data_movement_QA.md` Q1 and Q21.

### 11. **B** — Pseudo-columns inside the COPY SELECT list
`METADATA$FILENAME` and `METADATA$FILE_ROW_NUMBER` are projectable in the COPY INTO SELECT, letting you store source lineage per row.
See `01_data_movement_QA.md` Q12.

### 12. **B** — Switch to SNOWPIPE_STREAMING + key-pair auth
Streaming mode in the Kafka connector delivers ~5s latency with exactly-once per Kafka partition. Requires key-pair auth.
See `01_data_movement_QA.md` Q6.

### 13. **B** — Query `VALIDATE()`
`VALIDATE(target_table, JOB_ID => '_last')` returns per-file error details for the last COPY. Essential companion to `ON_ERROR='CONTINUE'`.
See `01_data_movement_QA.md` Q8.

### 14. **B** — Directory table
Directory tables are implicit metadata tables on named stages. External tables expose data content; Iceberg is for ACID tables; MVs are query materializations.
See `01_data_movement_QA.md` Q5.

### 15. **A, C** — Table replace invalidates stream; DML advances offset atomically
A `CREATE OR REPLACE TABLE` on source makes the stream stale. Stream offset advances on commit of consuming DML. `SHOW_INITIAL_ROWS=TRUE` only applies on first read. Multiple streams per table are allowed.
See `01_data_movement_QA.md` Q3.

### 16. **B** — Missing S3 permissions on the Snowflake-generated external ID
Even with trust policy correct, the role must grant Snowflake's external ID `s3:ListBucket` + `s3:GetObject`. External ID mismatch and missing permissions are the top two integration failures.
See `01_data_movement_QA.md` Q13 and `04_security_QA.md` Q11.

### 17. **C** — Snowpipe Streaming
Only Streaming provides exactly-once (per channel, via offset tokens). Others are at-least-once relying on filename dedup.
See `01_data_movement_QA.md` Q2 and Q21.

### 18. **B** — `VALIDATION_MODE = 'RETURN_ERRORS'`
Returns parse errors without inserting rows. `RETURN_N_ROWS` is a different mode showing N loadable rows (not errors).
See `01_data_movement_QA.md` Q8.

### 19. **C** — Metadata cache
`COUNT(*)` with no filter uses micro-partition metadata (row_count per MP) stored in cloud services — zero bytes scanned, free.
See `02_performance_optimization_QA.md` Q1 and Q19.

### 20. **B** — Result cache invalidated by INSERT; Query B re-executes
Any DML on underlying tables invalidates the result cache. The 24h TTL is for queries where underlying data hasn't changed.
See `02_performance_optimization_QA.md` Q1.

### 21. **B, D** — Expression wrapping on the column
`DATE_TRUNC('day', dt)` and `dt::VARCHAR LIKE` both wrap the column in an expression, defeating MP-level min/max pruning. Literal casts and range operators on the raw column are fine.
See `02_performance_optimization_QA.md` Q2.

### 22. **B** — Auto-clustering suspended or not catching up
Depth of 25 with filters on the key means pruning is broken. If credit usage is stable (not spiking), reclustering isn't actively working. Check `SYSTEM$CLUSTERING_INFORMATION` and `ALTER TABLE RESUME RECLUSTER`.
See `02_performance_optimization_QA.md` Q3 and Q10.

### 23. **B** — SOS only accelerates indexed columns; also, DATE() wrapping would defeat SOS
SOS is per-column. `order_date` isn't in the SOS definition, and even if it were, `DATE(order_date)` would defeat the equality lookup.
See `02_performance_optimization_QA.md` Q4.

### 24. **B** — Optimizer may rewrite to use the MV if equivalence provable
Snowflake MVs support transparent query rewrite. The optimizer checks if the MV can satisfy the query's GROUP BY/filters and substitutes automatically.
See `02_performance_optimization_QA.md` Q5.

### 25. **B, D** — Remote spill and synchronization skew
Remote spill (>GB) is the strongest resize signal; synchronization > 30% indicates data skew where more compute often helps. Local spill alone is tolerable; cache miss is a different problem; full MP scan may just mean no filter.
See `02_performance_optimization_QA.md` Q6 and Q9.

### 26. **B** — 60-second minimum per resume
Snowflake bills a 60s floor per start/resume, then per-second after. Even a 10s query triggers 60s billing.
See `02_performance_optimization_QA.md` Q25.

### 27. **C** — SCALING_POLICY
STANDARD spins up on first queue hit; ECONOMY delays ~6 min. Size controls per-query performance, not concurrency.
See `02_performance_optimization_QA.md` Q8.

### 28. **B** — Snowpark-optimized warehouse (M or larger)
Snowpark-optimized has 16x per-core memory, specifically designed for memory-heavy Python workloads. Minimum size is Medium.
See `02_performance_optimization_QA.md` Q20 and `05_data_transformation_QA.md` Q10.

### 29. **A** — Within 24h with cache valid and access permissions
Result cache is 24h idle; `RESULT_SCAN` taps it. User must still have SELECT on underlying objects when scanning the result.
See `02_performance_optimization_QA.md` Q14.

### 30. **B** — Scan-heavy portions of eligible queries borrow extra compute temporarily
QAS is a per-query burst mechanism, not a permanent resize. Max scale factor caps per-query acceleration.
See `02_performance_optimization_QA.md` Q12.

### 31. **D** — Rewrite the query; Snowflake has no `ANALYZE TABLE` and limited hints
Stats are always current (updated per write). The optimizer uses MP stats + distinct counts; influence it by restructuring SQL (CTEs, pre-aggregation).
See `02_performance_optimization_QA.md` Q11 and Q13.

### 32. **B** — `ACCOUNT_USAGE.QUERY_HISTORY` with JSON-parsed `query_tag`
Tags are propagated per query; parse with `TRY_PARSE_JSON(query_tag):team::STRING` and group.
See `02_performance_optimization_QA.md` Q23.

### 33. **B** — 7 days, non-configurable
Fail-safe is a 7-day post-TT recovery window for permanent tables, accessible only to Snowflake Support.
See `03_storage_and_data_protection_QA.md` Q3.

### 34. **B** — ~0 (metadata-only pointer sharing)
Clones share immutable MPs. Storage grows only when source or clone diverges.
See `03_storage_and_data_protection_QA.md` Q4.

### 35. **A, D** — Max TT 1 day, no Fail-safe tier
Transient tables don't have FS and max TT is 1 day. They are not session-scoped (that's TEMPORARY). You can't convert in place; must drop + recreate.
See `03_storage_and_data_protection_QA.md` Q5.

### 36. **B** — MPs move to Fail-safe immediately; drain over 7 days
Reducing retention classifies aged MPs as past-TT; permanent tables shift them to FS which drains over 7 days.
See `03_storage_and_data_protection_QA.md` Q11.

### 37. **A** — A new table named `sales` was created after the drop
Name collision after DROP + CREATE orphans the dropped object (name is taken). UNDROP fails because the name is already active.
See `03_storage_and_data_protection_QA.md` Q7.

### 38. **B, D** — External tables and Pipes
External tables have no MP state. Pipes clone as metadata but SQS subscription is not cloned — functionally the pipe is broken until recreated.
See `03_storage_and_data_protection_QA.md` Q21.

### 39. **C** — Business Critical
Same-cloud cross-region replication requires Enterprise; cross-cloud (AWS↔Azure↔GCP) requires Business Critical.
See `03_storage_and_data_protection_QA.md` Q6 and Q17.

### 40. **B** — Stream is stale beyond extension cap
20d > 1d (base) + 14d (max extension) = 15 days max. Stream offset exceeds the extended window; stream is stale and must be recreated.
See `03_storage_and_data_protection_QA.md` Q9 and Q16.

### 41. **B** — OLTP single-row writes + PK lookups
Hybrid Tables (Unistore) layer row-store + columnar for sub-millisecond PK ops; not for bulk scans.
See `03_storage_and_data_protection_QA.md` Q15.

### 42. **B** — Fails; shared tables cannot be cloned, use CTAS
Shared tables are read-only pointers in the consumer account; consumer doesn't own storage. CTAS creates a deep independent copy.
See `03_storage_and_data_protection_QA.md` Q23.

### 43. **B** — `TABLE_STORAGE_METRICS`
Snapshot view with per-table active_bytes, time_travel_bytes, failsafe_bytes, retained_for_clone_bytes.
See `03_storage_and_data_protection_QA.md` Q8 and Q25.

### 44. **B** — Entire MP is rewritten; old MP retained for TT
MPs are immutable. A single-row UPDATE triggers full MP rewrite plus TT retention of the old MP.
See `03_storage_and_data_protection_QA.md` Q14.

### 45. **C** — Scalar SQL UDF (inlined by optimizer)
SQL UDFs are inlined into the plan — effectively zero overhead vs. the raw expression. Python/UDTF add sandbox overhead.
See `05_data_transformation_QA.md` Q1.

### 46. **B** — Action methods trigger execution (lazy)
Snowpark compiles DataFrame ops to SQL; execution only on action. Allows plan optimization and avoids row-by-row Python.
See `05_data_transformation_QA.md` Q2.

### 47. **A, C** — Joins between sources, explicit TARGET_LAG
DTs support joins (MVs don't), use TARGET_LAG freshness SLA (MVs auto-maintain), and don't do transparent query rewrite (MVs do). Both have automatic maintenance.
See `05_data_transformation_QA.md` Q4 and `02_performance_optimization_QA.md` Q5.

### 48. **B** — External Access Integration + Network Rule + Secret
Snowflake sandboxes UDF/proc outbound calls. External Access Integration whitelists hosts via a Network Rule and manages credentials via a Secret.
See `04_security_QA.md` Q23 and `05_data_transformation_QA.md` Q15.

### 49. **B** — New column added to target; existing rows get NULL
`append_new_columns` adds columns to target without rebuilding. Existing rows get NULL for the new column.
See `05_data_transformation_QA.md` Q7.

### 50. **A** — MERGE with hash check on tracked columns
Add `WHERE d.hash <> s.hash` (SHA2 over tracked columns) to the MATCHED clause so unchanged dim attributes don't trigger history writes.
See `05_data_transformation_QA.md` Q6.

### 51. **A** — `PACKAGES = ('pandas==1.5.3')`
Pin versions in the CREATE FUNCTION declaration. Treats dependencies as deployment artifacts.
See `05_data_transformation_QA.md` Q22.

### 52. **C** — `SQLROWCOUNT`
Variable set by the last DML inside Snowflake Scripting. SQLSTATE and SQLERRM carry error info, not counts.
See `05_data_transformation_QA.md` Q9.

### 53. **B** — Call succeeds; GRANT runs with owner's privileges
`EXECUTE AS OWNER` elevates the procedure's effective privileges to the owner's. DEV only needs USAGE on the procedure.
See `05_data_transformation_QA.md` Q21 and `04_security_QA.md` Q20.

### 54. **C** — `.collect()` / `.to_pandas()`
Pulls all result rows to the Python client; OOM risk on large results. `.show()` samples; `.save_as_table()` streams to warehouse write.
See `05_data_transformation_QA.md` Q2.

### 55. **A, C** — Snowflake-managed = full DML/TT; stored in your cloud storage
Externally-managed Iceberg is often (but not always) read-only — depends on catalog type. Time travel on Iceberg uses Iceberg snapshots, not Snowflake TT bytes. Spark/Trino interop works via Iceberg format.
See `05_data_transformation_QA.md` Q5.

### 56. **C** — Python
Snowpark Python SPROCs can import Python packages (`PACKAGES=`) and use Snowpark DataFrame API. SQL Scripting is pure SQL; JavaScript is legacy with no package support.
See `05_data_transformation_QA.md` Q3 and Q10.

### 57. **C** — USERADMIN
USERADMIN specifically owns user/role lifecycle, granted via SECURITYADMIN. Minimizes blast radius.
See `04_security_QA.md` Q1.

### 58. **B** — Masking policy + ALTER TABLE SET MASKING POLICY
Masking policies evaluate per-row at query time, returning masked or raw value based on context. Views and GRANTs don't provide per-row dynamic masking.
See `04_security_QA.md` Q3.

### 59. **B, C** — Returns BOOLEAN; can reference entitlement tables
Standard RAP returns BOOLEAN per row and can query mapping tables. One policy per table attachment (multi-RAP was recent and not universally testable); doesn't modify values (that's masking); owners don't bypass.
See `04_security_QA.md` Q4 and Q15.

### 60. **B** — Union of all granted roles for authorization; primary role still owns new objects
`USE SECONDARY ROLES ALL` adds all granted roles to the session's privilege union, but ownership on CREATE still goes to the primary role.
See `04_security_QA.md` Q14.

### 61. **B** — Masking policy automatically applies
Tag-based masking policies apply to every column with the tag — add the tag to a new column and the policy fires automatically.
See `04_security_QA.md` Q9 and Q17.

### 62. **C** — Key-pair auth
Snowpipe REST and Streaming require key-pair auth (RSA). Password auth is explicitly disallowed for these endpoints.
See `04_security_QA.md` Q11 and `01_data_movement_QA.md` Q6.

### 63. **C** — `ACCESS_HISTORY` with FLATTEN on `base_objects_accessed`
ACCESS_HISTORY resolves views and projections down to base-table columns actually touched. QUERY_HISTORY has only text; GRANTS_TO_ROLES shows entitlements not actual access.
See `04_security_QA.md` Q8.

### 64. **B** — SECURE VIEW + GRANT TO SHARE
Secure views hide the definition from non-owners (DESC, EXPLAIN). Required for views exposed in data shares to prevent logic leakage.
See `04_security_QA.md` Q16 and `03_storage_and_data_protection_QA.md` Q23.

### 65. **B** — PAUSED_BY_SYSTEM; re-grant ownership and resume
Dropping the owner role puts the pipe into PAUSED_BY_SYSTEM. Fix: transfer ownership to a valid role and `ALTER PIPE RESUME`.
See `01_data_movement_QA.md` Q16.

---

## SCORING

- 59–65 correct (90%+): You're ready. Schedule the exam.
- 52–58 correct (80–89%): Review wrong-answer domain Q&A deep dives; retake in 3 days.
- 46–51 correct (70–79%): At pass line. Shore up weakest domain. Retake in a week.
- <46 correct (<70%): Return to Phase 2 of the README; focus on your lowest-weight domain performance.

Difficulty mix: 10 [F], 32 [I], 23 [A]. Real exam leans a touch more intermediate; this mock weights advanced slightly to prepare for the hardest end.
