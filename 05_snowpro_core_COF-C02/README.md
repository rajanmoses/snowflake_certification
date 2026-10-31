# SnowPro Core (COF-C02) — Study Pack Index

Welcome to the SnowPro Core certification study pack. This directory contains everything you need to go from zero to passing the current (COF-C02) exam, built around trivia-first mastery because that's what the exam actually rewards.

---

## 1. Exam at a Glance

| Attribute | Value |
|-----------|-------|
| **Exam code** | COF-C02 |
| **Credential** | SnowPro Core Certification |
| **Questions** | 100 (multiple choice + multiple select) |
| **Time limit** | 115 minutes |
| **Passing score** | **750 / 1000** (scaled score) |
| **Cost** | USD 175 (first attempt), USD 125 retake |
| **Delivery** | Online proctored or test center (Pearson VUE) |
| **Validity** | 2 years from pass date |
| **Prerequisite** | None (but 6+ months hands-on recommended) |
| **Languages** | English, Japanese |
| **Retake policy** | 7-day wait between attempts |

**Scoring is scaled, not raw.** You do not need to answer 75% correctly — the raw cut is in the 65–70% range depending on form. Multi-select questions require **ALL** correct options checked (no partial credit).

---

## 2. Domain Blueprint & Weights

| # | Domain | Weight | Companion file |
|---|--------|--------|----------------|
| 1 | Snowflake Data Cloud Features & Architecture | **~25%** | `01_snowflake_overview_architecture_QA.md` |
| 2 | Virtual Warehouses | **~20%** | `02_virtual_warehouses_QA.md` |
| 3 | Storage, Data Protection, and Data Sharing | **~10–15%** | `03_storage_and_protection_QA.md` |
| 4 | Data Movement (Load / Unload / Ingest) | **~20%** | `04_data_movement_QA.md` |
| 5 | Account Access and Security | **~15%** | `05_account_access_security_QA.md` |
| 6 | Performance Concepts | **~15%** | `06_performance_concepts_QA.md` |
| 7 | Data Transformations | **~15%** | `07_data_transformations_QA.md` |

Weights sum to more than 100% because sources vary slightly year-to-year. Treat Architecture, Virtual Warehouses, and Data Movement as the "big three."

---

## 3. Module Summaries

### Module 1 — Snowflake Overview & Architecture (~25%)
Three-layer architecture (Cloud Services, Compute/Virtual Warehouses, Centralized Storage) built on a **hybrid shared-disk / shared-nothing** model. Micro-partitions are ~16MB compressed, columnar, immutable. Four editions (Standard → Enterprise → Business Critical → VPS) with feature flags. Runs on AWS, Azure, GCP. Interfaces: Snowsight, SnowSQL, drivers, SQL REST API. Account identifier formats (org-based preferred). Marketplace and secure data sharing.

### Module 2 — Virtual Warehouses (~20%)
Sizes XS to 6XL, credits double each step (XS=1, 6XL=512). Vertical scale = resize; horizontal scale = multi-cluster (Enterprise+, max 10 clusters). Scaling policies: STANDARD (default, favors perf) vs ECONOMY (favors cost). AUTO_SUSPEND min 60s, per-second billing with 60s floor. Resource monitors (ACCOUNTADMIN) govern credit budgets. Local SSD cache per warehouse, lost on suspend.

### Module 3 — Storage & Protection (~10–15%)
Time Travel: 1 day Standard, up to 90 days Enterprise+. Fail-safe: fixed 7 days, Snowflake-only recovery, cannot be disabled. Three table types: Permanent (full TT + FS), Transient (max 1d TT, no FS), Temporary (session-scoped, max 1d TT, no FS). Zero-copy clone enabled by micro-partition immutability. UNDROP within TT window. Replication available Standard+; failover groups Business Critical+.

### Module 4 — Data Movement (~20%)
Four stage types: user (`@~`), table (`@%t`), named internal (`@s`), named external (points to S3/Blob/GCS). PUT/GET only via SnowSQL/drivers (not Snowsight). COPY INTO for bulk; Snowpipe for continuous/serverless; load history dedupes for **64 days**. ON_ERROR options: CONTINUE, SKIP_FILE, SKIP_FILE_N, SKIP_FILE_N%, ABORT_STATEMENT (default). Streams for CDC; Tasks for scheduling; Dynamic Tables replace much Stream+Task plumbing. External tables are read-only.

