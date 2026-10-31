# SnowPro Core (COF-C02) — Full-Form Mock Exam

**100 questions · 115 minutes · passing threshold ~70% raw (750/1000 scaled)**

Format matches real exam: single-select unless the question says "Which TWO" or "Which THREE". Multi-select requires ALL correct options; no partial credit.

Take this timed, closed-book, on a single sitting. Answers and explanations begin after Question 100.

---

## Section A: Architecture & Overview (Questions 1–20)

**1.** Which of the following best describes Snowflake's architecture?
- A. Pure shared-nothing MPP
- B. Pure shared-disk
- C. Hybrid: shared-disk storage + shared-nothing compute
- D. Serverless function-based

**2.** What is the approximate compressed size of a Snowflake micro-partition?
- A. 1 MB
- B. 16 MB
- C. 50 MB
- D. 500 MB

**3.** Which THREE of the following statements about micro-partitions are TRUE?
- A. They are columnar
- B. They are mutable; UPDATE rewrites them in place
- C. They are immutable
- D. They are automatically encrypted and compressed
- E. Users define their size via DDL

**4.** Which Snowflake edition is the minimum required for multi-cluster warehouses?
- A. Standard
- B. Enterprise
- C. Business Critical
- D. Virtual Private Snowflake

**5.** Which TWO of the following features require Business Critical edition or higher?
- A. Materialized Views
- B. Tri-Secret Secure
- C. AWS PrivateLink
- D. Column-level security
- E. Multi-cluster warehouses

**6.** Snowflake runs natively on which THREE cloud providers?
- A. AWS
- B. Oracle Cloud
- C. Azure
- D. GCP
- E. IBM Cloud

**7.** Which statement about the Cloud Services layer is TRUE?
- A. It is billed hourly regardless of usage
- B. The first 10% of daily compute cost is free, billed only above
- C. It requires a dedicated warehouse
- D. It executes DML queries

**8.** Which of the following queries will typically NOT require a running warehouse?
- A. SELECT COUNT(*) FROM huge_table WHERE col='x'
- B. SELECT * FROM huge_table LIMIT 1000
- C. SHOW TABLES IN SCHEMA public
- D. INSERT INTO t SELECT * FROM s

**9.** What is the preferred account identifier format in Snowflake?
- A. `<locator>.<region>.<cloud>`
- B. `<orgname>-<account_name>`
- C. `<cloud>-<account>`
- D. IP address

**10.** Which Snowflake interface CANNOT run the `PUT` command?
- A. SnowSQL
- B. Python connector
- C. Snowsight web UI
- D. JDBC driver

**11.** Which THREE objects are considered first-class in Snowflake's data sharing model?
- A. Shares
- B. Reader accounts
- C. Databases (via GRANT ... TO SHARE)
- D. Local tables on consumer
- E. Views (via GRANT ... TO SHARE)

**12.** Which THREE statements about the Snowflake Marketplace are TRUE?
- A. Data is physically copied to the consumer's storage
- B. Listings can be free, paid, or personalized
- C. Access to a listing creates a shared database on the consumer
- D. Updates on the provider are visible immediately to the consumer
- E. Marketplace requires Enterprise edition

**13.** On which cloud platform does Snowflake use S3 as its underlying storage?
- A. GCP
- B. AWS
- C. Azure
- D. All three

**14.** True or False: A single Snowflake account can span multiple regions simultaneously.
- A. True
- B. False

**15.** Which TWO of the following are responsibilities of the Cloud Services layer?
- A. Query optimization
- B. Executing joins and aggregations
- C. Access control checks
- D. Caching data in local SSD
- E. Persistent data storage

**16.** Which role is required to manage organization-level objects such as creating new accounts?
- A. ACCOUNTADMIN
- B. SECURITYADMIN
- C. ORGADMIN
- D. SYSADMIN

**17.** Which TWO of the following are TRUE about zero-copy cloning?
- A. It is a metadata-only operation
- B. The clone is read-only
- C. Modifications to the clone trigger copy-on-write
- D. Cloning doubles the storage cost immediately
- E. Cloning cannot be done on a database

**18.** What is Snowsight?
- A. The command-line interface for Snowflake
- B. The web-based UI for Snowflake
- C. A Snowflake connector for Python
- D. A data-ingestion service

