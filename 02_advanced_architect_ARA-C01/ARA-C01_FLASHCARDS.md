# ARA-C01 Flashcards

~110 flashcards grouped by domain. Format: question, then one-to-three-sentence answer. Emphasis on architect decision-making: when to use X vs Y, DR/HA tradeoffs, cross-region/cloud, sharing patterns.

Use spaced repetition: cover the answer, read the question, speak the answer aloud, reveal.

---

## Domain 1 — Accounts & Security

### Q: What is the relationship between Organization, Account, and ORGADMIN?
**A:** An Organization groups multiple Accounts under one customer; each Account is a region/cloud deployment with its own URL, users, data. ORGADMIN lives in exactly ONE designated Account and can create/manage accounts and enable replication but cannot access data in other accounts.
---

### Q: Which Snowflake edition is the minimum for PrivateLink?
**A:** Business Critical. PrivateLink requires network isolation features that only BC+ provides. Note: Tri-Secret Secure and Failover Groups also require Business Critical.
---

### Q: Which features require Enterprise edition minimum (not available on Standard)?
**A:** Multi-cluster warehouses, Time Travel beyond 1 day (up to 90), materialized views, dynamic data masking, row access policies, external tokenization, Search Optimization Service, ACCESS_HISTORY.
---

### Q: What is VPS and how does it differ from Business Critical?
**A:** Virtual Private Snowflake extends Business Critical isolation to the Cloud Services layer itself — dedicated metadata and services infrastructure rather than shared multi-tenant CS. Used for FedRAMP-High / defense / top-tier regulated workloads.
---

### Q: Describe the two-layer RBAC pattern Snowflake recommends.
**A:** Access roles grant object privileges (e.g., `sales_db_r` has SELECT on sales tables). Functional roles represent personas (e.g., `analyst_sales`) and are granted the access roles they need. Users receive functional roles. Swapping a user's team changes one grant rather than hundreds.
---

### Q: What does `GRANT ... ON FUTURE TABLES` do, and what overrides what?
**A:** Grants privileges on tables that don't exist yet but will be created in the specified scope. FUTURE grants at the schema level OVERRIDE FUTURE grants at the database level (schema wins).
---

### Q: What is SCIM used for in Snowflake?
**A:** SCIM 2.0 provisions users and groups automatically from an IdP (Okta, Azure AD) into Snowflake. SCIM maps IdP groups to Snowflake ROLES (not users). You still grant object privileges to those roles separately.
---

### Q: Why must `RUN_AS_ROLE` on a SCIM integration NOT be ACCOUNTADMIN?
**A:** Snowflake blocks SCIM from running as ACCOUNTADMIN or SECURITYADMIN to prevent an IdP compromise from gaining account-level control. Create a dedicated provisioner role with just `CREATE USER` and `CREATE ROLE`.
---

### Q: SP-initiated vs IdP-initiated SAML — which is more secure?
**A:** SP-initiated. The user hits Snowflake first, which generates and tracks the SAML request — this prevents some replay attacks. IdP-initiated flows (click-tile) are friendlier but expose a wider replay surface.
---

### Q: What are the two RSA_PUBLIC_KEY slots for and why?
**A:** `RSA_PUBLIC_KEY` and `RSA_PUBLIC_KEY_2` enable zero-downtime key rotation. Register the new key as _2, cut traffic over, verify, then promote by moving the new key to _1 and clearing _2.
---

### Q: Describe Snowflake's 4-level encryption key hierarchy.
**A:** Root Key (HSM) → Account Master Key → Table Master Key → File Key (one per micro-partition). All AES-256. Each level wraps the next. Keys rotate automatically every 30 days for new data.
---

### Q: What does Tri-Secret Secure add, and what's the consequence of revoking the customer key?
**A:** Tri-Secret Secure co-wraps the Account Master Key with a customer-managed key in AWS KMS / Azure Key Vault / GCP KMS. Revoking the customer key is cryptoshredding — data becomes permanently inaccessible, even to Snowflake support. Requires BC+.
---

### Q: When use Network Policy vs PrivateLink?
**A:** Network Policy = IP allowlist/blocklist on the public Snowflake endpoint (filter who can reach it). PrivateLink = a private endpoint inside your VPC that bypasses the public internet entirely. Policy restricts; PrivateLink replaces the route. PrivateLink requires BC+.
---

### Q: What happens if a user has both an account-level and a user-level network policy?
**A:** The user-level policy OVERRIDES the account-level policy for that user. This is a common trap — setting a user policy can accidentally grant broader IP access than the account default.
---

### Q: Custom OAuth vs External OAuth — when to use each?
**A:** Custom OAuth = Snowflake is the authorization server; use for apps written specifically to integrate with Snowflake (operationally simpler). External OAuth = Snowflake trusts tokens from Okta/Azure AD/Ping; use to centralize identity in your existing IdP (unified governance).
---

### Q: What is ACCESS_HISTORY and how is it different from QUERY_HISTORY?
**A:** QUERY_HISTORY is the statement log (who ran what SQL, when, duration). ACCESS_HISTORY (Enterprise+) is the data access log — which columns in which tables were actually touched by each query, including columns in WHERE/JOIN. Use ACCESS_HISTORY for column-level compliance and lineage.
---