### Module 5 — Account Access & Security (~15%)
Default role hierarchy: ORGADMIN → ACCOUNTADMIN → (SECURITYADMIN → USERADMIN) and (SYSADMIN) → custom → PUBLIC. ACCOUNTADMIN owns resource monitors + account params. SECURITYADMIN manages grants globally; USERADMIN manages users/roles only. SYSADMIN owns data assets. Authentication: password+MFA (Duo), key-pair JWT, SAML2 SSO, OAuth. Network policies allow/block IPs. AES-256 at rest, TLS 1.2+ in transit. Tri-Secret Secure requires Business Critical+. Dynamic Data Masking + Row Access Policies = Enterprise+.

### Module 6 — Performance Concepts (~15%)
Four caches: Result Cache (24h TTL / 31d max, cross-session, Cloud Services), Metadata Cache (stats/COUNT/MIN/MAX), Warehouse Local SSD (per-cluster, lost on suspend), Remote Storage. Partition pruning via micro-partition metadata. Clustering keys for huge tables with filter-skewed access; Automatic Clustering is serverless. Materialized Views (Enterprise+). Search Optimization Service (Enterprise+, serverless) for point lookups. Query Acceleration Service (Enterprise+, serverless) for scan-heavy outliers.

### Module 7 — Data Transformations (~15%)
VARIANT/OBJECT/ARRAY for semi-structured; 16MB per VARIANT value. Access via `:` colon, `.` dot, `[]` bracket syntax — always cast with `::`. LATERAL FLATTEN explodes arrays into rows (OUTER=TRUE for LEFT JOIN semantics). UDFs (SQL/JS/Python/Java/Scala) for per-row logic; Stored Procedures (Snowflake Scripting + others) for imperative flows / DML. CALLER vs OWNER rights on sprocs. Views: regular, secure, materialized. QUALIFY filters window-function output.

---

## 4. Recommended Study Order

If you have 3–6 weeks:

1. **Week 1 — Architecture foundation.** Read Module 1 cover to cover. Memorize the 3-layer diagram, editions matrix, and account identifier formats.
2. **Week 2 — Compute + Storage.** Modules 2 and 3. Memorize the credit doubling table and the TT/Fail-safe timeline. These two together own ~35% of the exam.
3. **Week 3 — Data movement.** Module 4. Hands-on: build at least one COPY INTO pipeline and one Snowpipe pipe in a trial account.
4. **Week 4 — Security + Performance.** Modules 5 and 6 in parallel. Draw the role hierarchy from memory daily.
5. **Week 5 — Transformations + review.** Module 7, then re-read Modules 1–4.
6. **Week 6 — Flashcards + mock exam.** Drill `COF-C02_FLASHCARDS.md`, take `COF-C02_MOCK_EXAM.md` under timed conditions (115 min), review every miss against `COF-C02_CHEATSHEET.md`.

If you only have 1 week: skim all QA files, drill flashcards hard, take the mock twice, and camp on the cheatsheet.

---

## 5. Companion Files in This Directory

| File | Purpose |
|------|---------|
| `01_snowflake_overview_architecture_QA.md` | Module 1 Q&A (25 questions) |
| `02_virtual_warehouses_QA.md` | Module 2 Q&A (25 questions) |
| `03_storage_and_protection_QA.md` | Module 3 Q&A (25 questions) |
| `04_data_movement_QA.md` | Module 4 Q&A (25 questions) |
| `05_account_access_security_QA.md` | Module 5 Q&A (25 questions) |
| `06_performance_concepts_QA.md` | Module 6 Q&A (25 questions) |
| `07_data_transformations_QA.md` | Module 7 Q&A (25 questions) |
| **`COF-C02_FLASHCARDS.md`** | **100+ rapid-fire flashcards for trivia drill** |
| **`COF-C02_MOCK_EXAM.md`** | **100-question full-form mock with answer key** |
| **`COF-C02_CHEATSHEET.md`** | **Dense single-document reference for last-week review** |
| `README.md` | This file |

---

