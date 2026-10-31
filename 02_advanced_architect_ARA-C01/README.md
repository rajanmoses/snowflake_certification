# Snowflake Advanced Architect (ARA-C01) — Study Pack

## Certification Overview

| Field | Value |
|-------|-------|
| **Certification** | SnowPro Advanced: Architect |
| **Exam Code** | ARA-C01 |
| **Format** | ~65 multiple-choice / multi-select questions |
| **Duration** | 115 minutes |
| **Passing Score** | 750 / 1000 (scaled; roughly ~65-70% raw correct) |
| **Prerequisites** | SnowPro Core certification (must be active) |
| **Delivery** | Online proctored (Pearson VUE) or in-person test center |
| **Languages** | English, Japanese |
| **Cost** | USD $375 |
| **Validity** | 2 years (recertification required) |

## Exam Domain Blueprint

The ARA-C01 exam is scenario-dominant: expect multi-step situations where you must weigh tradeoffs across editions, security, performance, cost, and DR. Approximate weights (Snowflake publishes narrow ranges; architect-level questions often cross domains):

| # | Domain | Weight | Companion File |
|---|--------|--------|-----------------|
| 1 | Accounts, Security, Governance | 20-25% | `01_accounts_and_security_QA.md` |
| 2 | Snowflake Architecture (compute, storage, caching, concurrency) | 15-20% | `02_snowflake_architecture_QA.md` |
| 3 | Data Engineering (ingest, transform, external integrations) | 15-20% | `03_data_engineering_QA.md` |
| 4 | Performance Optimization | 15-20% | `04_performance_optimization_QA.md` |
| 5 | Data Sharing & Collaboration | 15-20% | `05_data_sharing_collaboration_QA.md` |
| 6 | Storage, Protection, DR | 10-15% | `06_storage_and_protection_QA.md` |

> Weights sum to ~100% at the midpoint; Snowflake does not publish exact per-exam percentages.

## Module Summaries

### 1. Accounts & Security (`01_accounts_and_security_QA.md`)
Organization and account structure, edition matrix (Standard / Enterprise / Business Critical / VPS), RBAC two-layer pattern (access roles vs functional roles), SCIM, SAML SSO, key-pair auth, OAuth (custom vs external), encryption hierarchy + Tri-Secret Secure, network policies, PrivateLink, session and authentication policies, Trust Center, ACCESS_HISTORY, multi-tenant SaaS patterns. **Decision anchor:** matching an architectural need (e.g., "no public internet", "customer-managed keys", "automated user lifecycle") to the right Snowflake primitive and edition.

### 2. Snowflake Architecture (`02_snowflake_architecture_QA.md`)
Three-layer model (Cloud Services, Compute, Storage), virtual warehouses (size vs cluster count), multi-cluster scaling (STANDARD vs ECONOMY), three cache layers (result / metadata / SSD), micro-partitions and pruning, snapshot isolation, cross-cloud / cross-region topology, Snowgrid, query lifecycle, Query Acceleration Service, Snowpark Container Services, stream types, multi-tenant isolation. **Decision anchor:** scale-up vs scale-out, workload-isolated warehouse topology, metadata-only operations.

### 3. Data Engineering (`03_data_engineering_QA.md`)
Ingest patterns (bulk COPY vs Snowpipe vs Snowpipe Streaming vs Kafka Connector), streams (standard / append-only / insert-only), tasks and DAGs (serverless vs user-managed), Dynamic Tables, Materialized Views, external tables / Iceberg / Hybrid Tables, external functions, stored procedures (Scripting / JS / Python / Scala), UDF types, MERGE patterns, stage types, error handling, table types. **Decision anchor:** picking the right precomputation / ingestion / transformation primitive for the freshness SLA.

### 4. Performance Optimization (`04_performance_optimization_QA.md`)
Natural vs manual clustering, Search Optimization Service, Materialized Views vs Dynamic Tables, Query Acceleration Service, query profile diagnosis, pruning failures and rewrites, scale-up vs scale-out testing, auto-suspend tradeoffs, data skew, multi-cluster concurrency, UDF cost model, secure views, VARIANT/JSON pruning, join order, performance review methodology, cache sizing by WH size, result cache invalidation. **Decision anchor:** symptom-to-lever mapping (spilling → scale up; queue → scale out; range filter → cluster; point lookup → SOS).