### Q: What is the latency on ACCESS_HISTORY, and how long is it retained?
**A:** Up to 3-hour latency; 365-day retention in ACCOUNT_USAGE. For longer retention, schedule unloads to external storage.
---

### Q: What is the difference between SESSION_IDLE_TIMEOUT_MINS and SESSION_UI_IDLE_TIMEOUT_MINS?
**A:** The former applies to programmatic sessions (JDBC, Python, etc.); the latter applies to Snowsight UI sessions. Allows long ETL sessions while timing out abandoned browser tabs quickly.
---

### Q: What are Authentication Policies and how do they relate to older patterns?
**A:** Authentication Policies declaratively restrict which AUTHENTICATION_METHODS (PASSWORD, SAML, OAUTH, KEYPAIR) and CLIENT_TYPES (SNOWSIGHT, DRIVERS) are allowed, scoped to account or user. Replaces the ad-hoc pattern of `UNSET PASSWORD` + hoping users don't come back via other methods.
---

### Q: Compare Dynamic Data Masking, External Tokenization, and Row Access Policies.
**A:** DDM transforms column values at query time (storage keeps raw). External Tokenization replaces values at LOAD time via an external tokenization service (raw never enters Snowflake). RAP filters entire rows based on policy. Compose all three: tokenize load → DDM for role unmask → RAP for tenant isolation.
---

### Q: What is Trust Center and what does it detect vs prevent?
**A:** Trust Center is a Snowsight dashboard running CIS/custom scanners against the account on a schedule, surfacing security misconfigurations (MFA gaps, excessive grants, public shares). It's DETECTIVE, not preventive — it alerts after a misconfig exists.
---

### Q: Who owns what in Snowflake's shared responsibility model?
**A:** Snowflake owns the physical infrastructure, platform encryption, patching, availability, compliance certifications. Customer owns data classification, RBAC, masking, network policies, edition choice, and any secrets they put inside Snowflake.
---

### Q: What's the authentication policy consequence if you include `PASSWORD` in `MFA_AUTHENTICATION_METHODS` only?
**A:** Password cannot be used as a standalone auth method; it only functions during MFA enrollment or as the second factor. Combined with `MFA_ENROLLMENT = REQUIRED` this forces users through MFA for any password usage.
---

## Domain 2 — Snowflake Architecture

### Q: Name the three layers of Snowflake's architecture.
**A:** Cloud Services (brain: auth, metadata, compile, optimize, transactions), Compute (virtual warehouses: MPP clusters), and Centralized Storage (micro-partitions in S3/Blob/GCS). Decoupled, so compute scales without touching data.
---

### Q: When do you pay for Cloud Services credits?
**A:** Cloud Services usage is free up to 10% of your daily compute credits. Beyond that threshold you pay at the standard per-credit rate. Pure metadata query workloads (SHOW loops, tiny statements) can push you over.
---

### Q: Does `SELECT COUNT(*) FROM huge_table` consume warehouse credits?
**A:** No — if there's no WHERE clause, Snowflake answers from metadata and the warehouse doesn't resume. Add any WHERE and it becomes a scan requiring warehouse compute.
---

### Q: Explain WH size vs cluster count.
**A:** Size (XSMALL → 6XL, each step doubles compute) scales single-query parallelism. Cluster count (MIN_CLUSTER_COUNT, MAX_CLUSTER_COUNT) scales concurrent-query throughput. A single query cannot span clusters.
---

### Q: STANDARD vs ECONOMY scaling policy on multi-cluster warehouses?
**A:** STANDARD spawns new clusters immediately on queueing (minimizes wait, higher cost). ECONOMY waits ~6+ minutes of sustained queue before spawning (maximizes utilization, higher queue latency). Pick STANDARD for interactive BI, ECONOMY for batch.
---

### Q: Describe Snowflake's three cache layers in evaluation order.
**A:** (1) Result Cache in Cloud Services — key: query_text + role + underlying data state; 24h TTL. (2) Metadata Cache — powers pruning. (3) Warehouse SSD Cache — per-WH local disk, wiped on suspend or resize. Query path: result → metadata → SSD → remote.
---

### Q: What invalidates the result cache?
**A:** Any underlying table change, non-deterministic functions (CURRENT_TIMESTAMP, UUID_STRING, RANDOM), different role (different privileges may return different rows), `USE_CACHED_RESULT=FALSE`, result >1 GB, different session parameters.
---

### Q: What is a micro-partition's role?
**A:** 50-500 MB (compressed) immutable columnar files, each carrying per-column min/max metadata in Cloud Services. The optimizer prunes partitions whose metadata proves they can't contain matching rows — often skipping 95%+ of a table.
---

### Q: How does Snowflake achieve ACID?
**A:** Snapshot isolation via a transaction manager in Cloud Services. Writes create new MPs (never modify in place) and commit by atomically flipping a metadata pointer. Readers see the pointer at their start time. One isolation level: read-committed by statement, snapshot inside a BEGIN...COMMIT.
---

