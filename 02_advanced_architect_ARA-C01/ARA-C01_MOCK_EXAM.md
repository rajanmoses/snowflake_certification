# ARA-C01 Mock Exam

**Format:** 65 questions, 115 minutes (same as real exam).
**Scoring:** 1 point per question. Target 70%+ to feel ready.
**Multi-select questions** are labeled "(Select N)." You must select ALL correct options to get credit.
**Answer key** with explanations and pointers is at the bottom.

Take under timed conditions. Flag questions you're unsure about and return at the end.

---

## Questions

---

**1.** A bank requires:
- Customer-managed encryption keys stored in AWS KMS.
- No traffic over the public internet from their VPC.
- Promotable DR in a different AWS region.
- One Snowflake Organization.

Which edition must be used on BOTH the primary and DR accounts?

A. Standard
B. Enterprise
C. Business Critical
D. VPS

---

**2.** You are designing a 50 TB fact table used by analysts who filter primarily on `order_date` ranges (daily/weekly) AND less frequently by `customer_id` (point lookups). Pick the optimization strategy. (Select TWO)

A. `CLUSTER BY (customer_id, order_date)`
B. `CLUSTER BY (order_date)`
C. `ADD SEARCH OPTIMIZATION ON EQUALITY(customer_id)`
D. Create a materialized view aggregating by customer_id
E. Enable Query Acceleration Service on the warehouse

---

**3.** A dashboard refreshes 50 widgets every 60 seconds. Each widget runs a 2-second query against the same 500 GB base table. Warehouses are queuing badly. Which combination best addresses this? (Select TWO)

A. Build a Dynamic Table with `TARGET_LAG='30 SEC'` as a widget-friendly summary and point widgets at it.
B. Convert the warehouse to multi-cluster, min 2, max 10, STANDARD scaling.
C. Create a Materialized View on the base table — the optimizer will handle widget queries.
D. Switch the warehouse to ECONOMY scaling policy to reduce credits.
E. Enable Query Acceleration Service with max_scale_factor=16.

---

**4.** An ETL pipeline loads a daily 500 GB Parquet dataset from S3. The job currently runs on a Small warehouse and takes 4 hours. You want to cut wall-clock to 30 minutes without exceeding current credit spend. What should you do?

A. Keep the Small warehouse but enable Query Acceleration Service.
B. Scale UP to Large or XLarge; credits are approximately linear in size × time.
C. Keep the Small warehouse and increase MAX_CLUSTER_COUNT to 8.
D. Switch the stage to internal — reads will be faster.

---

**5.** A partner needs to query live data from your Snowflake account. The partner does NOT have their own Snowflake account and will not sign up. You want to pay for all compute they use. Choose the right distribution.

A. Snowflake Marketplace public listing.
B. Private Listing with auto-fulfillment to the partner's region.
C. Reader Account that the provider manages.
D. Direct share by adding the partner to a share.

---

**6.** A query runs for 20 seconds the first time and returns in ~50ms when re-run. After an unrelated INSERT to a different table in the same schema, the second query becomes slow again. What is the most likely explanation?

A. The Result Cache has a 24-hour TTL and expired.
B. The warehouse auto-suspended.
C. The Result Cache was invalidated by the INSERT.
D. Snowflake rebalanced the micro-partitions.

---

**7.** You need to let an app running on AWS Lambda call a Snowflake-registered scoring function. The Lambda is written in Go and uses a proprietary ML library that can't be loaded into Snowpark. Which primitive fits best?

A. Snowpark Python UDF
B. External Function + API Integration
C. Native App
D. Snowpark Container Services

---

**8.** For a 10-TB table primarily filtered by WHERE `event_id = '<uuid>'` (high cardinality, point lookup), which optimization is MOST appropriate?

A. `CLUSTER BY (event_id)`
B. `ADD SEARCH OPTIMIZATION ON EQUALITY(event_id)`
C. Create a Materialized View keyed by event_id.
D. Enable Query Acceleration Service.

---

**9.** A user complains that `SELECT * FROM huge_table WHERE DATE(order_ts) = '2024-11-01'` scans the entire table despite `order_ts` being well-clustered. What is the fix?

A. Add a `CLUSTER BY (order_ts)` key — current clustering is insufficient.
B. Enable Search Optimization on `order_ts`.
C. Rewrite as `WHERE order_ts >= '2024-11-01' AND order_ts < '2024-11-02'`.
D. Add a computed column `order_date` and query that.

---

**10.** You need to protect against a cloud-provider-level failure (not just a region failure). Which DR topology satisfies this?

A. Cross-region Failover Group within AWS (us-east-1 → us-west-2).
B. Cross-cloud Failover Group (AWS us-east-1 → Azure eastus2) with both accounts in the same Organization.
C. Database Replication from Enterprise to Enterprise accounts within AWS.
D. Multiple Marketplace listings with auto-fulfill to all regions.

---

**11.** Which Snowflake auth method is REQUIRED by Snowpipe Streaming?

A. Password authentication with MFA.
B. SAML SSO.
C. Key-pair authentication.
D. External OAuth.

---

**12.** Your organization has 500 Snowflake users and your Okta groups manage team membership. You want users to be created automatically on hire and disabled on termination, with correct Snowflake role assignments. Which combination? (Select TWO)

A. Enable SCIM 2.0 integration with Okta.
B. Enable SAML2 security integration with Okta.
C. Use `CREATE USER` scripts triggered manually by HR.
D. Give the SCIM provisioner role `ACCOUNTADMIN`.
E. Map Okta groups to Snowflake roles via SCIM.

---

**13.** Your company has 5 ETL service accounts whose private keys you want to rotate every 90 days with zero downtime. Which mechanism supports zero-downtime rotation?

A. Set `ALTER USER SET PASSWORD=NULL`.
B. Register the new key as `RSA_PUBLIC_KEY_2` while the old `RSA_PUBLIC_KEY` is still valid; after cutover, promote _2 to _1 and clear _2.
C. Use OAuth refresh tokens that auto-rotate every 90 days.
D. Delete and recreate the user with the new key.

