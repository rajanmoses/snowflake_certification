# DEA-C02 Flashcards

Rapid-review cards distilled from the five domain Q&A files. Flip, speak the answer, verify. Target: finish all cards in <40 min with >90% accuracy before sitting the exam.

Legend: domain-group counts shown at each header. Format `Q: question` → `A: 1-3 sentence answer`.

---

## MUST-KNOW HOTLIST (memorize before anything else) — 15 cards

### Q: Snowpipe file-history dedup window?
**A:** 14 days. After day 15 the same filename reloads. Compare to COPY INTO which dedups via LOAD_HISTORY for 64 days.
---
### Q: COPY INTO LOAD_HISTORY dedup window?
**A:** 64 days per-table.
---
### Q: Fail-safe window and who can access it?
**A:** 7 days, non-configurable, non-user-accessible — recoverable only via Snowflake Support. Permanent tables only.
---
### Q: Max Time Travel on Enterprise+?
**A:** 90 days. Standard edition caps at 1 day.
---
### Q: Warehouse billing minimum?
**A:** 60-second minimum per start/resume, then per-second billing. `AUTO_SUSPEND < 60` still bills a 60s floor.
---
### Q: Credit multipliers XS through 6XL?
**A:** 1, 2, 4, 8, 16, 32, 64, 128, 256, 512. Doubles per T-shirt step.
---
### Q: Snowpark-optimized warehouse memory bump?
**A:** ~16x per-core memory. Minimum size: Medium. Cost ~1.5x standard.
---
### Q: Serverless task minimum billing?
**A:** 1-minute minimum per task run.
---
### Q: Cloud services daily subsidy?
**A:** Up to 10% of daily warehouse compute credits — above that, cloud services are billable.
---
### Q: Snowpipe Streaming latency and semantics?
**A:** <10 seconds, exactly-once per channel via offset tokens.
---
### Q: Snowpipe auto-ingest latency and semantics?
**A:** ~1 minute, at-least-once (dedup by filename).
---
### Q: Default DATA_RETENTION_TIME_IN_DAYS for permanent / transient / temporary?
**A:** Permanent: 1 (configurable 0–90). Transient: 1 (max 1, no FS). Temporary: 1 (session-scoped, no FS).
---
### Q: Max VARIANT size per row?
**A:** 16 MB compressed.
---
### Q: Micro-partition target size?
**A:** 50–500 MB uncompressed, ~16 MB compressed typical.
---
### Q: Transient table storage savings vs permanent?
**A:** No Fail-safe tier (~30% storage reduction depending on churn).
---

## Domain: Data Movement (~27%) — 24 cards

