# Snowflake Specialty — AI Governance & Security (GAS-C01)

Companion study pack: six domain-focused Q&A modules plus flashcards, mock exam, and cheatsheet.

---

## Certification at a Glance

| Attribute | Value |
|-----------|-------|
| Exam code | **GAS-C01** |
| Credential | SnowPro Specialty: AI Governance & Security |
| Format | Multiple-choice + multi-select |
| Approx. questions | 65 scored (plus unscored items) |
| Approx. duration | 115 minutes |
| Passing score | ~750 / 1000 (scaled) |
| Prerequisite | SnowPro Core (recommended) + hands-on with Snowflake governance features |
| Delivery | Pearson VUE / online proctored |
| Target audience | Governance leads, data stewards, security engineers, ML/AI platform owners |

The exam is governance- and compliance-flavored, not performance-flavored. Expect scenario questions where you must choose the right policy / role / audit view / framework. Vocabulary matters: NIST uses "confabulation," EU AI Act uses "GPAI," Snowflake uses "Horizon" — mix them accurately.

---

## Blueprint Domains & Approximate Weights

| # | Domain | Weight | Companion File |
|---|--------|--------|----------------|
| 1 | AI Governance Frameworks | ~15–20% | `01_ai_governance_frameworks_QA.md` |
| 2 | Data Governance for AI | ~20–25% | `02_data_governance_for_ai_QA.md` |
| 3 | Access Control & RBAC | ~15–20% | `03_access_control_and_rbac_QA.md` |
| 4 | Data Protection & Masking | ~20–25% | `04_data_protection_and_masking_QA.md` |
| 5 | Model Governance & Monitoring | ~15–20% | `05_model_governance_and_monitoring_QA.md` |
| 6 | Compliance & Audit | ~15–20% | `06_compliance_and_audit_QA.md` |

Weights are directional (Snowflake does not publish exact weights). Domain 2 (Data Governance for AI) and Domain 4 (Data Protection & Masking) carry the most scenario-heavy question volume.

---

## Module Summaries

### 01 — AI Governance Frameworks
Foundational conceptual domain. Covers NIST AI RMF 1.0 (Govern/Map/Measure/Manage), the NIST Generative AI Profile (NIST AI 600-1), EU AI Act risk tiers (unacceptable / high / limited / minimal) and GPAI obligations, ISO/IEC 42001 (AI Management System) vs. ISO/IEC 23894, OECD AI Principles, model lineage theory, fairness metrics (demographic parity vs. equal opportunity vs. equalized odds), model cards, AI value chain, Snowflake Horizon as the implementation substrate.

### 02 — Data Governance for AI
Largest domain. Deep dive into Snowflake Horizon: Catalog, auto-classification (privacy_category + semantic_category), tags vs. classifications, tag inheritance semantics, tag-based masking policies, data lineage (column-level + table-level) vs. OBJECT_DEPENDENCIES, ACCESS_HISTORY (BASE_OBJECTS_ACCESSED is the key column), POLICY_REFERENCES, business glossary, Cortex Analyst governance, Document AI, data residency enforcement.

### 03 — Access Control & RBAC
RBAC primitives (users, roles, privileges, grants), DAC overlay, system roles (ACCOUNTADMIN, SECURITYADMIN, USERADMIN, SYSADMIN, ORGADMIN, PUBLIC) and separation of duties, the **functional + access roles pattern** (users receive only functional roles; functional roles compose access roles), primary vs. secondary roles (`USE SECONDARY ROLES ALL` — `CURRENT_ROLE()` only returns primary; use `IS_ROLE_IN_SESSION()`), service account auth (key pair, OAuth, PAT), Secure Views, Secure UDFs, future grants, managed access schemas, APPLY privileges, network policies, authentication policies, session policies.

### 04 — Data Protection & Masking
Second-largest domain. Dynamic Data Masking (DDM), External Tokenization (reversible, external vault), Row Access Policies (RAP — one per table), Aggregation Policies (MIN_GROUP_SIZE for k-anonymity-style floors), Projection Policies (block SELECT of a column while allowing WHERE/JOIN), Differential Privacy policies (ε, δ, privacy unit column), synthetic data via Cortex, policy evaluation order, Tri-Secret Secure + customer-managed keys, clean rooms as a protection pattern.