**19.** Which TWO statements about Snowflake storage billing are TRUE?
- A. Storage is billed on uncompressed data size
- B. Storage is billed on compressed data size, averaged daily
- C. Time Travel storage is billed as part of total storage
- D. Fail-safe storage is free
- E. Storage cost depends on warehouse size

**20.** Which SYSTEM$ function returns the current Snowflake edition?
- A. `SYSTEM$GET_EDITION()`
- B. `SYSTEM$CURRENT_EDITION()`
- C. `SYSTEM$GET_SNOWFLAKE_EDITION()`
- D. `SYSTEM$EDITION()`

---

## Section B: Virtual Warehouses (Questions 21–40)

**21.** How many credits per hour does a Large (L) warehouse consume?
- A. 4
- B. 8
- C. 16
- D. 32

**22.** How many credits per hour does a 4X-Large warehouse consume?
- A. 32
- B. 64
- C. 128
- D. 256

**23.** What is the minimum AUTO_SUSPEND value configurable via SQL?
- A. 1 second
- B. 30 seconds
- C. 60 seconds
- D. 600 seconds

**24.** Snowflake charges for warehouse compute with what billing increment and minimum?
- A. Per-minute, 1-minute minimum
- B. Per-hour, 1-hour minimum
- C. Per-second, 60-second minimum per resume
- D. Per-query, no minimum

**25.** What is the maximum number of clusters supported in a multi-cluster warehouse?
- A. 5
- B. 8
- C. 10
- D. 16

**26.** Which TWO of the following statements about scaling policies are TRUE?
- A. STANDARD is the default and favors performance
- B. ECONOMY starts new clusters immediately
- C. ECONOMY may queue queries longer to save credits
- D. STANDARD is only available in Enterprise+
- E. Scaling policy applies to single-cluster warehouses too

**27.** A user has 500 concurrent small queries from a BI tool. Which scaling approach is best?
- A. Resize warehouse to 6XL
- B. Enable Query Acceleration Service
- C. Add more clusters (horizontal/multi-cluster)
- D. Use a serverless task

**28.** What is a "maximized" multi-cluster warehouse?
- A. One with MAX_CLUSTER_COUNT=10
- B. One with MIN_CLUSTER_COUNT = MAX_CLUSTER_COUNT and > 1
- C. One with WAREHOUSE_SIZE='6XL'
- D. One with INITIALLY_SUSPENDED=FALSE

**29.** What is the default value of MAX_CONCURRENCY_LEVEL?
- A. 4
- B. 8
- C. 10
- D. 16

**30.** What happens when a running warehouse receives a resize command to a LARGER size?
- A. All running queries are cancelled
- B. Running queries continue on old resources; new size is used for queued queries
- C. The warehouse is suspended and restarted
- D. Nothing until the next AUTO_SUSPEND cycle

**31.** Which THREE queue states can appear in query history?
- A. QUEUED_PROVISIONING
- B. QUEUED_OVERLOADED
- C. QUEUED_REPAIR
- D. QUEUED_SLEEPING
- E. QUEUED_IDLE

**32.** Who is the ONLY default role that can create a Resource Monitor?
- A. SYSADMIN
- B. SECURITYADMIN
- C. ACCOUNTADMIN
- D. ORGADMIN

**33.** Which THREE trigger actions can a Resource Monitor perform?
- A. NOTIFY
- B. SUSPEND
- C. SUSPEND_IMMEDIATE
- D. DELETE_WAREHOUSE
- E. SCALE_DOWN

**34.** Which TWO statements about the warehouse local SSD cache are TRUE?
- A. It is shared across all warehouses in the account
- B. It is per-warehouse (per-cluster)
- C. It survives warehouse suspension reliably
- D. It may be lost when the warehouse is suspended
- E. It is managed by the Cloud Services layer

**35.** What is the default state of a newly created warehouse (without INITIALLY_SUSPENDED)?
- A. Suspended
- B. Started/running
- C. Repairing
- D. Suspending

**36.** Which TWO statements about Snowpark-optimized warehouses are TRUE?
- A. They have higher memory per node than standard warehouses
- B. Credit rate is approximately 1.5x standard
- C. Minimum size is X-Small
- D. Minimum size is Medium
- E. They are serverless