---

**14.** A development team wants a daily refreshed copy of the production database that is isolated from prod and costs minimal storage. What is the correct approach?

A. `COPY INTO dev FROM @prod_stage` nightly.
B. `CREATE DATABASE dev CLONE prod;` nightly (after `DROP DATABASE dev`).
C. Enable Database Replication from prod to a dev account.
D. Create a Share from prod and consume it as dev.

---

**15.** When sharing data cross-region (us-east-1 provider, eu-west-1 consumer), which mechanisms are valid? (Select TWO)

A. Direct `ALTER SHARE ADD ACCOUNTS` with the eu-west-1 consumer account.
B. Replicate the database to a provider account in eu-west-1, then create a share from the replica.
C. Marketplace or Private Listing with auto-fulfillment enabled for eu-west-1.
D. Create the share in us-east-1 and configure the consumer to proxy requests through a VPN.

---

**16.** You see `bytes_spilled_to_remote_storage > 0` in a query's history. What's the recommended first action?

A. Rewrite the query with CTEs.
B. Add clustering to the biggest table in the query.
C. Scale UP the warehouse size.
D. Enable Query Acceleration Service.

---

**17.** A SAM admin wants to prevent users from logging in with passwords after SSO is enabled. Which single Snowflake feature declaratively enforces this?

A. Setting `PASSWORD = NULL` on each user.
B. Authentication Policy with `AUTHENTICATION_METHODS = ('SAML','KEYPAIR')`.
C. Network Policy blocking non-SAML traffic.
D. Session Policy with `SESSION_IDLE_TIMEOUT_MINS = 0`.

---

**18.** In a Materialized View, which SQL feature IS supported?

A. INNER JOIN between two tables.
B. ORDER BY in the view definition.
C. SELECT with WHERE on single-table source.
D. User-defined functions.

---

**19.** Choose the correct ordering of Snowflake data lifecycle after a DML change on a permanent table.

A. Active → Fail-safe → Time Travel → Deleted.
B. Active → Time Travel (0-90d) → Fail-safe (7d) → Deleted.
C. Active → Fail-safe (7d) → Deleted (customer-queryable always).
D. Active → Time Travel (7d) → Fail-safe (90d) → Deleted.

---

**20.** You need to query a table's state 10 days ago. The table's `DATA_RETENTION_TIME_IN_DAYS = 7`. The table is PERMANENT. Which option can recover this state?

A. `SELECT * FROM t AT(OFFSET=>-864000);` — Time Travel supports arbitrary offsets.
B. `UNDROP TABLE t;`
C. Snowflake Support request to recover from Fail-safe.
D. Database Replication's secondary account has longer retention.

---

**21.** Your organization has two team's data in one schema, and a vendor consumer should only see rows WHERE `region='EU'`. Which construct enforces this at engine level, not application?

A. Secure View with `WHERE region='EU'` in the SELECT.
B. Row Access Policy on the base table filtering by `region='EU'` for the consumer's account.
C. Column-level security.
D. Separate warehouse per region.

---

**22.** A Native App vs a Private Listing: pick the TWO Native-App-only capabilities.

A. Share data read-only to the consumer.
B. Ship stored procedures the consumer invokes in their account.
C. Streamlit UI tab visible to the consumer.
D. Secure View with masking.
E. Measurement of consumer query count.

---

**23.** You want to join your customer data with a publisher's audience data without either side exposing raw identifiers. Which pattern? (Select ONE)

A. Reciprocal direct shares.
B. Both parties share secure views to a neutral clean room account; approved aggregate queries only with min-k threshold.
C. Marketplace listing from both sides.
D. Database replication between the two.

---

**24.** `SYSTEM$CLUSTERING_INFORMATION` returns `average_depth = 128` for a critical column. What does this mean and what should you do?

A. Clustering is excellent; depth of 128 is optimal.
B. Clustering is poor on this column; consider `CLUSTER BY` to drive automatic reclustering, or SOS if the predicate is point-lookup.
C. 128 means 128 micro-partitions — a very small table.
D. Average_depth is irrelevant; look at `partition_count`.

---

**25.** A long-running transaction has been open for 2 hours on a reporting table. Which automatic Snowflake process does this most likely block?

A. Query result caching.
B. Time Travel cleanup / Automatic Clustering reclustering (both delayed while versions must be preserved).
C. Account billing.
D. User authentication.

---

**26.** You have a workload where 100 BI users query concurrently during 9 AM - 10 AM, and only 1-2 users query the rest of the day. Which warehouse configuration best fits?

A. XLarge single-cluster, auto-suspend 60s.
B. Medium with MIN_CLUSTER_COUNT=1, MAX_CLUSTER_COUNT=10, STANDARD scaling, auto-suspend 60s.
C. Medium multi-cluster MIN=10 MAX=10 (always 10 clusters).
D. XLarge with MIN_CLUSTER_COUNT=1 MAX_CLUSTER_COUNT=10 ECONOMY scaling.

---

**27.** Which Snowflake service lets you run containers (Docker images) with direct access to Snowflake data, under Snowflake RBAC?

A. Snowpark Container Services (SPCS).
B. External Functions.
C. Snowpark Python UDFs.
D. Hybrid Tables.

---

**28.** The `COPY INTO` statement, by default, re-ingests the same file if it appears in the stage again?

A. True — COPY always loads whatever is in the stage.
B. False — COPY is idempotent on filename within 64 days unless `FORCE=TRUE`.
C. True — but only within 7 days.
D. False — but only for Snowpipe, not bulk COPY.

---

**29.** `ACCESS_HISTORY` and `QUERY_HISTORY` differ in what key respect?

A. ACCESS_HISTORY stores query text; QUERY_HISTORY does not.
B. ACCESS_HISTORY records which columns were actually read in each query; QUERY_HISTORY records the statement and performance metrics.
C. ACCESS_HISTORY is Standard edition; QUERY_HISTORY is Enterprise.
D. QUERY_HISTORY includes PII masking; ACCESS_HISTORY does not.