### 5. Data Sharing & Collaboration (`05_data_sharing_collaboration_QA.md`)
Secure Data Sharing (zero-copy), cross-region/cloud via replication + auto-fulfillment, Reader Accounts, Snowflake Marketplace, Private Listings, Data Clean Rooms, Native Apps, governance of shared PII, LISTING_CONSUMPTION_HISTORY, pricing models, Secure UDFs, federated analytics patterns, Horizon governance, consumer-side governance of imported data. **Decision anchor:** matching sharing distribution model (direct / listing / Marketplace / Reader / Native App / Clean Room) to audience and logic weight.

### 6. Storage & Protection (`06_storage_and_protection_QA.md`)
Micro-partition lifecycle (active → Time Travel → Fail-safe), AT/BEFORE syntax, zero-copy cloning, dev refresh patterns, database replication, failover groups vs replication groups, RPO/RTO design, DR architectures, storage cost optimization, permanent/transient/temporary tables, UNDROP, clone + TT interaction, cross-cloud vs cross-region DR. **Decision anchor:** matching recovery requirement (RPO/RTO) to Snowflake replication primitives, picking table type for cost vs durability.

## Recommended Study Order

1. **Start with `02_snowflake_architecture_QA.md`** — foundations the other domains assume.
2. **Then `06_storage_and_protection_QA.md`** — micro-partition / Time Travel / clone concepts underpin performance and DR.
3. **Then `01_accounts_and_security_QA.md`** — edition matrix governs what's available in every other domain.
4. **Then `03_data_engineering_QA.md`** — ingest/transform patterns reference storage and architecture.
5. **Then `04_performance_optimization_QA.md`** — ties everything together with tuning levers.
6. **Finally `05_data_sharing_collaboration_QA.md`** — most dependent on all other domains (edition, security, storage, replication).

After the first pass:
- Run `ARA-C01_FLASHCARDS.md` daily (spaced repetition).
- Hit `ARA-C01_CHEATSHEET.md` for last-week consolidation.
- Take `ARA-C01_MOCK_EXAM.md` under timed conditions ~1 week out, then again 2-3 days before.

## Companion Files in This Folder

| File | Purpose |
|------|---------|
| `01_accounts_and_security_QA.md` | Domain 1 Q&A deep dive |
| `02_snowflake_architecture_QA.md` | Domain 2 Q&A deep dive |
| `03_data_engineering_QA.md` | Domain 3 Q&A deep dive |
| `04_performance_optimization_QA.md` | Domain 4 Q&A deep dive |
| `05_data_sharing_collaboration_QA.md` | Domain 5 Q&A deep dive |
| `06_storage_and_protection_QA.md` | Domain 6 Q&A deep dive |
| `ARA-C01_FLASHCARDS.md` | ~100 Q/A flashcards for spaced repetition |
| `ARA-C01_MOCK_EXAM.md` | 65-question scenario-heavy practice exam |
| `ARA-C01_CHEATSHEET.md` | Dense single-page reference tables |

## Study Tips for Architect-Level Scenario Questions

ARA-C01 is NOT the Core exam. The questions rarely ask "what does X do?" — they ask "given these constraints, which combination of primitives satisfies them?" Prepare accordingly:

1. **Memorize the Edition Matrix cold.** Half the trick questions pivot on "requires Business Critical" (Tri-Secret Secure, PrivateLink, failover groups, Database Replication for failover) or "requires Enterprise" (multi-cluster warehouses, Time Travel >1 day, materialized views, dynamic data masking, SOS, ACCESS_HISTORY). See cheatsheet.

2. **Learn the "when to use X vs Y" decision tables** — not individual feature definitions. The exam asks you to pick between options, not describe one. Key dyads:
   - Materialized View vs Dynamic Table vs Streams+Tasks
   - Clustering vs Search Optimization
   - Scale UP vs scale OUT
   - Database Replication vs Failover Group vs Replication Group
   - Direct Share vs Reader Account vs Marketplace vs Private Listing vs Clean Room vs Native App
   - Snowpipe vs Snowpipe Streaming vs COPY INTO
   - Custom OAuth vs External OAuth
   - Secure View vs Masking Policy vs Row Access Policy vs External Tokenization