### Q: Three Snowpipe delivery modes?
**A:** Auto-ingest (cloud event notification via SQS/EventGrid/Pub-Sub), REST (`insertFiles` API call), Streaming (SDK writing directly to micro-partitions).
---
### Q: Snowpipe REFRESH lookback window?
**A:** 7 days — files uploaded in the last week can be re-enqueued; older history needs manual COPY.
---
### Q: Three Stream types and what each tracks?
**A:** Standard (inserts/updates/deletes with net CDC), Append-only (inserts only), Insert-only (external tables only).
---
### Q: What does `METADATA$ACTION` return?
**A:** 'INSERT' or 'DELETE' from a stream. Combined with `METADATA$ISUPDATE=TRUE` to flag update pairs.
---
### Q: `SHOW_INITIAL_ROWS = TRUE` on stream creation — what does it do?
**A:** First read of the stream after creation returns all existing rows as inserts; after first consume, acts normally.
---
### Q: `SYSTEM$STREAM_HAS_DATA` purpose?
**A:** Boolean pre-check — set as a Task's `WHEN` clause to skip scheduled runs when no new CDC data exists (free scheduler check).
---
### Q: Task Graph: resume order?
**A:** Resume child tasks first, then root. Suspend in reverse (root first, children after). Snowflake enforces this.
---
### Q: Task Graph max tasks and depth?
**A:** 1000 tasks per graph, 100 levels deep.
---
### Q: Serverless vs user-managed task — one key cost tradeoff?
**A:** Serverless has 1-minute minimum per run; good for sporadic workloads. User-managed shares warehouse and 60s minimum per resume cycle; good for dense pipelines on an already-warm WH.
---
### Q: `MATCH_BY_COLUMN_NAME` purpose?
**A:** Maps JSON/Parquet/Avro/ORC fields to table columns by name instead of position. Combined with `ENABLE_SCHEMA_EVOLUTION=TRUE` for auto-adding columns.
---
### Q: Privilege required to evolve schema during load?
**A:** `EVOLVE SCHEMA` privilege (distinct from INSERT) on the target table.
---
### Q: `ON_ERROR` default for `COPY INTO`?
**A:** `ABORT_STATEMENT` — fail entire COPY on first error.
---
### Q: `VALIDATION_MODE` options for COPY?
**A:** `RETURN_ERRORS` (list parse errors, no insert), `RETURN_N_ROWS` (preview N loadable rows). Snowpipe does **not** support VALIDATION_MODE.
---
### Q: `MAX_FILE_SIZE` default for COPY INTO @stage (unload)?
**A:** 16 MB uncompressed (compressed cap 5 GB with `SINGLE=TRUE`).
---
### Q: External Table vs Iceberg Table — one differentiator?
**A:** External Table is read-only metadata federation (no DML). Iceberg supports full DML + ACID via snapshots and interop with Spark/Trino.
---
### Q: Directory Table purpose?
**A:** Implicit metadata table on a named stage exposing `relative_path`, `size`, `last_modified`, `md5`, `file_url`. Needed for unstructured file workflows.
---
### Q: Snowpipe states and what triggers each?
**A:** RUNNING, SUSPENDED (`ALTER PIPE SUSPEND`), STOPPED_CLONED_OR_DROPPED (target dropped or pipe cloned), PAUSED_BY_SYSTEM (owner role dropped), PAUSED (`PIPE_EXECUTION_PAUSED=TRUE`).
---
### Q: SQS retention for Snowpipe missed events?
**A:** 4 days — beyond that use `ALTER PIPE REFRESH` (7-day lookback).
---
### Q: Kafka connector streaming mode auth requirement?
**A:** Key-pair auth (RSA) — passwords not allowed for Snowpipe REST / Streaming.
---
### Q: Dynamic Table `TARGET_LAG = DOWNSTREAM` meaning?
**A:** Refresh only when a dependent DT needs me — minimizes upstream refresh cost in chained DAGs.
---
### Q: Does a Dynamic Table consume a Stream?
**A:** No. DTs use change tracking directly on source tables, not stream objects.
---
### Q: What SQL constructs force full refresh on Dynamic Tables?
**A:** Non-deterministic functions, correlated subqueries, some window functions, `SELECT *` on evolving sources.
---
### Q: Three latency/cost options for continuous ingest?
**A:** COPY (min–hours, WH credits), Snowpipe (~1 min, per-file + compute), Snowpipe Streaming (<10s, per-row + compute, exactly-once).
---
### Q: Late-arriving data dedup pattern in MERGE?
**A:** `QUALIFY ROW_NUMBER() OVER (PARTITION BY business_key ORDER BY event_time DESC) = 1` on source, then compare target.event_time in MATCHED clause.
---

## Domain: Performance Optimization (~22%) — 22 cards