### 05 — Model Governance & Monitoring
Cortex Guard (content safety categories: violent crimes, non-violent crimes, sex-related crimes, CSAM, defamation, specialized advice, privacy violations, IP violations, indiscriminate weapons/CBRN, hate, suicide/self-harm, sexual content), prompt injection defenses (layered: input sanitation, structured messages, RAG scanning, output validation, Cortex Guard), Model Registry governance (versioning, tags, aliases, access control), drift detection (data / concept / performance; PSI thresholds 0.1 / 0.25), Cortex AI Observability (event tables), LLM eval metrics (faithfulness, relevance, groundedness, PII leakage), deployment gates.

### 06 — Compliance & Audit
Shared responsibility model, Snowflake attestations (SOC 1/2, ISO 27001/27017/27018/27701, PCI DSS L1, HIPAA/HITRUST, FedRAMP Moderate/High, IRAP, StateRAMP, C5), ACCOUNT_USAGE view catalog (LOGIN_HISTORY, QUERY_HISTORY, ACCESS_HISTORY, SESSIONS, GRANTS_TO_*, POLICY_REFERENCES, TAG_REFERENCES, OBJECT_DEPENDENCIES, DATA_CLASSIFICATION_LATEST, DATA_TRANSFER_HISTORY, COPY_HISTORY), immutable audit log patterns, GDPR DSAR / Art. 17 right-to-erasure, HIPAA Technical Safeguards § 164.312 mapping, PCI DSS scoping for AI, FedRAMP considerations, legal holds.

---

## Recommended Study Order

The modules are numbered in dependency order. If you read them cold, follow 01 → 02 → 03 → 04 → 05 → 06. If you have time constraints:

**Fast path (3 days):**
1. Day 1: Modules 02 + 04 (the heaviest weighted domains)
2. Day 2: Modules 03 + 06 (RBAC + compliance)
3. Day 3: Modules 01 + 05 (frameworks + model governance) + mock exam