### Q: Why does scaling up a warehouse not disrupt in-flight queries?
**A:** Resize spawns new nodes; existing queries complete on the old size. New queries use new size. SSD cache on new nodes is cold — benchmarks right after resize miss cache.
---

### Q: What is Query Acceleration Service and when does it help?
**A:** QAS is serverless compute that parallelizes the scan phase of eligible queries (large scans with lightweight post-processing). Enabled per-WH with `ENABLE_QUERY_ACCELERATION=TRUE` and a `QUERY_ACCELERATION_MAX_SCALE_FACTOR`. Billed as serverless credits, separate from WH credits.
---

### Q: What does Query Acceleration NOT help?
**A:** Join-heavy or CPU-bound queries don't benefit. QAS adds scan parallelism only. Also, QAS credits aren't capped by resource monitors on the warehouse.
---

### Q: Explain Snowpark Container Services.
**A:** SPCS runs Docker containers inside Snowflake with direct SQL access to tables, on Snowflake-managed compute pools (CPU or GPU). Used for custom runtimes (R, Julia), ML serving, Streamlit, legacy apps colocated with data — all under Snowflake RBAC.
---

### Q: What are the three stream types and when to use each?
**A:** Standard (I/U/D, full CDC). Append-only (I only, faster for append-heavy). Insert-only (external tables and directory tables — observes file arrivals). Standard for mirror pipelines; append-only for high-throughput event tables.
---

### Q: What does `MAX_CONCURRENCY_LEVEL` control?
**A:** How many queries a single cluster in a multi-cluster warehouse will accept before new queries queue (and in STANDARD mode, trigger an additional cluster). Default is 8. Lower = more aggressive multi-cluster spawn.
---

### Q: What is Snowgrid?
**A:** Snowflake's internal cross-region/cross-cloud mesh that enables replication, failover, cross-cloud sharing, Marketplace auto-fulfillment, and Organization identity. It's Snowflake-managed infrastructure — nothing for customers to configure directly.
---

### Q: Give the Snowflake query lifecycle in order.
**A:** Parse → Resolve → Optimize → Result cache check → Compile → Execute (on warehouse) → Return (populate result cache). All phases except Execute happen in Cloud Services.
---

### Q: Compute credits math: Small for 120 min vs 2XL for 7.5 min — equivalent cost?
**A:** Approximately yes (if query parallelizes linearly). Small=2 cr/hr, 2XL=32 cr/hr; 2×2=4 ≈ 32×0.125=4. Same credits, 16× faster. Verify via query profile that scaling helps.
---

### Q: Why do customers of VPS get stronger isolation than Business Critical?
**A:** VPS adds dedicated Cloud Services (metadata, services) rather than shared multi-tenant CS. BC still shares CS with other tenants (request-isolated but same infrastructure). Needed only for the most-regulated workloads.
---

## Domain 3 — Data Engineering

### Q: Compare bulk COPY, Snowpipe, and Snowpipe Streaming.
**A:** COPY: scheduled, warehouse credits, hours latency, cheapest/GB. Snowpipe: file-triggered serverless, minutes latency, medium cost. Snowpipe Streaming: row-level via Ingest SDK, seconds latency, highest cost/GB. Choose by freshness SLA.
---

### Q: What auth method is required for Snowpipe Streaming?
**A:** Key-pair authentication. Snowpipe Streaming does NOT support password auth. Same requirement applies to the Kafka Connector in streaming mode.
---

### Q: When does a stream become stale?
**A:** When its `STALE_AFTER` deadline passes without consumption. STALE_AFTER = base table's `DATA_RETENTION_TIME_IN_DAYS` + 14 days. A stale stream errors on read; recreate it to continue.
---

### Q: What does `SYSTEM$STREAM_HAS_DATA` do, and in what construct?
**A:** Returns TRUE if the stream has unconsumed rows. Used in a task's `WHEN` clause to skip execution when there's nothing new — the task evaluation runs but SQL doesn't execute, saving credits.
---

### Q: Does `SELECT * FROM stream` advance the stream's offset?
**A:** No. The stream offset only advances when the stream is consumed inside a DML (INSERT/UPDATE/MERGE/DELETE/CTAS) in a committed transaction. SELECT alone is read-only.
---

### Q: Serverless vs user-managed Tasks — pick the right flag for each.
**A:** Serverless: `USER_TASK_MANAGED_INITIAL_WAREHOUSE_SIZE='SMALL'` — Snowflake runs it, serverless billing, auto-tunes size based on history. User-managed: `WAREHOUSE=wh_etl` — uses the specified warehouse, regular compute billing.
---

### Q: Does a newly-created task run immediately?
**A:** No. Tasks are CREATED SUSPENDED. You must `ALTER TASK ... RESUME` to start them. For DAGs, resuming the root resumes the whole DAG.
---

### Q: Dynamic Tables: user-managed or serverless compute?
**A:** User-managed — you specify the WAREHOUSE in the DDL. (This differs from Materialized Views, which use serverless maintenance.)
---

### Q: Dynamic Table vs Materialized View — top differences.
**A:** MV = single table, limited SQL (no joins, limited aggregates), serverless maintenance, auto-rewrites queries. DT = any SQL (joins, windows), target_lag SLA, user-managed WH, chainable (DT on DT), queries reference by name.
---

