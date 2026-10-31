# SnowPro Core (COF-C02) — Flashcards

Format: **Q** on one line, **A** on the next. Fold a sheet of paper over the A column and drill. Trivia-heavy on purpose — this is what the exam actually asks.

Organized into sections, but review in random order after your first pass.

---

## Section 1 — Architecture & Fundamentals

**Q1.** Name Snowflake's three architectural layers.
**A.** Cloud Services, Compute (Virtual Warehouses), Centralized Storage.

**Q2.** What architecture style does Snowflake use?
**A.** Hybrid — shared-disk for storage + shared-nothing MPP for compute.

**Q3.** What size is a Snowflake micro-partition?
**A.** ~16 MB compressed (50–500 MB uncompressed logical).

**Q4.** Are micro-partitions mutable?
**A.** No — they are immutable; UPDATE/DELETE writes new partitions.

**Q5.** What format are micro-partitions stored in?
**A.** Columnar, compressed, encrypted.

**Q6.** Which cloud providers run Snowflake?
**A.** AWS, Azure, GCP (not Oracle, IBM, or Alibaba Cloud).

**Q7.** What storage backend does Snowflake use on AWS/Azure/GCP?
**A.** S3 / Azure Blob Storage / Google Cloud Storage respectively.

**Q8.** Can a single account span multiple cloud providers?
**A.** No — one account lives in exactly one cloud + region. Use replication for cross-region/cloud.

**Q9.** List the Snowflake editions in increasing order.
**A.** Standard → Enterprise → Business Critical → Virtual Private Snowflake (VPS).

**Q10.** Which edition first introduces multi-cluster warehouses?
**A.** Enterprise.

**Q11.** Which edition first introduces Tri-Secret Secure (CMK)?
**A.** Business Critical.

**Q12.** Which edition has a dedicated metadata store?
**A.** VPS only.

**Q13.** How much of daily compute does Cloud Services cost before billing?
**A.** First 10% is free; billed only above the threshold.

**Q14.** Which operations do NOT require a running warehouse?
**A.** Metadata-only: SHOW/DESC, SELECT CURRENT_TIMESTAMP(), SELECT 1, certain MIN/MAX/COUNT served from metadata.

**Q15.** What are the two Snowflake account identifier forms?
**A.** `<orgname>-<account_name>` (preferred) and `<account_locator>.<region>.<cloud>` (legacy).

**Q16.** Name the four Snowflake interfaces.
**A.** Snowsight (web UI), SnowSQL (CLI), drivers (JDBC/ODBC/Python/Node/.NET/Go/PHP/Spark), SQL REST API.

**Q17.** What TLS version does Snowflake require for connections?
**A.** TLS 1.2 or higher (mandatory, not configurable).

**Q18.** What is an ORGANIZATION in Snowflake?
**A.** Logical grouping of multiple accounts under one customer; managed by ORGADMIN.

**Q19.** Who manages org-level operations?
**A.** ORGADMIN role (cannot access data in child accounts, only administer).

**Q20.** What enables zero-copy cloning architecturally?
**A.** Micro-partition immutability — clones just copy pointers to the same partitions.

---

## Section 2 — Virtual Warehouses

**Q21.** List all warehouse sizes.
**A.** XS, S, M, L, XL, 2XL, 3XL, 4XL, 5XL, 6XL.

**Q22.** How many credits per hour does an XS warehouse consume?
**A.** 1 credit/hour.

**Q23.** How many credits per hour does a 2XL warehouse consume?
**A.** 32 credits/hour.

**Q24.** How many credits per hour does a 6XL warehouse consume?
**A.** 512 credits/hour.

**Q25.** How do credits scale with warehouse size?
**A.** Each size step doubles the credits/hour (1, 2, 4, 8, 16, 32, 64, 128, 256, 512).

**Q26.** What is the billing increment and minimum per resume?
**A.** Per-second billing with a 60-second minimum each time the warehouse resumes.

**Q27.** What is the maximum number of clusters in a multi-cluster warehouse?
**A.** 10.

**Q28.** What edition is required for multi-cluster warehouses?
**A.** Enterprise or higher.