**37.** What is the default STATEMENT_TIMEOUT_IN_SECONDS?
- A. 60
- B. 3600
- C. 86400
- D. 172800

**38.** Which TWO ALTER WAREHOUSE actions do NOT disrupt running queries?
- A. Scaling UP to a larger size
- B. SUSPEND
- C. ABORT ALL QUERIES
- D. Changing SCALING_POLICY
- E. DROP WAREHOUSE

**39.** Which THREE actions improve local SSD cache hit rate?
- A. Dedicating a warehouse per workload
- B. Setting AUTO_SUSPEND to 60 seconds (most aggressive)
- C. Keeping warehouses warm longer (higher AUTO_SUSPEND)
- D. Random warehouse rotation
- E. Using a maximized multi-cluster for predictable workloads

**40.** Which Snowflake feature requires Enterprise+ edition and offloads parts of eligible queries to serverless compute?
- A. Snowpipe
- B. Query Acceleration Service
- C. Search Optimization Service
- D. Automatic Clustering

---

## Section C: Storage, Protection, Sharing (Questions 41–55)

**41.** What is the maximum Time Travel retention on Standard edition?
- A. 1 day
- B. 7 days
- C. 30 days
- D. 90 days

**42.** What is the maximum Time Travel retention on Enterprise edition or higher?
- A. 7 days
- B. 30 days
- C. 90 days
- D. 365 days

**43.** How long is Fail-safe for permanent tables?
- A. 1 day
- B. 7 days (configurable)
- C. 7 days (fixed, non-configurable)
- D. 30 days

**44.** Which TWO statements about Fail-safe are TRUE?
- A. Users can query Fail-safe data via AT/BEFORE
- B. Fail-safe is 7 days and cannot be changed
- C. Fail-safe applies to permanent tables only
- D. Fail-safe applies to transient tables
- E. Fail-safe can be disabled by ACCOUNTADMIN

**45.** Which TWO statements about transient tables are TRUE?
- A. They persist across sessions
- B. They have Fail-safe for 7 days
- C. Maximum Time Travel is 1 day
- D. They are dropped at session end
- E. They support materialized views on them

**46.** What is the lifetime of a temporary table?
- A. 24 hours
- B. 1 day
- C. The current session
- D. Until explicitly dropped

**47.** Which THREE objects can be cloned?
- A. Databases
- B. Schemas
- C. Tables
- D. External tables
- E. Internal stage files

**48.** You accidentally dropped a critical table 2 hours ago. The table had DATA_RETENTION_TIME_IN_DAYS=5. How do you restore it?
- A. Call Snowflake support
- B. Use `UNDROP TABLE <name>`
- C. Replay the DDL from source control
- D. Restore from Fail-safe

**49.** Which TWO scenarios will consume additional Time Travel storage?
- A. SELECT queries
- B. UPDATE statements
- C. TRUNCATE operations
- D. CREATE TABLE AS SELECT (initial load)
- E. Pure INSERT of new rows (no updates)

**50.** Setting DATA_RETENTION_TIME_IN_DAYS=0 on a permanent table has what effect?
- A. Disables Time Travel AND Fail-safe
- B. Disables Time Travel; Fail-safe (7 days) still applies
- C. Reduces Fail-safe to 0 days
- D. Deletes the table immediately

**51.** What edition is required for database failover/failback (DR with promotion)?
- A. Standard
- B. Enterprise
- C. Business Critical
- D. VPS

**52.** Which TWO are NOT replicated by basic database replication (require failover groups)?
- A. Tables
- B. Users and roles
- C. Views
- D. Pipes, tasks, streams
- E. File formats

**53.** What is Tri-Secret Secure?
- A. Three-factor authentication for users
- B. A composite key encryption combining Snowflake key + customer-managed key
- C. A secure data sharing feature
- D. A network security policy

**54.** Which encryption algorithm does Snowflake use at rest?
- A. AES-128
- B. AES-256
- C. RSA-2048
- D. ChaCha20

**55.** The account-level parameter that enforces a minimum Time Travel retention across the account is:
- A. DATA_RETENTION_TIME_IN_DAYS
- B. MIN_DATA_RETENTION_TIME_IN_DAYS
- C. ACCOUNT_RETENTION_MIN
- D. TIME_TRAVEL_FLOOR

---

## Section D: Data Movement (Questions 56–75)