### Q: What does `TARGET_LAG = DOWNSTREAM` do on a Dynamic Table?
**A:** The intermediate DT defers refresh until a downstream DT needs its data. This avoids unnecessary refreshes on intermediate chain steps — important for cost on multi-hop pipelines.
---

### Q: What is an external volume used for?
**A:** An external volume is a Snowflake-managed credential + path to your cloud bucket, required to create Iceberg tables. Supports encryption config (e.g., AWS SSE-KMS with a specific KMS key).
---

### Q: Snowflake-managed Iceberg vs externally-managed — who writes?
**A:** Snowflake-managed: Snowflake controls the metadata catalog; writes come from Snowflake. Externally-managed (Glue, Nessie): external catalog is authoritative; Snowflake typically reads, writes may be limited based on catalog support.
---

### Q: What's special about Hybrid Tables?
**A:** They combine row-store (OLTP, PK + secondary indexes, enforced constraints) with columnar (analytics, auto-synced). Enables sub-ms point lookups + columnar scans in one table — eliminates the dual-write to OLTP + warehouse.
---

### Q: Default `ON_ERROR` for `COPY INTO`?
**A:** `ABORT_STATEMENT`. The entire COPY fails on the first error. Change to `CONTINUE`, `SKIP_FILE`, or `SKIP_FILE_<N>` / `SKIP_FILE_<N>%` for tolerance budgets.
---

### Q: How does Snowflake prevent duplicate loads of the same file?
**A:** COPY INTO is idempotent on filename within 64 days — the same filename won't be re-loaded. Use `FORCE=TRUE` to override, or change the filename.
---

### Q: Permanent vs Transient vs Temporary tables — key difference for storage.
**A:** Permanent: TT 0-90d + 7-day Fail-safe. Transient: TT 0-1d + NO Fail-safe (globally visible, persistent). Temporary: TT 0-1d + NO Fail-safe + session-scoped (dropped at session end).
---

### Q: Can you convert a Transient table to Permanent after creation?
**A:** No. Table type is immutable. You must recreate the table (`CTAS` into the new type) to change.
---

### Q: External functions — how is batching handled?
**A:** Snowflake batches rows (typically 100-1000 per HTTP call) and parallelizes across warehouse nodes. The request/response shape uses JSON with a `data` key where each row is a `[row_index, col1, col2, ...]` array; the response matches by row_index.
---

### Q: Snowpark Python SP — what are the two execution modes?
**A:** `EXECUTE AS CALLER` (runs with the invoking user's privileges) and `EXECUTE AS OWNER` (default — runs with the SP owner's privileges). OWNER is powerful but risky; CALLER is safer when the SP processes caller-provided object names.
---

### Q: Vectorized Python UDF — what makes it fast?
**A:** The `@vectorized(input=pd.DataFrame)` decorator batches rows into a pandas DataFrame, so the UDF runs once per batch (not per row). Avoids the per-row Python interpreter / serialization overhead. 10-100× faster than scalar Python UDFs for numeric work.
---

### Q: What's the recommended file size range for parallel COPY?
**A:** 100-250 MB compressed per file. Larger files bottleneck one node; smaller files have per-file overhead. Snowflake can parallelize many files across warehouse nodes × threads.
---

### Q: MERGE's default behavior when source has multiple rows for the same target key?
**A:** Errors with "non-deterministic merge" (`ERROR_ON_NONDETERMINISTIC_MERGE=TRUE` default). Fix by deduping source via `QUALIFY ROW_NUMBER() OVER (PARTITION BY key ORDER BY ts DESC) = 1` upstream.
---

### Q: Can a Dynamic Table be built on a Stream?
**A:** No. DTs read tables directly, not streams. Streams are consumption primitives for tasks/procedural pipelines. For CDC-into-DT patterns, insert stream output into a target table, then DT on the target table.
---

### Q: Does `AUTO_REFRESH=TRUE` on an external table work without cloud notifications?
**A:** No. AUTO_REFRESH requires SNS (AWS), EventGrid (Azure), or Pub/Sub (GCP) notifications pointing at a stage auto-ingest ARN. Without notifications, call `ALTER STAGE ... REFRESH` manually or on a task schedule.
---

### Q: How do you share a stream's output when streams can't be shared directly?
**A:** Build a target table consumed from the stream via a task + MERGE/INSERT, then share the target table (not the stream). Alternatively use a Dynamic Table as the sharable, versioned result.
---

## Domain 4 — Performance Optimization

### Q: Natural clustering vs manual clustering — when do you need manual?
**A:** Natural = data arrives sorted by a column (e.g., time), so MPs naturally have tight ranges on that column. Manual = `CLUSTER BY (col1, col2)` + automatic reclustering (serverless) when queries filter on columns the natural load order doesn't help with. Enable manual on >1 TB tables with poor measured clustering depth on query-relevant columns.
---

### Q: What does `SYSTEM$CLUSTERING_INFORMATION(table, '(col)')` return?
**A:** JSON with `total_partition_count`, `average_depth`, `partition_depth_histogram`. Lower average_depth = better pruning. Depth of 1 is perfect; depth of 100 means every MP overlaps 100 others on the key.
---