**Q29.** What is the minimum value for AUTO_SUSPEND via SQL?
**A.** 60 seconds.

**Q30.** What are the two scaling policies and which is default?
**A.** STANDARD (default, performance) and ECONOMY (cost-saving).

**Q31.** Which scaling policy favors low queue latency for BI?
**A.** STANDARD — spins new clusters immediately when queueing starts.

**Q32.** When does ECONOMY policy spin up another cluster?
**A.** When Snowflake estimates ~6 minutes of queued work would benefit.

**Q33.** What is a "maximized" multi-cluster warehouse?
**A.** MIN_CLUSTER_COUNT = MAX_CLUSTER_COUNT (and > 1); all clusters always on.

**Q34.** What is the default MAX_CONCURRENCY_LEVEL per warehouse?
**A.** 8.

**Q35.** What is the default STATEMENT_TIMEOUT_IN_SECONDS?
**A.** 172800 (48 hours).

**Q36.** What is the default STATEMENT_QUEUED_TIMEOUT_IN_SECONDS?
**A.** 0 (no limit — queries queue forever).

**Q37.** What does AUTO_RESUME=TRUE do?
**A.** Automatically starts a suspended warehouse when a query is submitted.

**Q38.** What happens when AUTO_RESUME=FALSE and a query hits a suspended warehouse?
**A.** The query fails (warehouse not running).

**Q39.** When you resize a warehouse LARGER, what happens to running queries?
**A.** Running queries continue on old resources; queued queries can use new size immediately.

**Q40.** When you resize a warehouse SMALLER, what happens?
**A.** Snowflake waits for running queries to finish on old resources before releasing them.

**Q41.** What role is required to create a resource monitor?
**A.** ACCOUNTADMIN only.

**Q42.** What are the resource monitor trigger actions?
**A.** NOTIFY, SUSPEND, SUSPEND_IMMEDIATE.

**Q43.** What's the difference between SUSPEND and SUSPEND_IMMEDIATE on a resource monitor?
**A.** SUSPEND lets running queries finish; SUSPEND_IMMEDIATE cancels them.

**Q44.** What are the valid resource monitor frequencies?
**A.** DAILY, WEEKLY, MONTHLY, YEARLY, NEVER.

**Q45.** Does the warehouse local SSD cache persist across suspension?
**A.** Not reliably — treat as lost on suspend (may linger briefly if resumed quickly).

**Q46.** Is the local SSD cache shared across warehouses?
**A.** No — it is per-warehouse, per-cluster.

**Q47.** Does resizing a warehouse clear the local cache?
**A.** No — cache is preserved on existing nodes; new nodes added start cold.

**Q48.** What is a Snowpark-optimized warehouse's minimum size?
**A.** Medium.

**Q49.** What is a Snowpark-optimized warehouse's credit rate vs standard?
**A.** Approximately 1.5x the standard rate per size.

**Q50.** How do you solve a high-concurrency problem (500 small queries)?
**A.** Scale out — multi-cluster warehouse, not a bigger single warehouse.

---

## Section 3 — Storage & Data Protection

**Q51.** What is Time Travel?
**A.** The ability to query/clone/restore data as of a prior point within a configurable retention window.

**Q52.** What is the maximum Time Travel retention on Standard edition?
**A.** 1 day.

**Q53.** What is the maximum Time Travel retention on Enterprise+?
**A.** 90 days.

**Q54.** What is the default DATA_RETENTION_TIME_IN_DAYS for permanent tables?
**A.** 1 day.

**Q55.** What does DATA_RETENTION_TIME_IN_DAYS = 0 do?
**A.** Disables Time Travel for that object (but Fail-safe still applies to permanent tables).

**Q56.** What is Fail-safe?
**A.** A 7-day disaster recovery window after Time Travel, recoverable only by Snowflake Support.

**Q57.** Can Fail-safe be configured or disabled?
**A.** No. It is a fixed 7 days on permanent tables and cannot be changed.

**Q58.** Which tables do NOT have Fail-safe?
**A.** Transient and temporary tables.