**56.** How many types of stages exist in Snowflake?
- A. 2
- B. 3
- C. 4
- D. 5

**57.** What is the symbol prefix for a user stage?
- A. `@%`
- B. `@~`
- C. `@`
- D. `#`

**58.** What is the symbol prefix for a table stage named ORDERS?
- A. `@~ORDERS`
- B. `@%ORDERS`
- C. `@ORDERS`
- D. `#ORDERS`

**59.** Which TWO statements about user and table stages are TRUE?
- A. They must be explicitly created
- B. They are implicit; always exist
- C. They can be altered and dropped
- D. They cannot be altered or dropped
- E. They point to external cloud buckets

**60.** Which command uploads a local file to a Snowflake stage?
- A. UPLOAD
- B. PUT
- C. COPY FROM LOCAL
- D. STAGE INSERT

**61.** Which interfaces can run the PUT command? (Pick ALL that apply conceptually — multi-select)
- A. SnowSQL CLI
- B. Snowsight web UI
- C. Python connector
- D. JDBC driver
- E. REST API /statements endpoint

**62.** How long does Snowflake's load history track loaded files to prevent re-loads?
- A. 24 hours
- B. 7 days
- C. 64 days
- D. Forever

**63.** Which option bypasses the load history check in COPY INTO?
- A. OVERRIDE=TRUE
- B. FORCE=TRUE
- C. SKIP_HISTORY=TRUE
- D. PURGE=TRUE

**64.** What is the default ON_ERROR option for COPY INTO?
- A. CONTINUE
- B. SKIP_FILE
- C. ABORT_STATEMENT
- D. SKIP_ROW

**65.** Which TWO statements about Snowpipe are TRUE?
- A. It uses the invoker's warehouse credits
- B. It is serverless and billed in Snowpipe credits
- C. It has a per-file overhead charge in addition to compute
- D. Typical latency is sub-second
- E. It requires Enterprise edition

**66.** What is the recommended file size for Snowpipe ingestion for optimal throughput?
- A. 1–10 KB
- B. 10–50 MB
- C. 100–250 MB compressed
- D. 1–5 GB compressed

**67.** Which THREE file formats does Snowflake natively support?
- A. CSV
- B. Parquet
- C. YAML
- D. JSON
- E. Protobuf

**68.** A Stream tracks DML changes. On a `SELECT * FROM mystream;` statement, what happens to the stream offset?
- A. It advances
- B. It does not advance; only consumed by DML statements
- C. It resets
- D. It is deleted

**69.** Which THREE are valid stream types?
- A. Standard
- B. Append-only
- C. Insert-only (on external tables)
- D. Update-only
- E. Delete-only

**70.** A newly created task is in which state?
- A. Running
- B. Suspended
- C. Scheduled
- D. Aborted

**71.** Which TWO statements about Dynamic Tables are TRUE?
- A. They are declarative and refresh toward a TARGET_LAG SLA
- B. They can only have one base table
- C. They can chain into DAGs
- D. They are serverless-only
- E. They require Enterprise edition

**72.** External tables are:
- A. Read-only metadata views over cloud files
- B. Fully writable regular tables
- C. Required to use Snowpipe
- D. Only supported on AWS

**73.** Which THREE objects/options are serverless features?
- A. Snowpipe
- B. Automatic Clustering
- C. Materialized View maintenance
- D. Warehouse resize
- E. ALTER TABLE SWAP

**74.** What privilege set is needed by the role running COPY INTO on a target table in a named internal stage?
- A. SELECT on table, USAGE on stage
- B. INSERT on table, USAGE on schema/database/stage/file format, USAGE on warehouse
- C. OWNERSHIP on table, USAGE on account
- D. INSERT on table only

**75.** Snowpipe executes the COPY INTO statement with whose privileges?
- A. The invoking role at ingest time
- B. The pipe's OWNER role
- C. ACCOUNTADMIN
- D. The role that created the stage

---

## Section E: Security & Access (Questions 76–85)

**76.** Which THREE actions require ACCOUNTADMIN role?
- A. Creating a Resource Monitor
- B. Creating a user
- C. Setting MIN_DATA_RETENTION_TIME_IN_DAYS
- D. Granting SELECT on a table owned by another role
- E. Approving a replication request