3. **Watch for compound requirements.** A scenario mentioning "PrivateLink + Tri-Secret + partner sharing" implies BC edition + Tri-Secret setup + Secure View + Share. Multi-step scenarios reward composing primitives correctly.

4. **Understand failure modes, not just happy paths.** Many questions describe something broken (query not pruning, replication falling behind, consumer complaining about latency) — you must diagnose the specific cause and pick the specific fix. Memorize the common gotchas listed at the end of each Q&A section.

5. **RBAC traps are everywhere.** Expect questions about:
   - ORGADMIN scope (one account, cross-account governance, no data access)
   - Access roles vs functional roles two-layer pattern
   - OWNERSHIP transfer vs regular grants
   - FUTURE grants (schema overrides database)
   - `IMPORTED PRIVILEGES` is the only grant verb on share databases
   - MANAGED ACCESS schemas (owner can't grant)

6. **Know what's shareable and what isn't.** Shareable: tables, external tables, secure views, secure UDFs, Iceberg (limited). NOT shareable: streams, tasks, pipes, sequences, stored procedures (directly). Workaround via derived tables + secure views.

7. **Know the storage lifecycle numbers.**
   - Time Travel: 0-1 day (Standard, transient, temp), 0-90 day (Enterprise+ permanent)
   - Fail-safe: 7 days (permanent only), always, non-configurable
   - Snowpipe file dedupe window: 64 days
   - Cloud Services billing threshold: 10% of daily compute
   - Minimum auto-suspend: 60 seconds
   - Minimum billing granularity: 60 seconds then per-second
   - OAuth refresh token max validity: 90 days
   - Default MAX_CONCURRENCY_LEVEL per cluster: 8

8. **Think in cost deltas.** Many questions have multiple "correct" answers; the exam wants the most cost-efficient. Example: "scale up to 2XL for 20 min" and "scale out to 10 clusters at M" may both work; one costs more.

9. **Practice query-profile reading.** Memorize the signals: `partitionsScanned / partitionsTotal` ratio, `bytes_spilled_to_remote_storage` (bad), `queued_overload_time` vs `queued_provisioning_time`, exploding joins, UDF overhead.

10. **Time management.** 65 questions in 115 min = ~1 min 46 sec per question. Scenario questions are often 200-300 words — read fast, re-read the question stem, eliminate obviously-wrong options, commit. Flag only questions where multiple options look defensible; return at the end.

11. **Multi-select discipline.** When a question says "choose TWO" or "choose THREE," the exam gives partial credit only if ALL correct options are selected. One wrong click wipes the point. Resist the urge to add a plausible-looking fourth option.

12. **Read the question before the options.** Scenario stems often contain the specific constraint that eliminates 2 of 4 options (edition, region, latency, cost). Highlight that constraint mentally first.

## Common Exam Patterns To Recognize

- **"Customer-managed keys"** → Tri-Secret Secure → Business Critical.
- **"No public internet"** → PrivateLink → Business Critical.
- **"Automatic user lifecycle"** → SCIM.
- **"Service account that we rotate every 90 days"** → key-pair with `RSA_PUBLIC_KEY_2`.
- **"Partner has no Snowflake account"** → Reader Account.
- **"Partner has Snowflake, we want self-service"** → Private Listing with auto-fulfill.
- **"Partner data, need joint analytics without exposing rows"** → Clean Room.
- **"RPO 1 min, promotable"** → Failover Group, 1-minute schedule, BC on both.
- **"Point lookup on high-cardinality column"** → SOS.
- **"Range filter on date"** → Clustering by date.
- **"Dashboard refreshes every 60s with 50 widgets"** → Dynamic Table + multi-cluster + result cache.
- **"Query scanned full table despite WHERE on date"** → function on column, rewrite as range.
- **"Session isolation across users"** → authentication policy or session policy per user.
- **"Data must stay in open format"** → Snowflake-managed Iceberg.
- **"OLTP + analytics on same table"** → Hybrid Table.

Good luck.