### Q: Search Optimization Service — when to use?
**A:** Point lookups on high-cardinality columns (equality, IN, substring, geo, VARIANT paths) returning few rows out of billions. Clustering can't help because the predicate column is unrelated to natural order. Enterprise+ only.
---

### Q: What predicates does SOS NOT help with?
**A:** Range scans (`WHERE col BETWEEN x AND y`). Use clustering for ranges. SOS targets equality/IN/substring/geo/VARIANT-path predicates.
---

### Q: Clustering vs SOS cost tradeoff.
**A:** Clustering: serverless credits proportional to new MP arrival rate (reclustering effort). SOS: serverless credits proportional to DML churn (index maintenance). High-churn tables with SOS can exceed query savings — measure before committing.
---

### Q: Pruning killer #1 — what rewrite fixes it?
**A:** Function on column in WHERE. `WHERE DATE(ts) = '2024-01-01'` prevents min/max pruning. Rewrite as range: `WHERE ts >= '2024-01-01' AND ts < '2024-01-02'`.
---

### Q: How does scaling up a WH help with memory spill?
**A:** `bytes_spilled_to_remote_storage > 0` means the WH's aggregate memory was insufficient for sort/join working sets. Larger WH = more nodes = more memory, often eliminates spill. Spill to remote (S3) is especially bad and should be fixed before other tuning.
---

### Q: What's the difference between `queued_overload_time` and `queued_provisioning_time`?
**A:** Overload: WH was saturated at submission — scale up or add clusters. Provisioning: WH was resuming from suspend — expected on first-of-day queries, not a tuning problem.
---

### Q: Scale UP vs scale OUT — which workloads need which?
**A:** Single-query latency improvement = scale UP (size). Many concurrent short queries = scale OUT (clusters). A single query can't span clusters. Test both: run one big query at different sizes; run N queries at different cluster counts.
---

### Q: Why can scaling up fail to help (even on a slow query)?
**A:** Serial operators (recursive CTE, cursor loops), data skew (one key dominates), or small source files (COPY from one giant file). Scale up adds parallelism only when the plan can use it.
---

### Q: How does data skew manifest in a query profile?
**A:** Unequal operator durations across nodes. One node processes dramatically more rows than others. Fix with salting (`HASH(id) % N` added to join key) or filtering/presum before the join. Scaling up usually does NOT help skew.
---

### Q: AUTO_SUSPEND tradeoff — short vs long.
**A:** Short (60s): saves credits during idle but wipes SSD cache more often (next query is cold). Long (300-600s): keeps cache warm but burns credits during longer gaps. Pick by workload shape — BI with bursty concurrency likes longer; batch likes shorter.
---

### Q: Minimum AUTO_SUSPEND value?
**A:** 60 seconds. Smaller values are rounded up. Also: Snowflake bills per-second after the first minute, so sub-minute idle is still charged for a full minute.
---

### Q: Why is `SELECT * FROM t WHERE payload:user_id = 123` often slow?
**A:** VARIANT columns don't have per-field min/max metadata by default. Fix: extract to a typed computed column (then normal pruning works) OR enable SOS on the specific VARIANT path.
---

### Q: What is a "cache hit" for the Result Cache? What invalidates it?
**A:** Hit = same query text + same role + unchanged underlying data + no non-deterministic functions. Invalidation: any DML on referenced tables, DDL on columns, different session parameters (timezone, format), different role.
---

### Q: Query says "is_cache_hit=TRUE" — did it use warehouse credits?
**A:** No. Zero warehouse credits. Result cache returns from Cloud Services in ~50ms. The warehouse doesn't even need to be running.
---

### Q: Is the Result Cache shared across users?
**A:** Yes, account-wide — but only to users with the SAME ROLE for the same query. Different role = different privilege set = different results possible, so the cache misses.
---

### Q: Max result cache TTL?
**A:** 24 hours per use, extended with each hit up to 31 days maximum. Also: results >1 GB are not cached.
---

### Q: What's the SSD cache behavior on resize?
**A:** Resize spawns new nodes with COLD cache. Benchmarks right after resize miss cache. For true comparisons, warm the cache with the workload before measuring.
---

### Q: Where does the UDF overhead come from (slow Python)?
**A:** Scalar Python UDFs cross the Python interpreter boundary per row, with serialization. Vectorized UDFs batch into pandas DataFrames, amortizing the crossing. SQL UDFs inline at optimize time. Order fastest to slowest: SQL → Scala/Java → vectorized Python → scalar Python → external function.
---

### Q: Secure View performance impact?
**A:** Typically single-digit percent slower due to constrained optimization (less aggressive predicate pushdown, UDFs run in protected scope). Secure Views are MANDATORY in shares — not optional.
---

### Q: Materialized View — biggest SQL restrictions.
**A:** Single base table only. No joins. No LIMIT, no ORDER BY. Limited aggregates. No outer joins. No UDFs. If your logic needs any of these, use a Dynamic Table.
---