### Q: Three caches in Snowflake?
**A:** Metadata cache (serverless, free, min/max/null/count stats), Result cache (24h, extends to 31d with reuse, invalidated by DML/role change), Warehouse SSD cache (per-WH, cleared on suspend/resize).
---
### Q: Result cache TTL?
**A:** 24 hours idle; extended up to 31 days if reused.
---
### Q: What disables result cache for a query?
**A:** Any underlying DML, role/permission changes, non-deterministic functions (`CURRENT_TIMESTAMP`, `UUID_STRING`, etc.), different session context.
---
### Q: Pruning killer — predicate anti-patterns?
**A:** Function-wrapping the column (`WHERE UPPER(col)='X'`, `WHERE DATE_TRUNC('day', col)=...`), implicit casts in both operands, OR across columns.
---
### Q: Clustering depth target?
**A:** ~1 (ideal). >10 signals a clustering problem on a filtered column.
---
### Q: Rule of thumb for cluster key column order?
**A:** 2–4 columns, lowest cardinality first, aligned with top WHERE/JOIN predicates.
---
### Q: Clustering credits — where tracked?
**A:** `SNOWFLAKE.ACCOUNT_USAGE.AUTOMATIC_CLUSTERING_HISTORY` — separate from warehouse credits.
---
### Q: Search Optimization Service — when to use?
**A:** Highly selective point lookups (<1% rows returned) on non-clustering columns. Supports EQUALITY, SUBSTRING, VARIANT path, geospatial.
---
### Q: Materialized View restrictions?
**A:** Single table, no joins, no UNION, no window functions, no HAVING/ORDER BY, no non-deterministic UDFs.
---
### Q: Dynamic Table vs Materialized View — one key difference?
**A:** DT supports joins, chaining, full SQL. MV is single-table + limited aggregations but has transparent query rewrite.
---
### Q: Query Profile red flag #1?
**A:** `bytes_spilled_to_remote_storage > 0` — resize WH up or use Snowpark-optimized.
---
### Q: Local disk spill — concerning?
**A:** Usually acceptable (SSD-speed). Only remote spill signals urgent resize.
---
### Q: "Double warehouse test" rule?
**A:** If doubling WH size cuts runtime >40%, keep going. <20% improvement = over-provisioned.
---
### Q: Multi-cluster vs scaling up — difference?
**A:** Multi-cluster adds horizontal concurrency (more clusters, same size). Scaling up (size) accelerates per-query compute.
---
### Q: Scaling policy STANDARD vs ECONOMY?
**A:** STANDARD spins up a new cluster on first queue hit (latency priority). ECONOMY waits ~6 min queued (cost priority).
---
### Q: `MAX_CONCURRENCY_LEVEL` default?
**A:** 8 concurrent queries per cluster. Lower to force earlier scale-out for many-small-queries workloads.
---
### Q: Join algorithm chosen by Snowflake?
**A:** Hash join for equi-joins (default), nested-loop for non-equi (`<`, `BETWEEN`, inequality). Broadcast vs shuffle decided by CBO on size stats.
---
### Q: Query Acceleration Service — eligible queries?
**A:** Scan-heavy, selective filters on large tables, some aggregations. Not UDFs, not high-cardinality DISTINCT, not small queries.
---
### Q: `QUERY_HASH` vs `QUERY_PARAMETERIZED_HASH`?
**A:** HASH preserves distinct literals; PARAMETERIZED_HASH treats literals as placeholders — use parameterized to group logically-same queries.
---
### Q: Resource monitor levels + actions?
**A:** `NOTIFY` (email), `SUSPEND` (let in-flight finish), `SUSPEND_IMMEDIATE` (kill in-flight). Monitor at account or warehouse level.
---
### Q: Does a resource monitor cap serverless/Snowpipe credits?
**A:** No — only warehouse credits. Snowpipe, auto-cluster, SOS maintenance are not resource-monitor-capped.
---
### Q: Snowpark-optimized WH credit cost multiplier?
**A:** ~1.5x of standard same-size WH, with 16x per-core memory.
---

## Domain: Storage & Data Protection (~18%) — 18 cards