**Q59.** Compare permanent, transient, and temporary tables.
**A.** Permanent: TT up to 90d + 7d Fail-safe; Transient: TT max 1d, no Fail-safe; Temporary: session-scoped, TT max 1d, no Fail-safe.

**Q60.** When does a temporary table disappear?
**A.** At the end of the session that created it.

**Q61.** How much total CDP protection does a permanent table with 30-day Time Travel get?
**A.** 37 days (30 TT + 7 Fail-safe).

**Q62.** What does UNDROP do?
**A.** Restores a dropped table/schema/database within its Time Travel retention window.

**Q63.** Does TRUNCATE preserve data in Time Travel?
**A.** Yes — TRUNCATE is reversible via AT/BEFORE within TT retention.

**Q64.** Is zero-copy cloning a data copy?
**A.** No — metadata-only, instant; copy-on-write when source or clone is modified.

**Q65.** Can you clone an object as of a prior time?
**A.** Yes — CLONE AT/BEFORE OFFSET/TIMESTAMP/STATEMENT within TT window.

**Q66.** What objects cannot be cloned?
**A.** External tables; internal stage files; (some pipes/streams historically).

**Q67.** Do clones inherit permissions?
**A.** No by default (except for database/schema clones where behavior varies — grants usually need reapplying).

**Q68.** Encryption algorithm Snowflake uses at rest?
**A.** AES-256.

**Q69.** Snowflake encryption key hierarchy (top to bottom)?
**A.** HSM root → account master key → table master key → file key.

**Q70.** How often is the account master key rotated by default?
**A.** Approximately annually (automatic).

**Q71.** What edition is required for Tri-Secret Secure (CMK)?
**A.** Business Critical.

**Q72.** What edition is required for periodic data rekeying?
**A.** Enterprise or higher.

**Q73.** What is MIN_DATA_RETENTION_TIME_IN_DAYS?
**A.** Account-level floor on TT retention; actual retention = MAX(min, object setting).

**Q74.** What edition is required for basic database replication?
**A.** Standard (available on all editions).

**Q75.** What edition is required for failover groups / database failover?
**A.** Business Critical.

**Q76.** What is NOT replicated by basic database replication?
**A.** Users, roles, grants, pipes, tasks, network policies, resource monitors (require failover groups).

**Q77.** What is the account-level parameter that enforces a minimum TT retention?
**A.** MIN_DATA_RETENTION_TIME_IN_DAYS (ACCOUNTADMIN sets).

---

## Section 4 — Data Movement

**Q78.** Name the four stage types.
**A.** User (`@~`), Table (`@%<table>`), Named Internal (`@<stage>`), Named External (`@<stage>` with URL).

**Q79.** What is the prefix for a user stage?
**A.** `@~`

**Q80.** What is the prefix for a table stage?
**A.** `@%<table_name>`

**Q81.** Can user and table stages be altered or dropped?
**A.** No — they are implicit and always exist.

**Q82.** What does PUT do?
**A.** Uploads local files to an internal stage. Works from SnowSQL/drivers; NOT in Snowsight.

**Q83.** What does GET do?
**A.** Downloads files from an internal stage to local disk. SnowSQL/drivers only.

**Q84.** Can PUT upload to an external stage?
**A.** No — PUT only targets internal stages.

**Q85.** How long does Snowflake's load history dedupe loaded files?
**A.** 64 days.

**Q86.** How do you force COPY INTO to reload already-loaded files?
**A.** Use `FORCE = TRUE` option.

**Q87.** What are the valid ON_ERROR options for COPY INTO?
**A.** CONTINUE, SKIP_FILE, SKIP_FILE_<n>, SKIP_FILE_<n>%, ABORT_STATEMENT (default).

**Q88.** What is the default ON_ERROR for COPY INTO?
**A.** ABORT_STATEMENT.

**Q89.** What does PURGE=TRUE do in COPY INTO?
**A.** Deletes source files from the stage after successful load.

**Q90.** What is Snowpipe?
**A.** Serverless continuous data ingestion service that runs COPY INTO automatically on new files.