### Q: Why can CTE materialization be unreliable in Snowflake?
**A:** Snowflake treats CTEs as hints — the optimizer may inline them. There's no `MATERIALIZED` keyword. For guaranteed intermediate materialization, use a transient or temporary table.
---

### Q: Query tag — how do you use it for cost attribution?
**A:** `ALTER SESSION SET QUERY_TAG='team=finance;workload=bi'` tags every subsequent query. Join `QUERY_HISTORY.query_tag` with `WAREHOUSE_METERING_HISTORY` to attribute credits by team/workload. Enforce via role-based session setup.
---

### Q: How do you observe reclustering costs?
**A:** `SNOWFLAKE.ACCOUNT_USAGE.AUTOMATIC_CLUSTERING_HISTORY` tracks reclustering spend per table. Snowflake's automatic clustering is serverless — doesn't appear on warehouse bills.
---

### Q: Dashboard refresh with 50 widgets at 60s intervals — your fix?
**A:** Dynamic Table precomputes the shared dataset at `TARGET_LAG='30 SEC'`; widgets query the DT. First widget scans DT; next 49 hit result cache. Add multi-cluster warehouse (M×2-10 STANDARD) for the refresh spike.
---

## Domain 5 — Data Sharing & Collaboration

### Q: Why is Secure Data Sharing "zero-copy"?
**A:** Consumer's queries authenticate via metadata and read provider's actual micro-partitions through Snowflake's storage path. No files move; no sync pipelines. Consumer provides compute (their warehouse); provider pays nothing for consumer queries.
---

### Q: Can Secure Data Sharing cross regions or clouds?
**A:** No — not directly. Shares work only within the same region of the same cloud. For cross-region/cloud, use database replication + share from the replica, OR use Marketplace/Private Listing with auto-fulfillment.
---

### Q: What is a Reader Account?
**A:** A Snowflake account the provider creates and PAYS FOR, for a consumer who doesn't have their own Snowflake. Up to 20 per provider account. Provider owns all costs; set resource monitors to cap.
---

### Q: Direct Share vs Private Listing vs Marketplace.
**A:** Direct = provider adds consumer by account (1:few known partners). Private Listing = curated discovery, consumer requests, allow-list by account. Marketplace = public catalog, discoverable, monetizable.
---

### Q: What is a Data Clean Room?
**A:** Pattern enforcing privacy-preserving collaboration: multiple parties contribute data, only pre-approved aggregated queries run, minimum aggregation thresholds (k >= N) and optional differential privacy prevent row inference. Snowflake primitives: secure views + row access policies + secure UDFs.
---

### Q: When use a Native App vs a Share?
**A:** Share = read-only data. Native App = packaged logic (stored procedures, UDFs) + optional Streamlit UI + versioning + telemetry (via event sharing) + monetizable. Native App when you ship behavior (ML scoring, benchmarking) rather than just data.
---

### Q: Secure View requirements for shares.
**A:** ALL views in a share MUST be `SECURE`. Non-secure views cannot be shared — the SQL body is hidden and side-channel leaks via error messages are blocked. `GET_DDL` on a secure view returns NULL/permission error for consumers.
---

### Q: What's the only valid grant on a database created `FROM SHARE`?
**A:** `GRANT IMPORTED PRIVILEGES ON DATABASE from_share TO ROLE ...`. Normal GRANT verbs don't apply because the consumer doesn't own the objects.
---

### Q: Can you re-share data you received from a share?
**A:** No — Snowflake blocks direct re-sharing. However, CTAS copies create independent objects that CAN be shared, which bypasses the provider's governance. Prevent via policy (e.g., no CREATE TABLE on consumer roles touching the share).
---

### Q: What objects are NOT shareable?
**A:** Streams, tasks, pipes, sequences, stored procedures, materialized views (historically limited), dynamic tables (limited), hybrid tables (limited), temporary objects. Workaround: build a target table from the stream/task output and share THAT.
---

### Q: Auto-fulfillment of a listing — what does Snowflake do?
**A:** When provider enables auto-fulfill on a listing and consumer in a target region accepts, Snowflake automatically creates a database replica in the consumer's region and keeps it refreshed. Consumer queries locally with no cross-region latency.
---