### Q: What makes zero-copy clones "zero-copy"?
**A:** Immutable micro-partitions — clone shares all MPs with source via metadata pointers. Only diverged MPs cost extra storage.
---
### Q: Does a clone inherit grants?
**A:** No, unless you specify `COPY GRANTS` option.
---
### Q: Does a clone inherit the source's Time Travel history?
**A:** No. Clone's TT starts fresh at clone time. For historical access, clone AT (TIMESTAMP) a specific past point.
---
### Q: `UNDROP` fails under what condition?
**A:** When a new object with the same name has been created after the drop — the name is taken. Also fails if beyond retention.
---
### Q: Storage breakdown components?
**A:** Active, Time Travel, Fail-safe, retained_for_clone, staging (internal stages), replicated.
---
### Q: Changing retention from 90d to 7d — what happens?
**A:** MPs aged beyond 7d immediately move to Fail-safe (if permanent) and drain over the next 7 days. Short-term storage spike possible.
---
### Q: `MAX_DATA_EXTENSION_TIME_IN_DAYS` purpose?
**A:** Extends retention beyond base `DATA_RETENTION_TIME_IN_DAYS` (up to 90 days) when a dependent object (stream, DT) needs older data — prevents stream staleness.
---
### Q: Edition for cross-region replication?
**A:** Enterprise (same cloud). Cross-cloud requires Business Critical.
---
### Q: Edition for Tri-Secret Secure?
**A:** Business Critical.
---
### Q: What's in a failover group's `OBJECT_TYPES`?
**A:** DATABASES, ROLES, USERS, WAREHOUSES, INTEGRATIONS, RESOURCE_MONITORS, NETWORK_POLICIES, SHARES.
---
### Q: Hybrid Table primary use case?
**A:** OLTP-style single-row writes and PK lookups (Unistore). Row-store + columnar mirror; sub-millisecond PK reads.
---
### Q: Can you query a shared table with Time Travel?
**A:** Yes — consumers can use `AT/BEFORE` within the provider's retention window.
---
### Q: Can you CLONE a shared table?
**A:** No. Use CTAS to materialize into consumer's own account.
---
### Q: DML on an MP physically does what?
**A:** Writes a new MP with modifications; old MP retained for TT. One-row updates rewrite the entire 500MB MP.
---
### Q: Encryption key hierarchy tiers?
**A:** Cloud KMS root → Account master → Table master → File (MP) keys. Rotated every 30 days.
---
### Q: Tri-Secret Secure revocation risk?
**A:** If customer-managed root key is revoked, data is permanently unreadable — Snowflake Support cannot recover.
---
### Q: Where to find storage usage by DB over time?
**A:** `SNOWFLAKE.ACCOUNT_USAGE.DATABASE_STORAGE_USAGE_HISTORY` (365-day retention, daily averages).
---
### Q: Can you convert a permanent table to transient?
**A:** Not in place — must create new transient + copy data + drop original (lose history).
---

## Domain: Data Transformation (~18%) — 20 cards