**Q91.** How is Snowpipe billed?
**A.** Serverless credits (not warehouse credits) + a small per-file overhead.

**Q92.** What file size does Snowpipe prefer for optimal throughput?
**A.** 100–250 MB compressed per file.

**Q93.** What is the typical latency of Snowpipe auto-ingest?
**A.** About 1 minute (can be longer under load).

**Q94.** Which file formats does Snowflake natively support?
**A.** CSV/TSV, JSON, Avro, ORC, Parquet, XML.

**Q95.** What is a Stream?
**A.** A change data capture (CDC) object that tracks DML on a source table since the last offset.

**Q96.** What are the three stream types?
**A.** Standard, Append-only, Insert-only (external tables).

**Q97.** How does a stream offset advance?
**A.** Only when consumed in a DML (INSERT/MERGE/UPDATE/DELETE); SELECT alone does not advance it.

**Q98.** What is a Task?
**A.** A scheduled SQL/sproc execution unit; supports cron or interval schedules; can chain via AFTER.

**Q99.** What state does a newly created task start in?
**A.** Suspended — must `ALTER TASK ... RESUME` to run.

**Q100.** What is a Dynamic Table?
**A.** Declarative materialized result with a TARGET_LAG freshness SLA; replaces much Stream+Task plumbing.

**Q101.** Does Snowpipe use OWNER or CALLER rights?
**A.** OWNER's rights (runs with the pipe owner's privileges).

**Q102.** What are the metadata columns on a stream?
**A.** METADATA$ACTION, METADATA$ISUPDATE, METADATA$ROW_ID.

**Q103.** Name the implicit metadata columns in external tables.
**A.** METADATA$FILENAME, METADATA$FILE_ROW_NUMBER (and VALUE for raw row content).

**Q104.** Can external tables be loaded with COPY INTO?
**A.** No — external tables are read-only; use INSERT ... SELECT to materialize into a native table.

**Q105.** What is a Storage Integration?
**A.** Snowflake object that abstracts cloud credentials for external stage access; preferred over inline creds.

---

## Section 5 — Caching

**Q106.** List the four caching layers (fastest to slowest).
**A.** Result cache → Metadata cache → Warehouse local SSD cache → Remote storage.

**Q107.** How long does the Result Cache persist?
**A.** 24 hours base TTL; extended per hit up to 31 days max.

**Q108.** Does the Result Cache require the original warehouse to be running?
**A.** No — it is served by Cloud Services and is cross-warehouse / cross-session.

**Q109.** What invalidates the Result Cache?
**A.** Any change to underlying micro-partitions; non-deterministic functions; some role/context changes.

**Q110.** Which Snowflake functions bypass the Result Cache?
**A.** Non-deterministic: CURRENT_TIMESTAMP, CURRENT_DATE, RANDOM, UUID_STRING, etc.

**Q111.** Which session parameter disables the Result Cache?
**A.** `USE_CACHED_RESULT = FALSE` (default is TRUE).

**Q112.** What does the Metadata Cache enable?
**A.** Instant COUNT(*)/MIN/MAX from stats; partition pruning decisions.

**Q113.** Which caches survive warehouse suspension?
**A.** Result cache and Metadata cache (both in Cloud Services). Local SSD does not reliably persist.

**Q114.** Are local SSD caches shared across clusters in a multi-cluster warehouse?
**A.** No — each cluster has its own cache.

---

## Section 6 — Performance Features

**Q115.** What is partition pruning?
**A.** Skipping micro-partitions whose metadata (min/max) guarantees no matching rows.

**Q116.** What SQL patterns defeat partition pruning?
**A.** Functions on columns (UPPER(col), DATE(ts)), leading wildcards (LIKE '%x%'), casts that hide semantics.

**Q117.** What is a clustering key?
**A.** Column(s)/expression(s) hint used by Automatic Clustering to co-locate data for better pruning.

**Q118.** How is Automatic Clustering billed?
**A.** Serverless credits (separate from warehouse credits).