### Q: How does masking work when sharing to a consumer?
**A:** Masking policies evaluate `CURRENT_ROLE()` in the CONSUMER's context. Since the consumer's role doesn't match the policy's allowlist (they can't grant themselves UNMASK on a provider policy), they see masked values regardless of what SQL they run. Policy owner is the provider.
---

### Q: How can a provider restrict rows per consumer account in a share?
**A:** Row Access Policy using `CURRENT_ACCOUNT()`, which returns the CONSUMER's account locator when evaluated in their context. Allowlist specific consumer accounts to specific row partitions.
---

### Q: How do you measure consumer engagement with a listing?
**A:** `SNOWFLAKE.ACCOUNT_USAGE.LISTING_CONSUMPTION_HISTORY` (per listing × consumer × day, query count and bytes), `LISTING_EVENTS_DAILY` (view/request/install funnel), `LISTING_ACCESS_HISTORY` (per-event). Latency up to 48 hours.
---

### Q: Reciprocal shares vs Clean Room — key difference.
**A:** Reciprocal = each party shares to the other, each sees aggregates of the other (symmetric trust). Clean Room = both share to a neutral environment that only runs pre-approved aggregated queries with min-k threshold (low-trust, formal privacy guarantees).
---

### Q: Native App upgrade path?
**A:** Provider publishes a new version. Consumer upgrades voluntarily, or via an auto-update policy on their application. Versioning + manifest tracking lets the provider ship evolution without breaking existing consumers.
---

### Q: Secure UDF in a share — can the consumer see the body?
**A:** No. SECURE UDFs hide the SQL/code body from non-privileged users and block side-channel leaks via query plans or error messages. Consumer invokes the function on their data; implementation stays confidential.
---

### Q: Can you include SHARES in a Failover Group?
**A:** Yes. `OBJECT_TYPES = (DATABASES, SHARES, ...)` includes the share definitions. On failover, shares in the DR account continue serving — but consumer accounts may need to reconfigure to the DR account's URL unless auto-fulfill listings are used.
---

### Q: Compute cost on a share — who pays?
**A:** Consumer pays, EXCEPT in a Reader Account where the provider pays all costs. On regular shares, the consumer uses their own warehouse to query the shared data.
---

## Domain 6 — Storage & Protection

### Q: Micro-partition lifecycle in one sentence.
**A:** ACTIVE → TIME TRAVEL (0-90 days, customer-queryable) → FAIL-SAFE (7 days, Snowflake-support only, permanent tables only) → gone.
---

### Q: Max Time Travel retention by edition / table type?
**A:** Standard: 1 day max. Enterprise+: up to 90 days for permanent tables. Transient and temporary: 1 day max regardless of edition.
---

### Q: Is Fail-safe configurable?
**A:** No. Always 7 days for permanent tables. Always 0 for transient and temporary. Cannot be shortened or extended. Access only via Snowflake support ticket (not self-service).
---

### Q: What does `AT(STATEMENT => '<query_id>')` give you?
**A:** The table state immediately AFTER that statement completed. `BEFORE(STATEMENT => ...)` gives the state immediately before — useful to see what a table looked like before a bad UPDATE.
---

### Q: Can you Time Travel through a DDL (like DROP COLUMN)?
**A:** No. Time Travel is for DML (row-level changes). DDL metadata changes are not reversible via `AT`/`BEFORE`. Mitigate by cloning before DDL (`CREATE TABLE t_bak CLONE t`).
---

### Q: What does zero-copy CLONE actually do?
**A:** Creates a new table pointing at the same micro-partitions as the source. No data copies; only metadata. Copy-on-write: subsequent writes to either side create new MPs owned by that side. Shared MPs stay as long as either side references them.
---

### Q: Clone + COPY GRANTS — what happens without it?
**A:** Without `COPY GRANTS`, the clone has no inherited grants (whoever created the clone owns it, no one else has access). With `COPY GRANTS`, current grants on the source are copied to the clone (future grants are not copied).
---

### Q: Does cloning a stream carry its offset?
**A:** No. A cloned stream has a fresh (empty) offset. A cloned task is created SUSPENDED. Cloning preserves the object definition but not runtime state.
---

### Q: Dev/test refresh pattern using clone.
**A:** Nightly: `DROP DATABASE dev; CREATE DATABASE dev CLONE prod;`. Instant, no storage copy. Per-developer clones from dev: `CREATE DATABASE dev_alice CLONE dev;`. Isolation at near-zero marginal cost.
---

### Q: What's the minimum edition for Database Replication (read-only)?
**A:** Enterprise. For Failover Groups (promotable DR), you need Business Critical on BOTH sides.
---

### Q: Database Replication vs Failover Group — one-sentence difference.
**A:** Database Replication = one-way read-only replica (any Enterprise). Failover Group = bundle of DBs + shares + roles + users + warehouses with promotable secondary (BC+ only).
---

### Q: Replication Group vs Failover Group.
**A:** Replication Group (Enterprise+) replicates for read scale-out; target is permanently read-only. Failover Group (BC+) does the same + supports `ALTER FAILOVER GROUP ... PRIMARY` promotion for DR.
---

### Q: What minimum REPLICATION_SCHEDULE achieves what RPO?
**A:** Minimum is 1 minute. Practical RPO = replication schedule (data loss up to one refresh interval). Aggressive schedules cost more egress on cross-region/cloud. For 5-min RPO, use 1-min schedule to leave headroom.
---

### Q: What's the typical RTO on a well-tested failover?
**A:** 5-15 minutes end-to-end. Metadata flip (`ALTER FAILOVER GROUP ... PRIMARY`) is seconds; the rest is consumer-side DNS/config switching + validation.
---

### Q: Can users/roles be in multiple failover groups?
**A:** No. Account-level objects (users, roles, warehouses) belong to AT MOST ONE failover group. This defines "the unit of DR" — all go together or none do.
---

### Q: What's `retained_for_clone_bytes` in TABLE_STORAGE_METRICS?
**A:** Bytes of micro-partitions kept because a clone (or Time Travel on a clone) references them, even though the source table has already evolved past. Dropping the clone frees those bytes.
---

### Q: Can you UNDROP after Time Travel expires?
**A:** No. UNDROP works within the TT retention window only. After retention expires, the object is in Fail-safe (if permanent) and only Snowflake support can restore it — a support ticket, not self-service.
---

### Q: What happens if you UNDROP but a new object exists with the same name?
**A:** UNDROP fails. Drop or rename the current object first, then UNDROP the historical one.
---

### Q: Does UNDROP DATABASE restore all contained tables?
**A:** Yes, transitively — all schemas and tables that still existed at the drop time, provided their own retention hasn't lapsed.
---

### Q: Why might cross-cloud DR be preferred over cross-region same-cloud?
**A:** Protects against cloud-provider failures or vendor-level outages. Trade-off: higher egress, more operational complexity, usually slower replication. Most customers use same-cloud cross-region; cross-cloud is for extreme threat models.
---

### Q: What tables should NOT get `DATA_RETENTION_TIME_IN_DAYS=90`?
**A:** High-churn staging tables, scratch tables, and anything rebuildable from source. Long retention on churny tables bloats Time Travel storage dramatically. Use TRANSIENT type (TT max 1 day, no FS) for these.
---

### Q: How does Time Travel interact with Automatic Clustering?
**A:** Reclustering writes new MPs; old MPs enter Time Travel for the retention window. Heavy clustering activity on tables with long retention inflates TT storage. Monitor `TABLE_STORAGE_METRICS.time_travel_bytes` and consider shortening retention or disabling clustering on non-essential tables.
---

### Q: External tables and Snowflake storage billing — relationship?
**A:** External tables contribute ZERO Snowflake storage bytes. Data lives in your bucket; you pay your cloud provider's storage. Snowflake charges only for the metadata it maintains. Iceberg tables with an external volume are similar.
---

### Q: Can you reduce retention and immediately reclaim storage?
**A:** No. Reducing retention affects FUTURE DML — existing versions age out at the OLD rate. Reclaiming is gradual over days. Immediate reduction requires UNSET + wait or recreate the table.
---

## Cross-Domain Decision Checks

### Q: Scenario — Bank wants PrivateLink, Tri-Secret Secure, cross-region DR with 1-hour RPO. Edition?
**A:** Business Critical on both primary and DR accounts under one Organization. All three requirements (PrivateLink, Tri-Secret, Failover Group) require BC.
---

### Q: Partner has no Snowflake account, needs to query live data daily. Choice?
**A:** Reader Account. Provider creates it, pre-provisions users/roles, pays all compute. Set resource monitor to cap costs. Up to 20 Reader Accounts per provider account.
---

### Q: Snowpipe Streaming authentication requirement?
**A:** Key-pair auth. The Ingest SDK does not support password auth. Common exam trap: "configure Snowpipe Streaming with username/password" is wrong.
---

### Q: Query against a 10 TB table filters by region (low cardinality) AND order_id (high cardinality). Both patterns common. Fix?
**A:** `CLUSTER BY (region)` for the range/low-cardinality filter + `SEARCH OPTIMIZATION ON EQUALITY(order_id)` for the point lookup. The two services complement each other.
---

### Q: 50 widgets × 60s refresh × 2s heavy query each — WH can't keep up. Primary fix?
**A:** Precompute via Dynamic Table (TARGET_LAG='30 SEC'), widgets query DT, most hit result cache. Add multi-cluster M×2-10 STANDARD for refresh-time concurrency.
---

### Q: Cross-region replication is lagging past the 5-min SLA. First check?
**A:** `TABLE(INFORMATION_SCHEMA.REPLICATION_GROUP_REFRESH_HISTORY('fg_name'))` for failed/stuck phases. Check egress bandwidth, source compute availability (serialization), DDL in flight (large DDL blocks replication). Set up a task-based alert on lag > SLA.
---

### Q: You discover a service account's private key was committed to Git. First four actions in order?
**A:** (1) Register new key in RSA_PUBLIC_KEY_2 slot. (2) Review LOGIN_HISTORY in exposure window for anomalies (do this BEFORE disabling). (3) Rotate CI/CD to new key, promote to RSA_PUBLIC_KEY, UNSET _2. (4) Apply Authentication Policy restricting to KEYPAIR only + Network Policy to CI runner CIDR.
---

### Q: Which Snowflake primitive lets consumers run a provider's proprietary ML scoring on consumer data without exposing either?
**A:** Native App with Secure UDFs (or a Streamlit UI). Provider ships code; consumer grants USAGE on their data to the APPLICATION role. Code runs in consumer's account; raw features and model internals stay confidential.
---

### Q: Why is tagging PII columns important for governance of shared data?
**A:** Tag-based masking policies apply uniformly to every tagged column — scalable governance. Classification (CALL SNOWFLAKE.CORE.CLASSIFY_SCHEMA) can auto-tag. Combined with secure views, consumers see masked values regardless of query shape.
---

### Q: Difference between Snowflake's shared responsibility model and your SOC2 obligations?
**A:** Snowflake's SOC2 covers Snowflake's controls (infrastructure, platform encryption, patching). Your SOC2 covers YOUR use of Snowflake — your RBAC, masking, network policies, user offboarding, retention choices. Both exist; neither substitutes for the other.
---