## 6. Tips for the Trivia-Heavy Exam

The SnowPro Core exam rewards memorization of specific numbers, defaults, and feature-to-edition mappings. Pure architectural intuition will not save you on these questions. Internalize these before test day.

### 6.1 Fixed numbers you MUST memorize

- **Micro-partition size**: ~16 MB compressed (50–500 MB uncompressed logical).
- **AUTO_SUSPEND minimum**: 60 seconds (via SQL).
- **Max clusters in a multi-cluster warehouse**: 10.
- **MAX_CONCURRENCY_LEVEL default**: 8.
- **STATEMENT_TIMEOUT_IN_SECONDS default**: 172800 (48 hours).
- **STATEMENT_QUEUED_TIMEOUT_IN_SECONDS default**: 0 (no limit).
- **Billing minimum per warehouse resume**: 60 seconds.
- **Result cache TTL**: 24 hours base, up to **31 days** max with hits.
- **Fail-safe**: fixed **7 days**, non-configurable.
- **Time Travel max**: 1 day (Standard), 90 days (Enterprise+).
- **Time Travel default**: 1 day for permanent tables.
- **COPY INTO load history**: 64 days.
- **VARIANT max per value**: 16 MB compressed.
- **Cloud services free tier**: 10% of daily compute spend.
- **Snowpipe target file size**: 100–250 MB compressed for optimal throughput.
- **Snowpark-optimized warehouse credit rate**: ~1.5x standard; minimum size Medium.

### 6.2 Credit doubling table (burn this in)

XS=1, S=2, M=4, L=8, XL=16, 2XL=32, 3XL=64, 4XL=128, 5XL=256, 6XL=512 credits/hour. Each step up doubles.

### 6.3 Edition feature matrix shortcuts

- **Standard**: Time Travel capped at 1 day; no multi-cluster; no MV; no DDM/RLS; no SOS/QAS.
- **Enterprise**: unlocks multi-cluster, Time Travel to 90d, materialized views, column/row security, SOS, QAS, multiple SAML integrations, periodic rekeying.
- **Business Critical**: adds HIPAA/PCI/HITRUST, Tri-Secret Secure (CMK), PrivateLink, database failover/failback.
- **VPS**: dedicated metadata store; full isolation.

### 6.4 Default role hierarchy quick-draw

ORGADMIN sits above account; ACCOUNTADMIN inherits SECURITYADMIN and SYSADMIN; SECURITYADMIN inherits USERADMIN; everyone has PUBLIC. SYSADMIN does NOT include SECURITYADMIN.

### 6.5 Exam strategy

- Multi-select questions tell you how many to pick ("TWO" / "THREE"). If they don't, assume strict and only pick what is unambiguously correct — partial credit does not exist.
- When two answers both look right, pick the one that is more specific or quotes a Snowflake-coined feature name.
- Trap words: "always", "never", "requires Standard edition" (usually wrong), "cannot be configured".
- Watch for Standard-vs-Enterprise feature traps on 10–15 questions.
- Metadata-only queries (SHOW, DESC, some COUNT) do NOT need a warehouse — this comes up repeatedly.
- Snowsight cannot run `PUT` or `GET` — these are client-side commands via SnowSQL/drivers.
- Fail-safe is NEVER user-queryable and NEVER configurable.
- Resource monitors can only be created by **ACCOUNTADMIN**.
- 115 min / 100 Qs = ~70 seconds per question. Flag anything that takes more than 2 minutes and move on; a reliable sweep is more valuable than wrestling with one question.

### 6.6 Final-week checklist

- Draw the three-layer architecture from memory.
- Recite the credit doubling table and max clusters (10).
- Recite the TT + Fail-safe timeline for permanent and transient tables.
- Recite the default role hierarchy.
- Explain Snowpipe vs COPY INTO in one minute.
- Know which 3 things are serverless (Snowpipe, Automatic Clustering, MV maintenance, SOS, QAS, replication, serverless tasks).
- Know the four stage types and their prefix symbols.
- Know the three VARIANT access syntaxes (`:`, `.`, `[]`) and the `::` cast.

Good luck. The exam is passable in 4–6 focused weeks if you drill the trivia and spend a few hours in a trial account doing hands-on work.