**Q119.** How do you check clustering health?
**A.** `SELECT SYSTEM$CLUSTERING_INFORMATION('table', '(col)');` — look at average_depth and average_overlaps.

**Q120.** What is an ideal clustering depth value?
**A.** Close to 1 (lower is better).

**Q121.** What is a Materialized View?
**A.** Pre-computed query result stored and auto-refreshed by a serverless service.

**Q122.** What edition is required for Materialized Views?
**A.** Enterprise or higher.

**Q123.** Can an MV use joins or window functions?
**A.** No — MVs are limited to a single base table, no joins, no window functions, no HAVING, no non-deterministic funcs.

**Q124.** What is the Search Optimization Service (SOS)?
**A.** Serverless background index for selective point lookups (equality, substring, IN lists) on large tables.

**Q125.** What edition is required for SOS?
**A.** Enterprise or higher.

**Q126.** What is the Query Acceleration Service (QAS)?
**A.** Offloads parts of eligible queries to serverless compute for burst speed.

**Q127.** What edition is required for QAS?
**A.** Enterprise or higher.

**Q128.** What does "Remote Disk Spilling" in Query Profile mean?
**A.** Query exceeded local SSD and is spilling to remote storage — severe performance issue; resize warehouse or rewrite.

**Q129.** What does "Local Disk Spilling" indicate?
**A.** Query spilled from RAM to local SSD; moderate penalty, often tolerable.

---

## Section 7 — Security & RBAC

**Q130.** List default roles from highest to lowest privilege.
**A.** ORGADMIN → ACCOUNTADMIN → (SECURITYADMIN → USERADMIN) and (SYSADMIN) → custom → PUBLIC.

**Q131.** What does ORGADMIN manage?
**A.** Organization-level: create accounts, enable features; cannot access data in child accounts.

**Q132.** What is ACCOUNTADMIN's scope?
**A.** Account-level superuser; owns top-level objects; manages billing + resource monitors.

**Q133.** What is the difference between SECURITYADMIN and USERADMIN?
**A.** USERADMIN manages users/roles only; SECURITYADMIN additionally manages grants globally and network policies.

**Q134.** Who can create a resource monitor?
**A.** ACCOUNTADMIN only.

**Q135.** What is PUBLIC role?
**A.** Default role every user and every role has; grants to PUBLIC = everyone.

**Q136.** What is SYSADMIN typically used for?
**A.** Owning data assets (warehouses, databases, schemas); recommended top-level role for data engineering.

**Q137.** Does SYSADMIN include SECURITYADMIN privileges?
**A.** No — SYSADMIN handles data; SECURITYADMIN handles identity/permissioning.

**Q138.** List supported authentication methods.
**A.** Password (+MFA), key-pair JWT, SAML2 SSO, external OAuth, Snowflake OAuth.

**Q139.** What MFA provider is bundled with Snowflake?
**A.** Duo Security (free, native).

**Q140.** How do you enforce MFA for a user?
**A.** `ALTER USER <u> SET MFA_ENROLLMENT_REQUIRED = TRUE;`

**Q141.** What is a Network Policy?
**A.** IP allowlist/blocklist applied at account or user level to control login origin.

**Q142.** Which takes precedence: ALLOWED_IP_LIST or BLOCKED_IP_LIST?
**A.** BLOCKED_IP_LIST wins.

**Q143.** Which IP version do network policies support?
**A.** IPv4 only (per exam syllabus).

**Q144.** What edition is required for AWS/Azure/GCP PrivateLink?
**A.** Business Critical or higher.

**Q145.** What is Dynamic Data Masking?
**A.** Policy-based column-level masking applied at query time based on invoking role; Enterprise+.

**Q146.** What is a Row Access Policy?
**A.** Policy returning boolean filter per row based on role/mapping-table context; Enterprise+.

**Q147.** What is a Secure View?
**A.** View that hides definition from non-owners and disables optimizations that could leak data.

**Q148.** What is a Session Policy?
**A.** Controls session idle timeout and UI timeout; Enterprise+ edition.

**Q149.** What privilege lets a role grant anything?
**A.** `MANAGE GRANTS` (held by ACCOUNTADMIN and SECURITYADMIN).