**77.** The role that handles user and role creation (but not grants on data objects globally) is:
- A. SYSADMIN
- B. USERADMIN
- C. SECURITYADMIN
- D. ACCOUNTADMIN

**78.** Which TWO statements about SECURITYADMIN are TRUE?
- A. It inherits USERADMIN
- B. It inherits SYSADMIN
- C. It has MANAGE GRANTS globally
- D. It can create warehouses by default
- E. It can create databases by default

**79.** Which TWO statements about the PUBLIC role are TRUE?
- A. Every user has it implicitly
- B. It has ACCOUNTADMIN-equivalent privileges
- C. Every role inherits from PUBLIC
- D. Only ACCOUNTADMIN can grant to PUBLIC
- E. PUBLIC is removed by default in new accounts

**80.** Which authentication methods does Snowflake support? (Pick THREE)
- A. Username + password + MFA (Duo)
- B. Key-pair JWT
- C. SAML2 SSO
- D. Biometric via device
- E. SMS-only authentication

**81.** You want to limit login access to corporate IPs only. What do you create?
- A. A role
- B. A network policy
- C. A session policy
- D. A masking policy

**82.** Which of the following takes precedence when a user matches both?
- A. ALLOWED_IP_LIST
- B. BLOCKED_IP_LIST
- C. Default policy
- D. None; both apply equally

**83.** Dynamic Data Masking is available starting at which edition?
- A. Standard
- B. Enterprise
- C. Business Critical
- D. VPS

**84.** Which TWO statements about Secure Views are TRUE?
- A. They hide the view definition from non-owners
- B. They are faster than regular views
- C. The optimizer suppresses certain plan optimizations to prevent data leakage
- D. They require Enterprise edition
- E. They cannot have row access policies applied

**85.** Which edition is required for AWS/Azure/GCP Private Link connectivity?
- A. Standard
- B. Enterprise
- C. Business Critical
- D. VPS only

---

## Section F: Performance (Questions 86–93)

**86.** The Result Cache base TTL and maximum extension are:
- A. 1 hour / 7 days
- B. 24 hours / 31 days
- C. 24 hours / 90 days
- D. 7 days / 1 year

**87.** Which TWO conditions must be met for a query to use the Result Cache?
- A. Identical query text and underlying data unchanged
- B. The original warehouse must still be running
- C. The user must have privileges to see all underlying data
- D. The query must have been run within the past 90 days
- E. The query must contain CURRENT_TIMESTAMP()

**88.** Which caches survive a warehouse suspension? (Pick TWO)
- A. Result cache
- B. Metadata cache
- C. Warehouse local SSD cache
- D. Remote storage cache
- E. Query compilation cache

**89.** Which TWO SQL patterns typically defeat partition pruning?
- A. `WHERE date_col = '2026-10-30'` (indexed-like column)
- B. `WHERE UPPER(name) = 'ALICE'`
- C. `WHERE event_time BETWEEN '2026-10-29' AND '2026-10-30'`
- D. `WHERE message LIKE '%error%'`
- E. `WHERE id IN (1,2,3)`

**90.** Which TWO statements about Materialized Views are TRUE?
- A. They support arbitrary SQL including joins
- B. They are maintained by Snowflake's serverless service
- C. The optimizer can transparently rewrite base queries to use MVs
- D. They are free (no credits consumed)
- E. They require Standard edition

**91.** Which Snowflake feature speeds up selective point-lookup queries via a background inverted index?
- A. Materialized Views
- B. Automatic Clustering
- C. Search Optimization Service
- D. Query Acceleration Service

**92.** You observe "Remote Disk Spilling" in Query Profile. Which TWO are reasonable immediate actions?
- A. Resize the warehouse larger
- B. Reduce auto-suspend to 60 seconds
- C. Rewrite the query to reduce data processed (better filters/pruning)
- D. Enable Snowpipe
- E. Convert the table to transient

**93.** Which session parameter disables the Result Cache?
- A. `DISABLE_CACHE = TRUE`
- B. `USE_CACHED_RESULT = FALSE`
- C. `RESULT_CACHE = OFF`
- D. `CACHE_MODE = NONE`

---

## Section G: Data Transformations (Questions 94–100)

**94.** What is the maximum size of a single VARIANT value?
- A. 1 MB
- B. 8 MB
- C. 16 MB compressed
- D. 100 MB