### Q: Scalar vs Vectorized Python UDF — key diff?
**A:** Vectorized receives a pandas DataFrame batch (~1000+ rows per call), amortizing Python interpreter + import overhead ~100x. Scalar runs row-by-row.
---
### Q: UDTF purpose?
**A:** Table-valued UDF returning multiple rows per input row. Supports partitioning for stateful windowing. Use for explode/split/generate.
---
### Q: Can a Python UDF execute SQL?
**A:** No. Only pure Python. For SQL execution, use a Python stored procedure (Snowpark).
---
### Q: Snowpark DataFrame — lazy or eager?
**A:** Lazy. Only `.collect()`, `.show()`, `.save_as_table()`, `.write.save()` trigger execution.
---
### Q: Does Snowpark execute Python row-by-row?
**A:** No — DataFrame ops compile to SQL and run on the warehouse. Only UDFs run in Python sandboxes.
---
### Q: SPROC `EXECUTE AS OWNER` vs `EXECUTE AS CALLER`?
**A:** OWNER = runs with proc owner's privileges (SUID-like escalation). CALLER (default) = caller's privileges. Owner-rights procs need careful SQL-injection review.
---
### Q: Dynamic Table refresh modes?
**A:** INCREMENTAL (default when SQL is incrementalizable) or FULL. `REFRESH_MODE = AUTO | INCREMENTAL | FULL` controls selection.
---
### Q: Iceberg catalog modes in Snowflake?
**A:** Snowflake-managed (Snowflake owns catalog, best perf, full R/W) or Externally-managed (Glue/REST/Nessie — multi-engine coordination; Snowflake often read-only).
---
### Q: SCD2 MERGE: close-old + insert-new in one MERGE?
**A:** No — MERGE can only one action per source row. Need two statements (close current via MERGE, then insert new current via INSERT ... FROM stream).
---
### Q: dbt materializations supported?
**A:** view, table, incremental, ephemeral, snapshot. Snowflake-specific: dynamic_table, iceberg_table, Snowpark Python models.
---
### Q: dbt `on_schema_change` options?
**A:** `ignore`, `fail`, `append_new_columns`, `sync_all_columns`. Controls behavior when incremental model sees new source columns.
---
### Q: Snowflake Scripting — what blocks look like?
**A:** `DECLARE ... BEGIN ... EXCEPTION WHEN ... THEN ... END` with LET, IF/THEN, FOR/WHILE/CURSOR loops, EXECUTE IMMEDIATE for dynamic SQL.
---
### Q: `SQLROWCOUNT` purpose?
**A:** Returns row count from the last DML executed inside a Snowflake Scripting block.
---
### Q: Snowpark package pinning syntax?
**A:** `PACKAGES = ('pandas==1.5.3','scikit-learn==1.3.0')` in CREATE FUNCTION/PROCEDURE.
---
### Q: UDF volatility types?
**A:** IMMUTABLE (same input → same output always, cacheable aggressively), STABLE (deterministic within a query), VOLATILE (unpredictable, no caching). Default: VOLATILE unless marked.
---
### Q: Secure UDF effect?
**A:** Hides body from non-owners (via DESC, EXPLAIN). Some pushdown optimizations restricted to prevent info leakage.
---
### Q: External function vs external access integration — pick one?
**A:** External Function: synchronous call to pre-configured Lambda/Azure/GCP via API integration. External Access Integration: outbound HTTP from UDFs/procs via network rules + secrets. Latter is newer, more flexible.
---
### Q: Streamlit in Snowflake auth model?
**A:** Inherits active session user + role via `get_active_session()`. No external credential management.
---
### Q: `CREATE OR REPLACE FUNCTION` risk?
**A:** Atomically replaces body but affects all callers immediately. Prefer version-suffixed names (`fn_v1`, `fn_v2`) with a wrapper for safe rollout.
---
### Q: Snowpark Python stored proc minimum WH size for Snowpark-optimized?
**A:** Medium. Cannot convert existing WH — must create new with `WAREHOUSE_TYPE='SNOWPARK-OPTIMIZED'`.
---

## Domain: Security (~15%) — 18 cards