**Q150.** What are secondary roles?
**A.** Additional roles active in a session alongside the primary CURRENT_ROLE; combine privileges.

---

## Section 8 — Data Transformations & SQL

**Q151.** What is a VARIANT column?
**A.** Semi-structured storage for JSON/Avro/Parquet/ORC/XML; max 16 MB per value compressed.

**Q152.** What is the max size of a single VARIANT value?
**A.** 16 MB compressed.

**Q153.** List three ways to access a VARIANT field named "user".
**A.** `payload:user`, `payload.user`, `payload['user']`.

**Q154.** What operator casts a VARIANT value to a scalar type?
**A.** `::` (double colon), e.g., `payload:age::INT`.

**Q155.** What does LATERAL FLATTEN do?
**A.** Explodes a VARIANT/ARRAY/OBJECT into one row per element; LATERAL joins back to parent.

**Q156.** What does FLATTEN's `OUTER => TRUE` option do?
**A.** Emits a row even when the input array is NULL/empty (LEFT JOIN semantics).

**Q157.** FLATTEN output columns?
**A.** SEQ, KEY, PATH, INDEX, VALUE, THIS.

**Q158.** Difference between PARSE_JSON and TO_JSON?
**A.** PARSE_JSON turns STRING → VARIANT; TO_JSON turns VARIANT → STRING.

**Q159.** What is QUALIFY?
**A.** Filters rows based on window function output (analog to HAVING for aggregates).

**Q160.** Order of SQL clauses: WHERE, GROUP BY, HAVING, QUALIFY?
**A.** WHERE → GROUP BY → HAVING → QUALIFY (window functions run before QUALIFY).

**Q161.** Difference between UDF and stored procedure?
**A.** UDFs are pure functions callable in SELECT (no DML); sprocs are imperative, called with CALL, can execute DML/DDL.

**Q162.** What languages are supported for UDFs?
**A.** SQL, JavaScript, Python, Java, Scala.

**Q163.** Default EXECUTE AS mode for sprocs?
**A.** OWNER's rights (CALLER's rights optional via `EXECUTE AS CALLER`).

**Q164.** What is Snowflake Scripting?
**A.** SQL procedural extension (BEGIN/END, variables, loops, cursors, exception handling) inside SQL sprocs.

**Q165.** Are sequences gap-free?
**A.** No — concurrent sessions may skip numbers.

**Q166.** Snowflake equivalent of GROUP_CONCAT / COLLECT_LIST?
**A.** `LISTAGG(col, ',') WITHIN GROUP (ORDER BY col)` or `ARRAY_AGG(col)`.

**Q167.** Function for fast approximate distinct count?
**A.** `APPROX_COUNT_DISTINCT(col)` (uses HyperLogLog).

**Q168.** What is a Recursive CTE?
**A.** `WITH RECURSIVE` syntax for hierarchical/graph traversal (anchor + recursive member united by UNION ALL).

**Q169.** TIMESTAMP_LTZ vs TIMESTAMP_NTZ vs TIMESTAMP_TZ?
**A.** LTZ = local timezone (session), NTZ = no timezone, TZ = with timezone offset stored.

**Q170.** Default TIMESTAMP flavor for table columns?
**A.** TIMESTAMP_NTZ (unless account/session parameter overrides).

---

## Section 9 — Serverless + Credit Features

**Q171.** List the serverless features in Snowflake.
**A.** Snowpipe, Snowpipe Streaming, Automatic Clustering, Materialized View maintenance, Search Optimization Service, Replication, Serverless Tasks.

**Q172.** How are serverless features billed?
**A.** Separate line items on the invoice; credits per compute-second, not from your warehouses.

**Q173.** Which separate credit lines appear on a Snowflake bill?
**A.** Warehouse compute, Cloud services (billed only above 10% threshold), Snowpipe, Automatic Clustering, Materialized Views, SOS, Replication, Serverless Tasks, Query Acceleration.

---

## Section 10 — Trivia Lightning Round

**Q174.** How many questions on COF-C02?
**A.** 100.

