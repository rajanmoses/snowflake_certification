# Snowflake Advanced Data Engineer (DEA-C02) — Study Hub

Curated prep for Snowflake's **SnowPro Advanced: Data Engineer** certification. This folder contains deep Q&A per exam domain plus three companion artifacts (flashcards, mock exam, cheatsheet).

---

## Exam at a Glance

| Attribute        | Value                                                                 |
|------------------|-----------------------------------------------------------------------|
| Exam code        | **DEA-C02** (current version; supersedes DEA-C01)                     |
| Format           | Multiple choice + multiple select                                     |
| Questions        | **65**                                                                |
| Time limit       | **115 minutes** (~105s per question)                                  |
| Passing score    | **750 / 1000** (scaled — roughly 70% raw)                             |
| Cost             | **$375 USD** (voucher pricing varies)                                 |
| Prerequisites    | SnowPro Core (must be active)                                         |
| Delivery         | Online proctored (Pearson VUE / Kryterion) or test center             |
| Retake policy    | 14-day wait after failure; same voucher cost                          |
| Validity         | 2 years from pass date                                                |

---

## Exam Blueprint (Domain Weights)

| # | Domain                                | Weight  | Primary file                          |
|---|---------------------------------------|---------|---------------------------------------|
| 1 | **Data Movement**                     | ~27%    | `01_data_movement_QA.md`              |
| 2 | **Performance Optimization**          | ~22%    | `02_performance_optimization_QA.md`   |
| 3 | **Storage & Data Protection**         | ~18%    | `03_storage_and_data_protection_QA.md`|
| 4 | **Data Transformation**               | ~18%    | `05_data_transformation_QA.md`        |
| 5 | **Security**                          | ~15%    | `04_security_QA.md`                   |

Totals ~100%. Snowflake occasionally rebalances +/- 2 pts per domain between exam versions — treat weights as planning, not gospel.

---

## Module Index

| File | Focus |
|------|-------|
| `01_data_movement_QA.md` | COPY, Snowpipe, Streaming, Streams, Tasks, Iceberg, Kafka |
| `02_performance_optimization_QA.md` | Caches, pruning, clustering, SOS, MVs, Query Profile, sizing |
| `03_storage_and_data_protection_QA.md` | Micro-partitions, TT, Fail-safe, clones, replication, Hybrid |
| `04_security_QA.md` | RBAC, masking/RAP, OAuth/SAML/SCIM, Private Link, Access History |
| `05_data_transformation_QA.md` | UDFs, Snowpark, SPs, Dynamic Tables, Iceberg R/W, dbt, SCD2 |

Each Q&A file has **25 questions** structured as: *Answer → First Principles → ASCII Sketch → Lab Exercise → Gotchas / Exam Traps*. Count: **125 total scenario-grade questions**.

---

## Companion Artifacts

| File | Purpose |
|------|---------|
| `DEA-C02_FLASHCARDS.md` | ~100 rapid-fire Q/A cards. 15-min daily drills. |
| `DEA-C02_MOCK_EXAM.md` | 65-Q practice exam mirroring real distribution + difficulty. |
| `DEA-C02_CHEATSHEET.md` | One-page dense reference: credits, retention, syntax, SYSTEM$. |

---

## Recommended Study Order

### Phase 1 — Foundation (week 1)
Build the mental model before diving into edge cases.
1. `03_storage_and_data_protection_QA.md` **Q1–Q5** — micro-partition, Time Travel, Fail-safe, clone, table types.
2. `02_performance_optimization_QA.md` **Q1–Q2, Q7, Q25** — caches, pruning, warehouse sizing, credit math.
3. `04_security_QA.md` **Q1, Q7, Q13** — RBAC basics, network policy, ACCOUNTADMIN.
4. `01_data_movement_QA.md` **Q1, Q8, Q9, Q12** — COPY, errors, unload, metadata.

### Phase 2 — Core Domain (weeks 2–3)
Work the full Q&A file for each domain in blueprint-weight order.
1. **Data Movement** (highest %): all 25 Qs — focus on Snowpipe vs. Streaming, Streams, Tasks, Dynamic Tables, Iceberg.
2. **Performance Optimization**: all 25 Qs — clustering, SOS, MVs vs DTs, Query Profile, multi-cluster, QAS, join algorithms.
3. **Storage & Data Protection**: remaining Qs — replication, failover, retention edge cases, Hybrid Tables.
4. **Data Transformation**: all 25 Qs — Snowpark, UDFs, Snowpark-optimized WH, dbt, SCD2.
5. **Security**: all 25 Qs — masking, RAP, OAuth/SCIM, Access History, tag-based policies, external access.