**95.** Which THREE are valid syntaxes to access a key `name` in a VARIANT column `payload`?
- A. `payload:name`
- B. `payload.name`
- C. `payload['name']`
- D. `payload->name`
- E. `payload#name`

**96.** What does `LATERAL FLATTEN(input => arr, OUTER => TRUE)` do?
- A. Returns only rows where the input array is non-empty
- B. Returns a row even when the input array is NULL or empty
- C. Deletes rows from the parent table
- D. Flattens the array into a single string

**97.** QUALIFY is used to:
- A. Filter rows based on aggregate functions
- B. Filter rows based on window function output
- C. Qualify a column name with a schema prefix
- D. Validate NULL constraints

**98.** Which TWO statements about UDFs vs Stored Procedures are TRUE?
- A. UDFs can execute DML
- B. Stored procedures can execute DML and DDL
- C. UDFs are callable in SELECT; sprocs are called with CALL
- D. Sprocs are always synchronous and cannot schedule themselves
- E. UDFs replace sprocs in all cases

**99.** Which TWO view types exist in Snowflake?
- A. Materialized views
- B. Partitioned views
- C. Secure views
- D. Pivot views
- E. Reference views

**100.** Which approximate aggregate function uses HyperLogLog for fast cardinality estimation?
- A. `COUNT(DISTINCT col)`
- B. `APPROX_COUNT_DISTINCT(col)`
- C. `COUNT_UNIQUE(col)`
- D. `ESTIMATE_DISTINCT(col)`

---

# Answer Key & Explanations

Scoring: give yourself 1 point per correct single-select answer. For multi-select, you earn the point only when ALL correct options are chosen AND no incorrect options are chosen. 70+ correct = passing range.

---

### Section A — Architecture

**1. C** — Hybrid architecture. See Module 1 Q2.
**2. B** — ~16 MB compressed. Module 1 Q3, Module 3 Q1.
**3. A, C, D** — Columnar, immutable, auto-encrypted/compressed. Module 1 Q3.
**4. B** — Enterprise required for multi-cluster. Module 1 Q4.
**5. B, C** — Tri-Secret Secure and PrivateLink both Business Critical+. Module 5 Q7, Q9.
**6. A, C, D** — AWS, Azure, GCP. Module 1 Q5.
**7. B** — Cloud services first 10% of daily compute is free. Module 1 Q1 Gotchas.
**8. C** — SHOW TABLES is metadata-only. Module 1 Q9.
**9. B** — `<orgname>-<account_name>` is preferred. Module 1 Q6.
**10. C** — Snowsight cannot run PUT. Module 4 Q2, Q24.
**11. A, C, E** — Shares contain granted database/schema/view/table objects. Reader accounts are a delivery mechanism, not an object. Local consumer tables aren't shared. Module 5 Q13.
**12. B, C, D** — Marketplace listings free/paid/personalized; access creates shared DB; live updates. Not physically copied; not Enterprise-gated. Module 1 Q13.
**13. B** — AWS → S3. Module 1 Q24.
**14. B — False** — One account = one cloud + one region. Module 1 Q14.
**15. A, C** — Query optimization and access control are Cloud Services responsibilities. Module 1 Q8.
**16. C** — ORGADMIN manages org-level objects. Module 5 Q1.
**17. A, C** — Metadata-only clone + copy-on-write. Module 1 Q23, Module 3 Q5.
**18. B** — Web-based UI. Module 1 Q11.
**19. B, C** — Storage billed on compressed size + TT storage counted. Module 1 Q18, Module 3 Q10.
**20. C** — `SYSTEM$GET_SNOWFLAKE_EDITION()`. Module 1 Q4 Lab.

### Section B — Virtual Warehouses