### Q: System role hierarchy top to bottom?
**A:** ACCOUNTADMIN → SECURITYADMIN (manages grants/roles), USERADMIN (creates users/roles), SYSADMIN (warehouses/DBs), PUBLIC. Custom functional/access roles branch off SYSADMIN.
---
### Q: Difference between GRANT TO ALL TABLES and GRANT TO FUTURE TABLES?
**A:** `ALL TABLES` is a point-in-time snapshot — only covers existing tables. `FUTURE TABLES` covers newly-created tables going forward. Standard pattern uses both.
---
### Q: Who owns an object by default?
**A:** The role that executed the CREATE statement. Transferable via `GRANT OWNERSHIP ... TO ROLE ... [COPY|REVOKE] CURRENT GRANTS`.
---
### Q: Can a column have multiple masking policies?
**A:** No — one masking policy per column. One policy can protect many columns.
---
### Q: Masking vs Row Access Policy?
**A:** Masking changes VALUES per cell (hides data in a column). RAP filters ROWS entirely (hides records matching policy).
---
### Q: `CURRENT_ROLE()` vs `IS_ROLE_IN_SESSION()`?
**A:** `CURRENT_ROLE()` returns only the primary role. `IS_ROLE_IN_SESSION('X')` returns TRUE if X is primary OR secondary. Use the latter in policies when secondary roles should count.
---
### Q: OAuth flow for Snowflake?
**A:** Client gets JWT access token from IdP (Okta/Azure AD/etc.), presents it to Snowflake. Snowflake validates signature via JWKS URL, maps scope to role, establishes session.
---
### Q: SCIM purpose?
**A:** REST API for IdP to auto-provision users and groups into Snowflake. IdP push creates/updates/deletes users; Snowflake-managed by SCIM cannot be edited manually.
---
### Q: `BLOCKED_IP_LIST` vs `ALLOWED_IP_LIST` precedence?
**A:** BLOCKED wins when they overlap.
---
### Q: Private Link edition requirement?
**A:** Business Critical (AWS/Azure) or Enterprise (GCP).
---
### Q: Where to find column-level access audit?
**A:** `SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY` — `base_objects_accessed` (per-column), `objects_modified`. Enterprise+.
---
### Q: Tag-based masking policy — how it works?
**A:** Attach masking policy to a tag; any column tagged with that tag auto-gets the policy. One ALTER TAG covers many columns.
---
### Q: Service user hardening defaults?
**A:** `TYPE = SERVICE` (disallows password), key-pair auth (`RSA_PUBLIC_KEY`), network policy restricting IPs, default role minimum-privilege.
---
### Q: Max RSA public keys per user?
**A:** 2 (`RSA_PUBLIC_KEY` + `RSA_PUBLIC_KEY_2`) — for zero-downtime rotation.
---
### Q: Secure view purpose?
**A:** Hides view definition from non-owners (via DESC, EXPLAIN). Required for views shared externally. Some pushdown restricted.
---
### Q: ACCESS_HISTORY latency?
**A:** Up to 3 hours in Account Usage.
---
### Q: External access integration components?
**A:** Network rule (allowed hosts list), Secret (credentials), Integration object binding them. UDF/proc references the integration + specifies secret names.
---
### Q: GRANT OWNERSHIP options?
**A:** `COPY CURRENT GRANTS` (default — preserve existing grants) or `REVOKE CURRENT GRANTS` (clear and start fresh).
---

## Rapid Random Mix — 10 extra cards

### Q: What triggers an orphaned future grant?
**A:** Dropping the role that created the future grant. New objects stop getting the grant.
---
### Q: Cheapest storage for intermediate pipeline tables?
**A:** Transient tables — no Fail-safe (~30% savings vs permanent).
---
### Q: Can two Snowpipes share one SQS queue?
**A:** Yes, if listening to disjoint prefixes.
---
### Q: What invalidates an auto-clustering table into "no benefit"?
**A:** Clustering key with very high cardinality (UUIDs, millisecond timestamps) — constant reclustering with little pruning gain.
---
### Q: What breaks a stream permanently?
**A:** `CREATE OR REPLACE TABLE` on source, or letting stream offset age beyond `DATA_RETENTION + MAX_DATA_EXTENSION`.
---
### Q: Query that uses zero warehouse compute?
**A:** `SELECT COUNT(*) FROM t` (metadata), `SHOW TABLES`, `DESC TABLE`, cached result reuse — all cloud services only.
---
### Q: What does `OVERWRITE = TRUE` do in COPY INTO @stage (unload)?
**A:** Overwrites existing files in the stage with the same name; without it, duplicate names fail.
---
### Q: Max number of tasks in a graph / depth?
**A:** 1000 tasks, 100 levels deep.
---
### Q: Can a Dynamic Table be a source of another Dynamic Table?
**A:** Yes — DTs chain; use `TARGET_LAG = DOWNSTREAM` upstream for cost efficiency.
---
### Q: Primary diagnostic view for Snowpipe errors in last 2 hours?
**A:** `INFORMATION_SCHEMA.COPY_HISTORY` (realtime, 14-day retention). For long-term, `ACCOUNT_USAGE.COPY_HISTORY` (365d, 90-min lag).
---

Total cards: **~110**. Group drill by domain, mix drill at the end, repeat until <2s per card.