### Phase 3 — Reinforcement (week 4)
- Flashcard sprints (`DEA-C02_FLASHCARDS.md`), 2x/day, 20 min each.
- Full mock exam (`DEA-C02_MOCK_EXAM.md`) under 115-min timer. Grade. Review wrong answers via the pointer-to-source links.
- Keep `DEA-C02_CHEATSHEET.md` taped above the desk for syntax recall.

### Phase 4 — Hands-On (ongoing)
Each Q&A file ships an executable lab block. Run them in a Snowflake trial account to build muscle memory for syntax, Query Profile reading, and cost telemetry views (ACCOUNT_USAGE, INFORMATION_SCHEMA).

---

## Lab Setup

Full step-by-step guide (free Snowflake account → VS Code → Git → daily loop): see [`lab_setup.md`](lab_setup.md).

---

## Study Tips Tailored to DEA-C02

**Scenario questions dominate.** Expect prompts like *"A data engineer needs to ingest JSON from S3 at <10s latency, with exactly-once semantics... which service?"* — eliminate wrong answers using blueprint matching:
- Latency < 10s + exactly-once → **Snowpipe Streaming**
- Latency ~1 min + at-least-once → **Snowpipe auto-ingest**
- Hourly bulk → **COPY INTO**
- Query-in-place → **External Table / Iceberg**

**Memorize crossovers:**
- 14-day vs 64-day load dedup (Snowpipe file history vs COPY LOAD_HISTORY).
- 1-day vs 90-day retention (Standard vs Enterprise+ Time Travel).
- 60-second minimum warehouse billing.
- 1-minute minimum serverless-task charge.
- Fail-safe is **always 7 days**, **permanent tables only**, **non-user-accessible**.

**Learn RBAC role chain cold.** The exam loves "which role should execute this?" questions. Default answer hierarchy:
- Create user/role → **USERADMIN** (via SECURITYADMIN)
- Grant privilege / manage grants → **SECURITYADMIN**
- Create warehouse / DB / schema → **SYSADMIN**
- Account-level config, replication, resource monitors → **ACCOUNTADMIN** (minimize)

**Query Profile red-flag table.** If a question includes a Query Profile hint, map it:
- Remote spill → warehouse too small (or Snowpark-optimized needed).
- 0% partitions pruned → missing filter, expression wrapping, or clustering key mismatch.
- Exploding join rows → missing predicate, cross-join, or wrong key.
- High "Synchronization %" → data skew.

**Cost math shows up as a word problem.** Know credit multipliers (XS=1, S=2, M=4, L=8, XL=16, 2XL=32...) and that resizing doesn't restart the 60s minimum.

**Replication tiers.**
- **Standard**: metadata, shares.
- **Enterprise**: same-cloud cross-region DB/account replication, materialized views, 90-day TT, Access History.
- **Business Critical**: cross-cloud replication, Tri-Secret Secure, PrivateLink, HIPAA/PCI.
- **VPS**: dedicated infrastructure.

**Snowpipe Streaming vs classic Snowpipe is the single most-tested pair.** Know every differentiator (latency, semantics, auth, cost model, failure modes).

**When unsure, pick the "Snowflake-native, serverless, declarative" option.** The exam strongly favors Dynamic Tables over Stream+Task when the SQL permits, tag-based policies over per-column policies, and serverless tasks over user-managed when workload is sporadic.

---

## Troubleshooting Your Prep

| Symptom | Fix |
|---------|-----|
| TT vs Fail-safe confusion | `03_storage..._QA.md` Q2–Q3; cheatsheet retention table |
| MV vs DT vs View choice | `02_performance..._QA.md` Q5 decision table |
| Pipe/stream states blur | `01_data_movement_QA.md` Q16 state transitions |
| Grants/future grants | `04_security_QA.md` Q19, Q25 end-to-end |
| Snowpark vs standard WH | `02_performance..._QA.md` Q20 triggers |

Good luck. Return to this hub for a weekly checkpoint; drive your weak domains to <10% error rate on the mock exam before scheduling the real one.