**Q175.** How many minutes to complete the exam?
**A.** 115 minutes.

**Q176.** What is the passing scaled score?
**A.** 750 out of 1000.

**Q177.** What edition does Secure Data Sharing start at?
**A.** Standard (basic sharing is available on all editions).

**Q178.** Can Snowsight run PUT or GET?
**A.** No — PUT/GET are client-side via SnowSQL or drivers.

**Q179.** What's the max row size shown by a SHOW command?
**A.** 10,000 rows (from metadata).

**Q180.** What SYSTEM$ function shows account-permitted IPs?
**A.** `SYSTEM$ALLOWLIST()` or `SYSTEM$WHITELIST()`.

**Q181.** What SYSTEM$ function checks if a stream has data?
**A.** `SYSTEM$STREAM_HAS_DATA('stream_name')`.

**Q182.** What SYSTEM$ function returns clustering info?
**A.** `SYSTEM$CLUSTERING_INFORMATION('table', '(col)')`.

**Q183.** What SYSTEM$ function returns the current Snowflake edition?
**A.** `SYSTEM$GET_SNOWFLAKE_EDITION()`.

**Q184.** AT vs BEFORE in Time Travel queries?
**A.** AT is inclusive of the moment; BEFORE is just before.

**Q185.** Three ways to specify a Time Travel point?
**A.** `TIMESTAMP => '...'`, `OFFSET => -<seconds>`, `STATEMENT => '<query_id>'`.

**Q186.** What's the max concurrent clusters * max warehouse size?
**A.** 10 clusters × 6X-Large (512 credits/hr) = up to 5,120 credits/hr theoretical.

**Q187.** Which Snowflake account URL doesn't include region (legacy quirk)?
**A.** AWS us-west-2 (Oregon) accounts in legacy locator form.

**Q188.** Secondary roles activation command?
**A.** `USE SECONDARY ROLES ALL;` (or specific ones; `NONE` to disable).

**Q189.** Can you grant SELECT on all tables in a schema in one statement?
**A.** Yes — `GRANT SELECT ON ALL TABLES IN SCHEMA <schema> TO ROLE <r>;` (plus FUTURE TABLES for new ones).

**Q190.** What does GRANT ... ON FUTURE TABLES do?
**A.** Applies the privilege to tables created after the grant was made.

**Q191.** Which role owns the SNOWFLAKE database (ACCOUNT_USAGE schema)?
**A.** Shared via `IMPORTED PRIVILEGES`; ACCOUNTADMIN by default; delegable to others via grant.

**Q192.** What is a Reader Account?
**A.** A no-cost Snowflake account provisioned by a provider for consumers who don't have Snowflake.

**Q193.** Where do you look at Snowpipe credit usage?
**A.** `SNOWFLAKE.ACCOUNT_USAGE.PIPE_USAGE_HISTORY`.

**Q194.** Where do you look at automatic clustering history?
**A.** `SNOWFLAKE.ACCOUNT_USAGE.AUTOMATIC_CLUSTERING_HISTORY`.

**Q195.** Where do you look at warehouse credit usage?
**A.** `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY`.

**Q196.** How many days back does ACCOUNT_USAGE schema retain data?
**A.** 365 days (1 year).

**Q197.** How much latency does ACCOUNT_USAGE have vs real-time?
**A.** Up to 3 hours (INFORMATION_SCHEMA is more real-time but shorter retention).

**Q198.** INFORMATION_SCHEMA retention vs ACCOUNT_USAGE?
**A.** INFORMATION_SCHEMA: 7 days to 6 months depending on view; near real-time. ACCOUNT_USAGE: 365 days, ~3h delay.

**Q199.** What is the quickest way to see which cache a query used?
**A.** `percentage_scanned_from_cache` column in query history, or Query Profile view.

**Q200.** How many file formats can one COPY INTO reference?
**A.** One — specified inline or via FORMAT_NAME; COPY options override stage format if both set.

---

Drill tip: After passing all 200 cold, shuffle and drill again. The exam asks these facts in disguised wording — recognize the fact fast enough to earn the mark.