**21. B** — L = 8 credits/hr. Module 2 Q1.
**22. C** — 4XL = 128 credits/hr. Module 2 Q1.
**23. C** — 60 seconds minimum via SQL. Module 2 Q5.
**24. C** — Per-second billing, 60s minimum per resume. Module 2 Q6.
**25. C** — 10 clusters max. Module 2 Q3.
**26. A, C** — STANDARD is default/performance-biased; ECONOMY may queue longer. Module 2 Q4.
**27. C** — Multi-cluster scale-out solves concurrency. Module 2 Q10.
**28. B** — MIN=MAX>1. Module 2 Q9.
**29. B** — Default MAX_CONCURRENCY_LEVEL = 8. Module 2 Q8.
**30. B** — Running queries continue on old resources; new size used immediately for queued/new. Module 2 Q2 Gotchas, Q11.
**31. A, B, C** — QUEUED_PROVISIONING / QUEUED_OVERLOADED / QUEUED_REPAIR. Module 2 Q18.
**32. C** — Only ACCOUNTADMIN can create resource monitors. Module 2 Q13.
**33. A, B, C** — NOTIFY, SUSPEND, SUSPEND_IMMEDIATE. Module 2 Q12.
**34. B, D** — Per-warehouse (per-cluster) and lost on suspend. Module 2 Q14, Q22.
**35. B** — Started by default (unless INITIALLY_SUSPENDED=TRUE). Module 2 Q16.
**36. A, B** — More memory + ~1.5x credit rate. Minimum size Medium (not XS). Module 2 Q21.
**37. D** — 172800 seconds (48h). Module 2 Q17.
**38. A, D** — Scaling up and changing scaling policy do not disrupt running queries. SUSPEND waits for them; ABORT ALL cancels; DROP destroys. Module 2 Q11.
**39. A, C, E** — Dedicated warehouses, longer AUTO_SUSPEND, maximized clusters all improve cache hits. Module 6 Q20.
**40. B** — Query Acceleration Service. Module 2 Q24, Module 6 Q12.

### Section C — Storage & Protection

**41. A** — Standard max 1 day. Module 3 Q2.
**42. C** — Enterprise+ up to 90 days. Module 3 Q2.
**43. C** — Fixed 7 days, non-configurable. Module 3 Q3, Q8.
**44. B, C** — 7-day fixed; permanent tables only. Not user-queryable; not disable-able. Module 3 Q3.
**45. A, C** — Persist across sessions; max TT 1 day; no Fail-safe. Module 3 Q17.
**46. C** — Current session only. Module 3 Q4.
**47. A, B, C** — Databases/schemas/tables cloneable. External tables and internal stage files cannot be cloned. Module 3 Q23.
**48. B** — UNDROP within TT retention window. Module 3 Q7.
**49. B, C** — UPDATE and TRUNCATE preserve old versions in TT. Module 3 Q19.
**50. B** — Only TT disabled; Fail-safe (7d) still applies on permanent tables. Module 3 Q9.
**51. C** — Business Critical+ for failover groups. Module 3 Q12.
**52. B, D** — Users/roles and pipes/tasks/streams need failover groups. Module 3 Q13.
**53. B** — Composite Snowflake key + customer-managed key (CMK). Module 5 Q9.
**54. B** — AES-256 at rest. Module 3 Q16, Module 5 Q8.
**55. B** — MIN_DATA_RETENTION_TIME_IN_DAYS. Module 3 Q24.

### Section D — Data Movement