**Framework-first path (if you're new to AI governance):**
1. Start with 01 to build vocabulary (NIST RMF, EU AI Act, ISO 42001, OECD).
2. Move to 02 for the Snowflake implementation (Horizon, classification, lineage).
3. Then 04 for the protection toolbox (masking, RAP, aggregation, projection, DP).
4. Then 03 for RBAC (which policies reference).
5. Then 05 for the model-specific overlay.
6. Finish with 06 to tie everything to compliance evidence.

**Snowflake-practitioner path (if you know Snowflake, new to AI gov):**
1. Start with 05 (model governance is the newest layer).
2. Then 01 (frameworks give the vocabulary for the exam).
3. Then 02, 04, 06 (confirm you know these in the AI context).
4. Skim 03 (should already be familiar).

Each module ends with a "Gotchas Recap" or similar cheat sheet section — skim those before each study session.

---

## Companion Documents

| File | Purpose |
|------|---------|
| `GAS-C01_FLASHCARDS.md` | 100 flashcards organized by domain; front/back format. Use for daily review. |
| `GAS-C01_MOCK_EXAM.md` | 65 scenario questions with answer key, explanations, and module pointers. Simulates the real exam. |
| `GAS-C01_CHEATSHEET.md` | Dense single-document reference: frameworks, policies, ACCOUNT_USAGE views, compliance mapping, Cortex Guard, prompt-injection layers. |

---

## Study Tips for a Governance / Compliance-Flavored Exam

1. **Memorize the policy evaluation order** — row access → aggregation → projection → masking is almost certainly tested. (Masking runs *last* on surviving rows.)

2. **Know the ACCOUNT_USAGE view catalog cold.** Which view answers each question (ACCESS_HISTORY for who-read-what, POLICY_REFERENCES for what's bound, DATA_CLASSIFICATION_LATEST for what's classified). Latencies matter (ACCESS_HISTORY ~45 min, POLICY_REFERENCES ~2 h).

3. **BASE_OBJECTS_ACCESSED vs. DIRECT_OBJECTS_ACCESSED.** The exam will test the difference. BASE_OBJECTS_ACCESSED resolves transitively through views — this is the key insight for AI data auditing.

4. **TAG_REFERENCES vs. TAG_REFERENCES_WITH_LINEAGE.** Direct tags appear in the first; inherited tags only in the second (and only via the INFORMATION_SCHEMA table function).

5. **CURRENT_ROLE() vs. IS_ROLE_IN_SESSION().** Masking policies using `CURRENT_ROLE()` see only the **primary** role — a common bug trap. Use `IS_ROLE_IN_SESSION()` to check any active role (primary or secondary).

6. **Framework legal force.** NIST AI RMF = voluntary (US). EU AI Act = binding (EU, extraterritorial, up to €35M / 7% of global turnover). ISO 42001 = voluntary certifiable. GDPR = binding (EU, up to 4%). OECD = non-binding values.

7. **EU AI Act fine thresholds.** Prohibited practices: up to €35M or 7%. High-risk non-compliance: up to €15M or 3%. GPAI non-compliance: up to €15M or 3%. Grace period: 24 months for most high-risk (Aug 2026 enforcement); GPAI 12 months (Aug 2025).

8. **NIST AI RMF functions.** GOVERN is **cross-cutting**, not first. Expect a distractor saying "complete GOVERN before MAP."

9. **Confabulation, not hallucination.** NIST AI 600-1 uses "confabulation" as the formal term.

10. **12 Cortex Guard categories.** Memorize them — violent crimes, non-violent crimes, sex-related crimes, CSAM, defamation, specialized advice, privacy violations, IP violations, indiscriminate weapons (CBRN), hate speech, suicide/self-harm, sexual content.

11. **Policy composition facts.** One masking policy per column (direct OR tag-based, not both). One row access policy per table. One aggregation policy per table. One projection policy per column. These single-binding limits appear repeatedly.

12. **DP policy specifics.** Snowflake's DP policy uses `EPSILON`, optional `DELTA`, and `PRIVACY_UNIT_COLUMN` (the join key representing one subject). Lower epsilon → more noise → more privacy.

13. **Shared responsibility bite-size.** Snowflake runs the platform + certifies it. You classify data, write policy bodies, manage users, retain audit beyond 365 days, scope your workloads. "Snowflake is SOC 2" does not make your workload SOC 2.

14. **HIPAA + PCI editions.** Require **Business Critical** or **Virtual Private Snowflake**. HIPAA requires a signed BAA.

15. **Service account auth.** Prefer **key-pair** for scheduled jobs, **OAuth** (external IdP) for user-impersonating apps, **PAT (Programmatic Access Token)** for CI/CD short-lived tokens. Avoid passwords for services.

16. **Secure views for optimizer leakage.** `SECURE` disables leaky predicate pushdowns and hides definitions. Pay the small performance cost when restriction is the point.

17. **Model lineage vs. data lineage.** Model lineage includes code SHA, random seed, hyperparameters, framework version — things Horizon's native lineage does not capture. You instrument this via the Model Registry + custom tables.

18. **Aggregation policy returns `AGGREGATION_CONSTRAINT`**, not BOOLEAN. Projection policy returns `PROJECTION_CONSTRAINT`. Row access policy returns BOOLEAN. Masking policy returns the column's datatype.

19. **Trust Center requires enablement** + the `SNOWFLAKE.TRUST_CENTER_ADMIN` application role. Scanner packages include CIS Benchmarks, Snowflake Security Reference Architecture, and custom ones.

20. **ORGANIZATION_USAGE vs. ACCOUNT_USAGE.** Org = cross-account rollups (credits, replication, data transfer). Account = per-account activity (access, query, login). Cross-account AI governance needs replication of ACCOUNT_USAGE to a central governance account.

Good luck. The exam rewards precision over breadth — know your SQL for each policy type and your view for each audit question.