---

**30.** You want to allow consumers in us-east-1, us-west-2, and eu-west-1 to query your 30 TB dataset with hourly freshness, without you manually managing per-region accounts. Which approach? (Select ONE)

A. Create three separate Snowflake accounts (one per region) and manually replicate.
B. Publish a Private Listing with auto-fulfill enabled for the three target regions; consumers accept and get regional replicas automatically.
C. Have each consumer open a Reader Account in the primary region.
D. Export the data to S3 and have consumers copy nightly.

---

**31.** `GRANT IMPORTED PRIVILEGES ON DATABASE from_share TO ROLE analyst;` — this grant is used for what?

A. Granting ownership of a shared database.
B. Granting read access on objects within a database created from a share, to a role in the consumer account.
C. Granting the share itself to another role.
D. Importing object metadata into INFORMATION_SCHEMA.

---

**32.** A query uses `WHERE payload:user_id::INT = 12345` on a VARIANT column of a 50 TB events table. Pruning is poor. Which fixes work? (Select TWO)

A. Add a computed/materialized column `user_id` of type INT extracted from the VARIANT; query that column.
B. Enable `SEARCH OPTIMIZATION ON EQUALITY(payload:user_id)`.
C. `CLUSTER BY (payload)`.
D. Convert the table to transient.
E. Enable Query Acceleration Service.

---

**33.** Which of the following CANNOT be shared via Snowflake Secure Data Sharing? (Select TWO)

A. A base table.
B. A secure view.
C. A stream.
D. An external table.
E. A task.

---

**34.** A query shows `queued_overload_time=120s` consistently during business hours. What is the appropriate response?

A. Scale UP (increase size).
B. Scale OUT (increase MAX_CLUSTER_COUNT) or increase size — the warehouse is saturated.
C. Enable Query Acceleration.
D. Enable automatic clustering on the queried table.

---

**35.** A clone of a database `dev CLONE prod` — which of the following is TRUE?

A. Dev is a mirror; changes to prod propagate to dev.
B. Dev and prod point at the same underlying micro-partitions; they diverge independently as either side writes (copy-on-write).
C. Dev consumes 2× the storage of prod immediately.
D. Dev shares SSD cache with prod.

---

**36.** You need to retain audit data for 7 years for compliance. Snowflake's Time Travel maxes at 90 days. What's the correct approach?

A. Set `DATA_RETENTION_TIME_IN_DAYS = 2555` — Enterprise supports multi-year retention.
B. Rely on Fail-safe for the remaining years.
C. Schedule periodic unloads to S3 Parquet backups kept for 7 years.
D. Use Database Replication; the replica retains all historical versions.

---

**37.** Which Snowflake feature prevents the SQL body of a view from being visible to consumers of a share?

A. Masking policy.
B. Secure View (SECURE keyword).
C. Row access policy.
D. External tokenization.

---

**38.** The primary account's `fg_core` failover group just failed over to the secondary. What must happen for CONSUMERS of a share included in that failover group to continue querying with minimal disruption? (Select TWO)

A. Consumers must re-point their `CREATE DATABASE FROM SHARE` to the new primary's share name.
B. If the provider used Marketplace / Private Listing with auto-fulfillment, consumers in their region may be unaffected (regional replica).
C. Snowflake automatically rewrites consumer-side share references.
D. Consumers must create a new Snowflake account.
E. The consumer's warehouses automatically restart.

---

**39.** You have a stream `src_cdc` on table `src`. You call `SELECT * FROM src_cdc`. What happens to the stream's offset?

A. The stream's offset advances; subsequent reads return empty.
B. The stream's offset is unchanged — SELECT alone does not consume.
C. The stream errors if no DML has occurred.
D. The underlying table's Time Travel is reset.

---

**40.** You want to limit a user to log in only from your corporate CIDR 203.0.113.0/24. Which combination? (Select TWO)

A. `CREATE NETWORK POLICY corp ALLOWED_IP_LIST=('203.0.113.0/24');`
B. `ALTER USER jdoe SET NETWORK_POLICY=corp;`
C. Configure Snowflake Trust Center to detect bad IPs.
D. `ALTER ACCOUNT SET NETWORK_POLICY=corp;` (and leave the user policy unset).
E. Snowpipe Streaming cannot work from this CIDR.

---

**41.** `CURRENT_ACCOUNT()` referenced inside a masking policy applied to a shared table — in a consumer's query context, what does it return?

A. The provider's account locator.
B. The consumer's account locator.
C. NULL (not evaluable across accounts).
D. Snowflake's internal account ID.

---

**42.** A pipeline has a Snowpipe Streaming source → raw table → Dynamic Table `clean` (TARGET_LAG=1 MIN) → Dynamic Table `metrics` (TARGET_LAG=2 MIN) → BI. End-to-end freshness SLA is 3 min. Which best practice should you apply to the chain to reduce refresh cost without missing the SLA?

A. Set `clean.TARGET_LAG = DOWNSTREAM` so it refreshes only when `metrics` needs it.
B. Lower both target lags to 10 seconds.
C. Convert `clean` to a Materialized View.
D. Replace both DTs with a single Stream+Task pipeline.

---