**56. C** — 4 types: user, table, named internal, named external. Module 4 Q1.
**57. B** — `@~`. Module 4 Q1.
**58. B** — `@%ORDERS`. Module 4 Q1.
**59. B, D** — Implicit and cannot be altered/dropped. Module 4 Q1.
**60. B** — PUT. Module 4 Q2.
**61. A, C, D** — PUT works via SnowSQL/drivers (Python/JDBC/ODBC). Not Snowsight; not pure REST /statements endpoint. Module 4 Q2, Q24.
**62. C** — 64 days. Module 4 Q23.
**63. B** — FORCE=TRUE bypasses load history. Module 4 Q3 Gotchas, Q23.
**64. C** — ABORT_STATEMENT default. Module 4 Q16.
**65. B, C** — Serverless + per-file overhead. Module 4 Q9.
**66. C** — 100–250 MB compressed. Module 4 Q4 Gotchas.
**67. A, B, D** — CSV, Parquet, JSON (plus Avro/ORC/XML). YAML and Protobuf not native. Module 4 Q5.
**68. B** — Only DML consumption advances the offset. Module 4 Q7.
**69. A, B, C** — Standard, Append-only, Insert-only. Module 4 Q17.
**70. B** — Tasks start suspended. Module 4 Q8.
**71. A, C** — Target lag SLA; chainable DAGs. Not restricted to one base table (that's MVs). Module 4 Q13.
**72. A** — Read-only metadata views over cloud files. Module 4 Q10, Q20.
**73. A, B, C** — All serverless. Warehouse resize and SWAP are not serverless features. Module 4 Q25.
**74. B** — Needs INSERT on table + USAGE on schema/DB/stage/file-format and USAGE on warehouse. Module 4 Q12.
**75. B** — Pipe runs with OWNER's privileges. Module 4 Q12 Gotchas, Q18.

### Section E — Security & Access

**76. A, C, E** — Resource monitors, account params, replication approval = ACCOUNTADMIN. Creating users is USERADMIN; granting on an owned table is doable by owner or SECURITYADMIN. Module 5 Q23.
**77. B** — USERADMIN. Module 5 Q3.
**78. A, C** — SECURITYADMIN inherits USERADMIN and has MANAGE GRANTS. Doesn't inherit SYSADMIN; doesn't create warehouses/DBs by default. Module 5 Q3.
**79. A, C** — Every user has PUBLIC; every role inherits PUBLIC. Module 5 Q17.
**80. A, B, C** — Password+MFA(Duo), key-pair JWT, SAML2 SSO. No biometric or SMS-only. Module 5 Q4.
**81. B** — Network policy. Module 5 Q6, Q18.
**82. B** — BLOCKED_IP_LIST wins over ALLOWED_IP_LIST. Module 5 Q6.
**83. B** — Enterprise+ for DDM. Module 5 Q10.
**84. A, C** — Hide definition + prevent plan-based data leakage. Secure views are available in Standard; DDM policies can still apply. Module 5 Q12, Module 6 Q19.
**85. C** — Business Critical+ for PrivateLink. Module 5 Q7.

### Section F — Performance

**86. B** — 24h base, 31d max. Module 6 Q2.
**87. A, C** — Same query text + unchanged data + role privileges to see all data + no non-deterministic funcs. Does NOT require original warehouse. Module 6 Q3.
**88. A, B** — Result + metadata caches (both Cloud Services) survive. Local SSD does not reliably persist. Module 6 Q16.
**89. B, D** — Function-on-column and leading-wildcard LIKE both defeat pruning. Module 6 Q14.
**90. B, C** — Serverless maintenance + transparent query rewrite. MVs have SQL restrictions and cost credits; require Enterprise+. Module 6 Q17, Module 7 Q25.
**91. C** — Search Optimization Service. Module 6 Q11.
**92. A, C** — Resize larger or rewrite query. Reducing auto-suspend wouldn't help the active query; Snowpipe/transient are irrelevant. Module 6 Q13.
**93. B** — `USE_CACHED_RESULT = FALSE`. Module 6 Q23.

### Section G — Data Transformations

**94. C** — 16 MB compressed. Module 7 Q1.
**95. A, B, C** — Colon, dot, bracket all valid. Arrow/hash are not. Module 7 Q1 Gotchas, Q14.
**96. B** — OUTER=TRUE preserves parent rows with NULL children (LEFT JOIN semantics). Module 7 Q24.
**97. B** — Filters window-function output. Module 7 Q11.
**98. B, C** — Sprocs can do DML/DDL; UDFs in SELECT, sprocs via CALL. Module 7 Q6.
**99. A, C** — Materialized and Secure views are explicit types in Snowflake (plus standard regular view). "Partitioned/Pivot/Reference" views are not Snowflake-defined types. Module 7 Q7.
**100. B** — `APPROX_COUNT_DISTINCT` uses HyperLogLog. Module 7 Q20.

---

## Score interpretation

- **90–100 correct:** You are exam-ready. Light review of weak domains and schedule the test.
- **75–89 correct:** Close but not safe. Target each miss, re-read the referenced Q&A module, and retake the mock in a week.
- **60–74 correct:** Core material still has gaps. Re-read all seven QA modules, drill flashcards daily for 1–2 weeks, then retake.
- **Below 60:** Treat as a study baseline. Work through modules sequentially; do hands-on labs in a trial account. Retake after 3–4 weeks.

Focus especially on:
- Any domain where you lost >25% of its questions.
- All multi-select misses (even single-wrong-option tanks the question).
- Trivia misses on numeric facts — these are the fastest to fix.