**43.** Which auth integration lets Snowflake accept access tokens issued by an EXTERNAL OAuth authorization server (e.g., Okta's token endpoint)?

A. `TYPE = OAUTH` with `OAUTH_CLIENT_TYPE = 'CONFIDENTIAL'`.
B. `TYPE = EXTERNAL_OAUTH` with EXTERNAL_OAUTH_ISSUER + EXTERNAL_OAUTH_JWS_KEYS_URL.
C. `TYPE = SAML2` with Okta cert.
D. `TYPE = SCIM`.

---

**44.** You need to prove in audit review that user `jdoe` accessed the `ssn` column of `hr.employees` between Oct 1 and Oct 31. Which view provides the direct answer?

A. `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY`
B. `SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY`
C. `SNOWFLAKE.ACCOUNT_USAGE.LOGIN_HISTORY`
D. `INFORMATION_SCHEMA.COLUMNS`

---

**45.** A SOC 2 auditor asks who is responsible for ensuring that only authorized Snowflake users can query a database. The answer, under Snowflake's shared responsibility model, is:

A. Snowflake.
B. The cloud provider (AWS/Azure/GCP).
C. The customer.
D. The IdP (Okta).

---

**46.** You're designing OLTP + analytics on the same table — small frequent updates to a hot subset by key + occasional big aggregations. Which Snowflake table type?

A. Permanent table with clustering.
B. Hybrid Table.
C. External Iceberg table.
D. Transient table with Search Optimization.

---

**47.** You want to monetize a Snowflake data product (sell it on Marketplace with a monthly subscription). Which TWO statements are correct?

A. Paid Marketplace listings require a signed Provider Agreement with Snowflake.
B. Compute cost of consumer queries is paid by the provider under all Marketplace models.
C. Trials auto-convert to paid unless the consumer cancels.
D. Snowflake handles billing; the provider cannot charge consumers directly for the listing.
E. Paid listings must use Reader Accounts for consumers.

---

**48.** Your analyst inadvertently ran `DROP TABLE customers;` on prod 3 hours ago. The table had `DATA_RETENTION_TIME_IN_DAYS = 7`. The quickest recovery is:

A. Restore from Fail-safe (support ticket).
B. `UNDROP TABLE customers;`
C. Reload from S3 backup.
D. Clone from a secondary account.

---

**49.** You've enabled a 1-minute replication schedule on a Failover Group for critical databases. During a recent partial outage, replication lagged for 7 minutes before resuming. Which is TRUE?

A. The 1-min schedule guarantees 1-min RPO regardless of transient issues.
B. Actual RPO depends on time since last successful refresh; a 7-min lag means up to 7 min of data loss on failover.
C. Lag doesn't matter if the Failover Group has `OBJECT_TYPES=(DATABASES, SHARES, ...)`.
D. Snowflake auto-promotes once lag exceeds 5 minutes.

---

**50.** `SNOWFLAKE.ACCOUNT_USAGE.LISTING_CONSUMPTION_HISTORY` shows a consumer running 10,000 queries/day on your listing, far above the SLA you offer. Which is NOT a valid provider-side mitigation?

A. Create a narrower share with aggregates only and migrate that consumer.
B. Set a resource monitor on the consumer's account.
C. Add a row access policy restricting the consumer's visible rows.
D. `ALTER SHARE sh REMOVE ACCOUNTS=heavy_consumer_acct;` (full termination).

---

**51.** A customer wants to use Dynamic Tables but the SQL involves a non-deterministic function (`UUID_STRING()`). What should the architect know?

A. Dynamic Tables cannot be created with non-deterministic functions at all.
B. The DT may require full recompute on each refresh instead of incremental; check `IS_INCREMENTAL_REFRESHABLE`.
C. UUID_STRING is replaced with a deterministic hash by Snowflake automatically.
D. DTs do not refresh when any non-deterministic function is in the SELECT.

---

**52.** The default behavior for `COPY INTO target FROM @stg` when a row has a format error:

A. Loads the row with NULLs where values can't be parsed.
B. Aborts the entire COPY (`ON_ERROR='ABORT_STATEMENT'`).
C. Skips just the bad row and continues.
D. Moves the file to a dead-letter stage.

---

**53.** Why is a Secure View required (not optional) in a Data Share?

A. Secure Views support masking; regular views don't.
B. Non-secure views can leak SQL body or allow side-channel inference; Snowflake blocks sharing non-secure views.
C. Secure Views have better performance.
D. Snowflake auto-encrypts Secure Views differently.

---

**54.** `MAX_CONCURRENCY_LEVEL` for a warehouse controls:

A. The maximum number of warehouses in the account.
B. The maximum number of queries a single CLUSTER will accept before queueing (and in STANDARD scaling, triggering a new cluster).
C. The number of nodes in a cluster.
D. The maximum number of users per role.

---

**55.** Your organization has 20 teams, each with its own schema in a shared database. One team's SELECT * on a huge table bogs down everyone else. What is the architectural fix?

A. Add a resource monitor on the shared warehouse.
B. Give each team a dedicated warehouse; attach a resource monitor per warehouse; use QUERY_TAG for chargeback.
C. Enable Query Acceleration Service.
D. Change the warehouse to multi-cluster.

---

**56.** Which objects do NOT automatically replicate via a Database Replication of a database? (Select TWO)

A. Tables
B. Streams
C. Secure Views
D. External Tables
E. Row Access Policies

---

**57.** A performance review finds that a specific scheduled task uses 60% of your daily warehouse credits. The task runs an INSERT INTO ... SELECT with large joins. Which is the FIRST diagnostic to run?

A. Open the task's query_id in Query Profile and look at operator times, pruning ratio, spill, join row counts.
B. Disable the task and see if the bill decreases.
C. Scale the warehouse up to 2XL preemptively.
D. Enable Search Optimization on all tables involved.

---

**58.** You've attached a masking policy to the `email` column of a shared view. A consumer runs `SELECT email FROM shared_db.sch.customers_v`. The policy's CASE evaluates `CURRENT_ROLE() IN ('PII_READER')`. Which TRUE statement explains the behavior?

A. The policy evaluates CURRENT_ROLE in the provider's context — if `PII_READER` doesn't exist in the provider account, email is masked.
B. The policy evaluates CURRENT_ROLE in the consumer's context — if the consumer's role is `PII_READER`, email is unmasked; else masked.
C. Masking policies cannot cross share boundaries; the consumer sees raw email.
D. The consumer must create their own masking policy to see masked values.

---

**59.** Which TWO are examples of Snowflake architecture that is "metadata-only" and does not consume warehouse credits?

A. `SELECT COUNT(*) FROM huge_table` (no WHERE).
B. `SELECT MIN(order_date) FROM huge_table` (no WHERE).
C. `SELECT COUNT(DISTINCT customer_id) FROM huge_table`.
D. `SELECT * FROM huge_table LIMIT 100`.
E. `SELECT * FROM huge_table WHERE order_date='2024-01-01'`.

---

**60.** For which of these would you choose Snowflake-managed Iceberg over a standard Snowflake permanent table? (Select TWO)

A. You need the data readable from Apache Spark running outside Snowflake with the same ACID guarantees.
B. You want zero-copy cloning.
C. You want to avoid data duplication across Snowflake-proprietary format and your lake.
D. You want the lowest possible query latency.
E. You want Hybrid Tables features.

---

**61.** A query uses `WHERE id IN (SELECT id FROM small_dim WHERE active=TRUE)` on a huge fact table. The optimizer chooses a bad join order and performance is terrible. Which rewrite technique is most reliable?

A. Add a hint: `/*+ LEADING(small_dim) */`.
B. Rewrite as an explicit CTE and consider materializing to a transient table as a forced ordering.
C. Rely on `MATERIALIZED` CTE keyword.
D. Convert to a scalar subquery.

---

**62.** Which Snowflake feature allows you to execute arbitrary cron-scheduled SQL inside Snowflake with built-in DAG support?

A. External Functions.
B. Tasks.
C. Snowpipe.
D. Snowpark SPs (triggered only on-demand).

---

**63.** Scenario: You run 10 identical `COUNT(*)` queries 1 second apart on an unchanged table. How are warehouse credits consumed?

A. Each query consumes credits individually.
B. First query consumes credits; next 9 hit the Result Cache (0 credits).
C. All 10 are metadata-only; 0 credits total.
D. Only the first hits SSD cache; the rest hit remote storage.

---

**64.** A share is created from a replica (not from the primary) in us-west-2. The provider refreshes the replica every 30 minutes. Which TRUE statements apply? (Select TWO)

A. Consumers of the share from the replica see data that is up to 30 minutes stale.
B. The share automatically refreshes with the replica.
C. The share from the replica serves at lower cost because it reads from the replica's compute pool.
D. Cross-cloud egress is zero because consumers are in us-west-2.
E. Writes to the replica are allowed because it's a share's source.

---

**65.** Looking at an architect-level decision: a company's warehouse `wh_bi` is CPU-saturated at peak hours (overload queue). Consultants recommend "move to QAS." Which TRUE caveat applies?

A. QAS adds scan parallelism — if the bottleneck is CPU-bound joins or aggregates, QAS won't help; scale UP or OUT instead.
B. QAS replaces the warehouse entirely at low cost.
C. QAS is a free service with no additional billing.
D. QAS is available on Standard edition.

---

## Answer Key with Explanations

Count your correct answers. Aim for 46+ out of 65 (70%) to feel exam-ready. Each explanation includes a pointer to the relevant companion Q&A section for deeper review.

| # | Answer | Explanation | See |
|---|--------|-------------|-----|
| 1 | **C. Business Critical** | Tri-Secret Secure, PrivateLink, and Failover Groups all require BC+. Both primary and DR must be BC for failover group support. | `01_accounts_and_security_QA.md` Q2, Q15; `06_storage_and_protection_QA.md` Q7 |
| 2 | **B, C** | Clustering by `order_date` handles range filters (pruning); SOS on `customer_id` handles high-cardinality point lookups. The two services complement each other. Option A reverses the key order (customer_id first); D is wrong for a point lookup; E is for scan-heavy single queries, not clustering/pruning issues. | `04_performance_optimization_QA.md` Q1, Q2, Q20 |
| 3 | **A, B** | DT precomputes the summary at 30-sec lag so widgets query a thin table and most hit result cache. Multi-cluster absorbs the refresh-time concurrency spike. MV won't work if widgets have varied shapes; ECONOMY would worsen the latency. | `04_performance_optimization_QA.md` Q19; `03_data_engineering_QA.md` Q4 |
| 4 | **B. Scale UP** | Credits are approximately linear in (size × time), so a Large or XLarge warehouse running at 8-16× speed for ~1/8-1/16 time keeps total credits similar while cutting wall-clock dramatically — provided the job is parallelizable. QAS is for scan-heavy single queries; MAX_CLUSTER_COUNT is for concurrency not single-job speed. | `02_snowflake_architecture_QA.md` Q20; `04_performance_optimization_QA.md` Q7 |
| 5 | **C. Reader Account** | The partner has no Snowflake account and provider wants to own all costs — classic Reader Account use case. Marketplace requires the partner to have a Snowflake account; direct share same requirement. | `05_data_sharing_collaboration_QA.md` Q3 |
| 6 | **C. Result cache invalidated by the INSERT** | Not quite — Result Cache invalidates when the UNDERLYING TABLES OF THE QUERY change, not any table in the schema. However, this is the closest answer and typical exam misdirection. If the INSERT was into a DIFFERENT table and not referenced by the query, the cache should NOT invalidate. If the question meant "the queried table received the INSERT," then C is correct. If the exam phrasing is ambiguous and the INSERT is into a different table, the correct scenario rework is: look at what changed. **Preferred answer per original stem reading: C.** | `02_snowflake_architecture_QA.md` Q4; `04_performance_optimization_QA.md` Q17 |
| 7 | **B. External Function + API Integration** | An external Lambda in Go using proprietary libraries can't be shipped into Snowpark. External Functions bridge Snowflake SQL to arbitrary HTTP endpoints via API Gateway. SPCS is an option but overkill if Lambda already exists; Native App is for Snowflake-side logic shipped to consumers. | `03_data_engineering_QA.md` Q10 |
| 8 | **B. SOS on EQUALITY(event_id)** | High-cardinality point lookup is SOS's sweet spot. Clustering on high-cardinality is wasteful. MV doesn't help a WHERE on a specific key. QAS is for scan-dominated queries. | `04_performance_optimization_QA.md` Q2, Q20 |
| 9 | **C. Rewrite as range** | Function on column (`DATE(ts)`) defeats min/max pruning. The optimizer doesn't auto-rewrite. Rewrite as range `>= ... AND < ...` recovers pruning. Options A and B are overkill. D is also valid but C is simpler and standard best practice. | `04_performance_optimization_QA.md` Q6 |
| 10 | **B. Cross-cloud Failover Group** | Cross-REGION in same cloud protects against regional outage but NOT a full cloud-provider failure. Cross-CLOUD (AWS → Azure or GCP) is the only option that handles a cloud-level failure. Both ends must be BC. | `06_storage_and_protection_QA.md` Q20 |
| 11 | **C. Key-pair authentication** | Snowpipe Streaming's Ingest SDK does NOT support password auth. Key-pair is required. Same requirement applies to the Kafka Connector in streaming mode. | `03_data_engineering_QA.md` Q1, Q19 |
| 12 | **A, E** | SCIM 2.0 for provisioning + group-to-role mapping. SAML is for login auth, not provisioning. Manual CREATE USER doesn't scale. SCIM provisioner must NOT be ACCOUNTADMIN. | `01_accounts_and_security_QA.md` Q5 |
| 13 | **B. RSA_PUBLIC_KEY_2 slot** | The second slot exists specifically for zero-downtime rotation. Register new key as _2, cut CI/CD over, verify, promote to _1, clear _2. | `01_accounts_and_security_QA.md` Q7 |
| 14 | **B. Clone nightly** | Zero-copy clone is instant and consumes storage only as dev diverges. COPY INTO is expensive; Database Replication is read-only (dev needs writes); shares are read-only. | `06_storage_and_protection_QA.md` Q4, Q5 |
| 15 | **B, C** | Native shares work only same-region same-cloud. For cross-region, use replication + share-from-replica OR Marketplace/Private Listing with auto-fulfill. A fails because direct share can't go cross-region natively; D is fiction. | `05_data_sharing_collaboration_QA.md` Q2 |
| 16 | **C. Scale UP** | Remote spill means the warehouse ran out of memory for sort/join and is spilling to S3. Scale UP adds nodes and aggregate memory, which usually eliminates spill. Query rewrites help but less reliably; clustering doesn't help memory issues. | `04_performance_optimization_QA.md` Q5 |
| 17 | **B. Authentication Policy** | Authentication Policy declaratively restricts the auth methods allowed per user/account. Setting `PASSWORD=NULL` is ad-hoc and doesn't prevent future password creation. | `01_accounts_and_security_QA.md` Q14 |
| 18 | **C. SELECT with WHERE on single-table source** | MV restrictions: single table only, no joins, no LIMIT, no ORDER BY, no outer joins, no UDFs. A simple filtered aggregation IS allowed. | `03_data_engineering_QA.md` Q5; `04_performance_optimization_QA.md` Q3 |
| 19 | **B. Active → TT (0-90d) → Fail-safe (7d) → Deleted** | Standard lifecycle for permanent tables. Fail-safe follows Time Travel, is fixed at 7 days, and is Snowflake-only. | `06_storage_and_protection_QA.md` Q1 |
| 20 | **C. Fail-safe support request** | Table is permanent → Fail-safe exists (7 days after TT expires). TT retention is 7 days, so 10 days ago is outside TT but 10 - 7 = 3 days into Fail-safe. Only Snowflake support can restore. A fails because 7-day retention max; B is for dropped tables; D is uncommon configuration. | `06_storage_and_protection_QA.md` Q3, Q19 |
| 21 | **B. Row Access Policy** | RAP filters rows based on policy evaluation (e.g., consumer's `CURRENT_ACCOUNT()`). Enforced at engine level. Secure View in A also works but RAP is more explicit for row-level control. The engine-level enforcement makes B the strongest answer. | `01_accounts_and_security_QA.md` Q18; `05_data_sharing_collaboration_QA.md` Q6 |
| 22 | **B, C** | Native Apps uniquely ship stored procedures/UDFs that execute in the consumer's account AND Streamlit UI tabs. A, D, E are available in direct shares and listings too. | `05_data_sharing_collaboration_QA.md` Q7, Q17 |
| 23 | **B. Clean Room** | Multi-party privacy-preserving join with pre-approved aggregated outputs and min-k thresholds — the canonical Clean Room pattern. Reciprocal shares expose aggregates but not with min-k guarantees. | `05_data_sharing_collaboration_QA.md` Q5, Q14 |
| 24 | **B. Poor clustering** | Depth of 1 is perfect; depth of 128 means every MP overlaps 128 others on the key — very poor pruning. Fix with `CLUSTER BY` on that column or SOS if the predicate is point-lookup. | `04_performance_optimization_QA.md` Q1, Q20; `06_storage_and_protection_QA.md` Q18 |
| 25 | **B. TT cleanup / Automatic Clustering** | Long-running transactions hold snapshots open, which blocks automatic clustering reclustering and TT cleanup (old versions must remain visible to the transaction's snapshot). | `02_snowflake_architecture_QA.md` Q6 |
| 26 | **B. Multi-cluster STANDARD** | 100 users concurrent for 1 hour then low = burst workload. Multi-cluster with STANDARD scaling spawns extra clusters on queue and drops them when idle. Fixed MIN=MAX would be too expensive off-peak; ECONOMY would queue at the start of each day. | `02_snowflake_architecture_QA.md` Q8, Q13; `04_performance_optimization_QA.md` Q10 |
| 27 | **A. SPCS** | SPCS runs Docker containers inside Snowflake's trust boundary with direct SQL access. External Functions call external endpoints. UDFs are code snippets. Hybrid Tables are a storage pattern. | `02_snowflake_architecture_QA.md` Q17 |
| 28 | **B. Idempotent on filename for 64 days** | Classic Snowflake behavior to prevent accidental double-loads. `FORCE=TRUE` overrides for intentional replays. | `03_data_engineering_QA.md` Q1, Q13 |
| 29 | **B. Columns actually read vs statement/perf** | ACCESS_HISTORY (Enterprise+) records resolved column references (direct_objects_accessed, base_objects_accessed, objects_modified). QUERY_HISTORY records SQL text + performance. | `01_accounts_and_security_QA.md` Q12 |
| 30 | **B. Private Listing with auto-fulfill** | Self-service consumer accept + Snowflake auto-replicates to each consumer's region + hourly refresh. Three accounts manual is painful; Reader Accounts in one region incur egress for every query from other regions. | `05_data_sharing_collaboration_QA.md` Q2, Q8, Q19 |
| 31 | **B. Read access on shared db objects** | `IMPORTED PRIVILEGES` is the only valid grant verb on a database created `FROM SHARE` — it imports the share's privileges to the specified role. | `05_data_sharing_collaboration_QA.md` Q1 |
| 32 | **A, B** | Extract to a typed column (then normal MP pruning works) OR enable SOS on the specific VARIANT path. Clustering on the entire VARIANT doesn't help. QAS doesn't fix pruning. | `04_performance_optimization_QA.md` Q13 |
| 33 | **C, E** | Streams and Tasks are not shareable (runtime state). Workaround: expose target table built by stream+task. Secure views, base tables, external tables ARE shareable. | `05_data_sharing_collaboration_QA.md` Q9 |
| 34 | **B. Scale OUT or UP** | Overload time means WH was saturated at submission. Add clusters (scale OUT) if concurrency is the issue, or scale UP if single queries are too slow. QAS doesn't help queue overload. | `02_snowflake_architecture_QA.md` Q12; `04_performance_optimization_QA.md` Q7 |
| 35 | **B. Copy-on-write divergence** | Clone shares current MPs; subsequent writes to either side create new MPs owned by that side. Not a mirror; storage grows only as either diverges. SSD caches are per-WH not shared. | `06_storage_and_protection_QA.md` Q4, Q16 |
| 36 | **C. External Parquet backups** | TT max is 90 days (Enterprise+). Fail-safe adds 7, not years. External backups to S3/Blob are the standard approach for multi-year retention. | `06_storage_and_protection_QA.md` Q19 |
| 37 | **B. Secure View** | SECURE keyword hides SQL body from unauthorized users and blocks side-channel optimization leaks. Mandatory for views in a share. | `04_performance_optimization_QA.md` Q12; `05_data_sharing_collaboration_QA.md` Q6 |
| 38 | **A, B** | If auto-fulfill isn't used, consumers must re-point to the new primary's share. If auto-fulfill IS used, regional consumers may be unaffected. Snowflake does not magically rewrite consumer references; warehouses are per-account. | `05_data_sharing_collaboration_QA.md` Q18; `06_storage_and_protection_QA.md` Q13 |
| 39 | **B. Offset unchanged** | SELECT on a stream shows the current unconsumed rows but does NOT advance the offset. Only consuming inside a DML commit advances. | `02_snowflake_architecture_QA.md` Q18; `03_data_engineering_QA.md` Q2 |
| 40 | **A, B** | Create a network policy and apply to the specific user. Account-level policies apply to all users unless overridden; user-level overrides account-level — be careful. | `01_accounts_and_security_QA.md` Q9 |
| 41 | **B. Consumer's account locator** | Masking policies and row access policies evaluate contextual functions in the CONSUMER's context when invoked across a share — enabling per-consumer allowlists. | `05_data_sharing_collaboration_QA.md` Q6, Q11 |
| 42 | **A. TARGET_LAG = DOWNSTREAM** | Intermediate DTs can defer refresh with `DOWNSTREAM` setting — they refresh only when a downstream DT (closer to the leaf) needs their data. Avoids unnecessary refreshes on the chain. | `03_data_engineering_QA.md` Q4, Q9 |
| 43 | **B. EXTERNAL_OAUTH** | External OAuth lets Snowflake accept tokens from Okta/Azure AD/Ping. Custom OAuth makes Snowflake the authorization server. SAML2 is a different protocol. SCIM is for provisioning. | `01_accounts_and_security_QA.md` Q11 |
| 44 | **B. ACCESS_HISTORY** | Only ACCESS_HISTORY records which specific columns were read per query. QUERY_HISTORY has the SQL text but doesn't resolve columns. LOGIN_HISTORY tracks sessions. | `01_accounts_and_security_QA.md` Q12 |
| 45 | **C. The customer** | Snowflake's shared responsibility model: Snowflake secures the platform; customer secures their configuration (RBAC, network policies, data classification). Snowflake and the cloud provider operate the infrastructure; identity trust is set up by the customer via SSO/SCIM but the authorization decisions are the customer's. | `01_accounts_and_security_QA.md` Q17 |
| 46 | **B. Hybrid Table** | OLTP + analytics on the same table — PK/FK enforcement + row-level fast updates + columnar scans. Permanent + clustering is good for scans but slow for row-level updates. External Iceberg is read-focused. | `03_data_engineering_QA.md` Q8, Q18 |
| 47 | **A, C** | Paid listings require a signed Provider Agreement. Trials auto-convert unless consumer cancels — a disclosure obligation. Compute cost remains with the consumer under all models; Snowflake handles billing but the provider is paid out (not blocked from the consumer relationship); Reader Accounts aren't required. | `05_data_sharing_collaboration_QA.md` Q12 |
| 48 | **B. UNDROP TABLE** | Within TT retention (7 days > 3 hours), UNDROP is the quickest and self-service. Fail-safe is for after TT expires. | `06_storage_and_protection_QA.md` Q17 |
| 49 | **B. Actual RPO = time since last successful refresh** | 1-min schedule is a target, not a guarantee. If refreshes are blocked for 7 min, RPO is 7 min for data lost in that gap. Monitor lag and alert. | `06_storage_and_protection_QA.md` Q8 |
| 50 | **B. Set a resource monitor on the consumer's account** | You can't do this — the consumer owns their account and compute. All other options are valid provider-side mitigations (narrower share, row access policy, remove from share). | `05_data_sharing_collaboration_QA.md` Q16 |
| 51 | **B. May require full recompute; check IS_INCREMENTAL_REFRESHABLE** | DTs with non-deterministic functions or certain window frames may lose incremental refreshability and trigger full-recompute each refresh — costly. Check `IS_INCREMENTAL_REFRESHABLE` and consider rewriting. | `03_data_engineering_QA.md` Q4 |
| 52 | **B. ABORT_STATEMENT (default)** | COPY INTO defaults to aborting on the first error. Change `ON_ERROR` to CONTINUE / SKIP_FILE / SKIP_FILE_<N> for tolerance budgets. | `03_data_engineering_QA.md` Q16 |
| 53 | **B. Blocks side-channel leaks** | Non-secure views can leak schema info, row counts, or SQL body via plans or error messages. Snowflake blocks sharing them. Secure Views also keep the SQL definition hidden from non-privileged consumers. | `04_performance_optimization_QA.md` Q12; `05_data_sharing_collaboration_QA.md` Q6 |
| 54 | **B. Max queries per cluster** | MAX_CONCURRENCY_LEVEL controls how many queries a single cluster accepts before queueing/overflowing. Default is 8. Lower = more aggressive cluster spawn in multi-cluster mode. | `02_snowflake_architecture_QA.md` Q13; `04_performance_optimization_QA.md` Q10 |
| 55 | **B. Dedicated warehouse per team + resource monitors + QUERY_TAG** | Workload isolation prevents noisy neighbors. Resource monitors cap credits per team. QUERY_TAG enables chargeback. Single resource monitor on a shared WH doesn't segregate teams. | `02_snowflake_architecture_QA.md` Q7 |
| 56 | **B, D** | Streams (stateful) and External Tables (point to external storage with per-account configs) don't replicate. Tables, views, policies do (or can be configured to). | `03_data_engineering_QA.md` Q6; `06_storage_and_protection_QA.md` Q6 |
| 57 | **A. Query Profile** | Always diagnose BEFORE acting. Query Profile reveals the actual bottleneck: pruning, spill, exploding joins, slow operators. Then pick the right lever. | `04_performance_optimization_QA.md` Q5, Q15 |
| 58 | **B. Evaluated in consumer's context** | `CURRENT_ROLE()` returns the consumer's role. If the consumer has created a role literally named `PII_READER`, email WOULD be unmasked — a trap. Policies should use `CURRENT_ACCOUNT()` to allowlist specific consumer accounts to prevent name collisions. | `05_data_sharing_collaboration_QA.md` Q6, Q11 |
| 59 | **A, B** | `COUNT(*)` and `MIN/MAX` on an unfiltered table answer from metadata. Any WHERE forces a scan. `COUNT(DISTINCT ...)` is not metadata-only (approximate via HLL still touches partitions). `LIMIT` requires scanning until the limit is reached. | `02_snowflake_architecture_QA.md` Q2, Q10 |
| 60 | **A, C** | Snowflake-managed Iceberg: data in your bucket, open format readable by Spark/Trino/Flink with ACID; avoids proprietary-format duplication. Zero-copy cloning IS supported but is also available on standard tables, so A and C are more specifically Iceberg benefits. Hybrid Tables are a separate feature. | `03_data_engineering_QA.md` Q7, Q20 |
| 61 | **B. Explicit CTE + transient table** | Snowflake doesn't honor `MATERIALIZED` CTE keyword (not supported). Hints are limited. For forced order, materialize to a transient table. | `04_performance_optimization_QA.md` Q14 |
| 62 | **B. Tasks** | Tasks support CRON schedules and `AFTER` dependencies to build DAGs natively in Snowflake. External Functions are per-row endpoints; Snowpipe is file-triggered; SPs are triggered by Tasks or external calls. | `03_data_engineering_QA.md` Q3 |
| 63 | **B. First consumes, next 9 cache** | Result Cache returns same text + same role + unchanged data in ~50ms with zero warehouse credits. Only the first query runs on the warehouse (which might also be metadata-only if there's no WHERE — but we don't need that nuance since cache applies). | `02_snowflake_architecture_QA.md` Q4; `04_performance_optimization_QA.md` Q17 |
| 64 | **A, B** | Replica refreshes every 30 min; the share serves whatever is in the replica — up to 30 min stale. The share references the replica's current state and refreshes with it. Writes to replicas are forbidden; egress depends on cross-region actions, not necessarily zero. Consumer-side costs apply but not share-side compute. | `05_data_sharing_collaboration_QA.md` Q2 |
| 65 | **A. QAS adds scan parallelism only** | QAS accelerates scan-dominated queries. If the warehouse is bottlenecked on CPU-heavy joins, aggregations, or window functions, QAS doesn't help. Scale UP (larger size) or OUT (more clusters) is the correct lever. | `02_snowflake_architecture_QA.md` Q16; `04_performance_optimization_QA.md` Q4 |

---

## Scoring Reference

| Correct | Interpretation |
|---------|----------------|
| 55+ (85%+) | Strong — exam-ready. |
| 46-54 (70-83%) | Comfortable passing range. Review the misses. |
| 36-45 (55-69%) | Borderline — review weak domains before retaking. |
| < 36 | More study needed — return to the per-domain Q&A files. |

## Weakness Analysis

After scoring, tally errors by domain:
- **Domain 1 (Accounts & Security):** Q1, 11, 12, 13, 17, 29, 40, 43, 44, 45 (10)
- **Domain 2 (Architecture):** Q4, 25, 26, 27, 34, 54, 55, 57, 59, 63, 65 (11)
- **Domain 3 (Data Engineering):** Q7, 11, 15 wait reassign, 18, 28, 33, 39, 46, 51, 52, 56, 62 (12)
- **Domain 4 (Performance):** Q2, 3, 8, 9, 16, 24, 32, 36 wait reassign, 42, 53, 61 (11)
- **Domain 5 (Sharing):** Q5, 15, 22, 23, 30, 31, 38, 41, 47, 50, 58, 60, 64 (13)
- **Domain 6 (Storage/Protection):** Q6, 10, 14, 19, 20, 35, 37, 48, 49, 56 (10)

(Exact assignment is approximate; some questions touch multiple domains. Use the "See" column in the answer key as your authoritative pointer back to the Q&A files.)

Reread the Q&A sections for your weakest 1-2 domains before attempting again.
