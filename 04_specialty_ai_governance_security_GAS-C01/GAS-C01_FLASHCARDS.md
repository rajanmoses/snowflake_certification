# GAS-C01 Flashcards

100 flashcards, front/back format. Organized loosely by domain but front-loaded with high-frequency exam topics (NIST AI RMF, EU AI Act, policy evaluation order, ACCESS_HISTORY columns, Cortex Guard, aggregation policies, DP).

Format:
```
### N. [Topic] — Front
> Back
```

---

## Front-loaded: High-frequency exam topics

### 1. NIST AI RMF — the four functions
**Front:** Name and briefly describe the four functions of NIST AI RMF 1.0.
> **GOVERN** — cross-cutting culture, policy, roles, accountability. Always on.
> **MAP** — establish context for a specific AI system (purpose, stakeholders, risks).
> **MEASURE** — quantitatively assess risk/performance (bias, drift, safety).
> **MANAGE** — respond to risks (prioritize, treat, monitor, incident response).
> GOVERN is cross-cutting; MAP → MEASURE → MANAGE iterate on each system.

### 2. NIST AI RMF — Govern function trap
**Front:** True or false: "Complete GOVERN before you can begin MAP."
> **False.** GOVERN is cross-cutting — it operates continuously, not as a sequential first step. MAP, MEASURE, MANAGE run on specific AI systems with GOVERN as the containing structure.

### 3. EU AI Act — the four risk tiers
**Front:** List the four EU AI Act risk tiers and give one example each.
> **Unacceptable** — banned (social scoring, real-time biometric ID in public).
> **High** — strict obligations Art. 8–15 (healthcare, hiring, credit, education, law enforcement, critical infra).
> **Limited** — transparency only Art. 50 (chatbots must disclose, deepfakes must be labeled).
> **Minimal** — no obligations (spam filters, game AI).

### 4. EU AI Act — GPAI vs. GPAI with systemic risk
**Front:** What distinguishes a "GPAI model" from a "GPAI model with systemic risk"?
> A **GPAI model** (Art. 3(63)) is any model trained at scale showing significant generality. A **GPAI with systemic risk** (Art. 51) additionally crosses the 10^25 FLOPs training compute threshold OR is designated by the Commission. The latter has stricter obligations: additional model evaluations, incident reporting, cybersecurity measures.

### 5. EU AI Act — maximum fine for prohibited practices
**Front:** What is the EU AI Act maximum fine for prohibited practices?
> Up to **€35 million or 7% of global annual turnover**, whichever is higher. (Higher than GDPR's 4%.)

### 6. EU AI Act — grace periods
**Front:** When do high-risk and GPAI obligations enter into force?
> **GPAI** obligations: ~12 months after entry into force (approximately Aug 2025). **High-risk** obligations: ~24 months (approximately Aug 2026). Prohibited practices: 6 months (Feb 2025).

### 7. Policy evaluation order
**Front:** In what order are Snowflake data policies evaluated for a SELECT query?
> **1. Secure view** (hide definition) → **2. Row Access Policy** (filter rows) → **3. Aggregation Policy** (enforce min group size) → **4. Projection Policy** (block disallowed columns) → **5. Masking Policy** (transform surviving values on surviving rows).

### 8. Policy composition — single-binding limits
**Front:** How many of each policy can be bound to a single column/table?
> **1 masking policy per column** (direct OR tag-based, not both). **1 row access policy per table**. **1 aggregation policy per table**. **1 projection policy per column**. **1 privacy (DP) policy per table**.

### 9. ACCESS_HISTORY — the four object columns
**Front:** Name the four ACCESS_HISTORY columns describing objects touched by a query.
> **DIRECT_OBJECTS_ACCESSED** — objects named explicitly (e.g., a view).
> **BASE_OBJECTS_ACCESSED** — underlying tables resolved transitively through views (critical for AI auditing).
> **OBJECTS_MODIFIED** — targets written (CTAS, INSERT target).
> **OBJECT_MODIFIED_BY_DDL** — objects affected by DDL (ALTER TABLE etc.).

### 10. ACCESS_HISTORY — latency and retention
**Front:** What's the ACCESS_HISTORY latency and retention in ACCOUNT_USAGE?
> **~45 minutes** latency, **365 days** retention. For near-real-time use INFORMATION_SCHEMA or event tables. Longer retention requires custom export.

### 11. POLICIES_REFERENCED column
**Front:** What does the `POLICIES_REFERENCED` column of ACCESS_HISTORY tell an auditor?
> It lists every policy evaluated for that query (masking, row-access, agg, projection) and whether it was effective. This is the forensic proof that governance *actually ran* on a particular query — distinct from POLICY_REFERENCES (static binding record).

### 12. Cortex Guard — the 12 categories
**Front:** List the 12 content safety categories Cortex Guard detects.
> 1. Violent crimes
> 2. Non-violent crimes
> 3. Sex-related crimes
> 4. Child sexual exploitation (CSAM)
> 5. Defamation
> 6. Specialized advice (medical/legal/financial)
> 7. Privacy violations (PII leakage)
> 8. Intellectual property violations
> 9. Indiscriminate weapons (CBRN)
> 10. Hate speech
> 11. Suicide and self-harm
> 12. Sexual content

### 13. Cortex Guard — invocation
**Front:** How do you enable Cortex Guard on a `COMPLETE` call?
> Pass `{'guardrails': TRUE}` in the options parameter: `SNOWFLAKE.CORTEX.COMPLETE(model, messages, {'guardrails': TRUE})`. Returns metadata on triggers; unsafe content is blocked.

### 14. Aggregation policy — key parameter
**Front:** What's the main parameter of an aggregation policy, and what does it enforce?
> **MIN_GROUP_SIZE** (integer). It enforces that query results only return groups where `COUNT(*) >= MIN_GROUP_SIZE`. Direct `SELECT *` is blocked; only aggregated queries succeed. Rows in undersized groups are redacted (NULL sentinel) rather than dropped.

### 15. Aggregation policy — return type
**Front:** What type does an aggregation policy body return?
> `AGGREGATION_CONSTRAINT` — e.g., `AGGREGATION_CONSTRAINT(MIN_GROUP_SIZE => 50)` or `NO_AGGREGATION_CONSTRAINT()` to exempt. **Not** BOOLEAN.

### 16. Differential Privacy — epsilon basics
**Front:** In Snowflake's DP policy, what does lower epsilon mean?
> **Lower ε = more privacy, more noise, less utility**. ε is the privacy budget. Each query consumes some budget; a running total is tracked. The DP policy also accepts `DELTA` (small failure probability) and `PRIVACY_UNIT_COLUMN` (the subject-identifying key).

### 17. DP vs. aggregation policy
**Front:** Is a Snowflake aggregation policy differential privacy?
> **No.** Aggregation policies implement a k-anonymity-style floor (min group size). They can leak via composition attacks across repeated queries. DP (ε, δ) provides formal mathematical guarantees against inclusion-of-individual inference. Different tools for different threats.

### 18. CURRENT_ROLE() in masking policy — the gotcha
**Front:** A masking policy uses `CURRENT_ROLE() = 'PII_READER'`. The user has `PII_READER` as a secondary role. What does the policy see?
> It returns the user's **primary** role, which is not PII_READER. The policy evaluates to the masked branch. **Fix:** use `IS_ROLE_IN_SESSION('PII_READER')` which checks every active role (primary + secondary).

### 19. USE SECONDARY ROLES ALL
**Front:** What does `USE SECONDARY ROLES ALL` do, and what still uses only the primary role?
> It activates every granted role for *access evaluation* — SELECT/USAGE succeed if any role has the privilege. **Object creation (ownership) still uses the primary role.** `CURRENT_ROLE()` returns primary; `CURRENT_SECONDARY_ROLES()` returns the rest.

### 20. ISO 42001 vs. NIST AI RMF
**Front:** What is the key practical difference between ISO 42001 and NIST AI RMF?
> **ISO 42001** is a **certifiable standard** (AIMS) with ten clauses + Annex A (38 controls). You can be audited and awarded a certificate. **NIST AI RMF** is a voluntary framework of guidance with four functions; no certification body. Many orgs use ISO 42001 structure with NIST as the risk methodology.

---

## Domain 1 — AI Governance Frameworks

### 21. NIST AI 600-1 — formal name of hallucination
**Front:** What term does NIST AI 600-1 use for "hallucination"?
> **Confabulation.** NIST uses "confabulation" as the formal label for plausible-sounding false outputs. The GenAI Profile adds 12 risk categories including confabulation, CBRN info uplift, info integrity, info security, dangerous content.

### 22. OECD AI Principles — count and purpose
**Front:** How many OECD AI Principles are there, and what role do they play in other frameworks?
> **Five** value-based principles (inclusive growth, human rights, transparency, robustness, accountability) plus five government recommendations. They're the **conceptual root** cited by NIST AI RMF, EU AI Act, and ISO 42001 — a common cross-jurisdiction vocabulary.

### 23. Model lineage vs. data lineage
**Front:** What does model lineage include that classical data lineage does not?
> Code SHA, random seed, hyperparameters, framework/library versions, hardware, prompts/system instructions for LLMs, and output→input feedback edges (RLHF, self-training). Snowflake Horizon lineage covers only the *data* edges; the code/config edges require the Model Registry + custom tables.

### 24. Fairness — demographic parity vs. equal opportunity
**Front:** Define demographic parity and equal opportunity in one line each.
> **Demographic parity:** selection rates equal across groups — `P(ŷ=1 | A=a) = P(ŷ=1 | A=b)`.
> **Equal opportunity:** TPR equal across groups among qualified individuals — `P(ŷ=1 | Y=1, A=a) = P(ŷ=1 | Y=1, A=b)`.

### 25. Chouldechova impossibility
**Front:** State the impossibility theorem for fairness.
> When base rates differ across groups, you **cannot simultaneously** satisfy demographic parity, equalized odds, and calibration. Trade-offs are mandatory. Snowflake measures all three and leaves the choice to the deployer.

### 26. 4/5ths rule
**Front:** What is the "4/5ths rule" and what body publishes it?
> US EEOC employment-law heuristic: if the selection-rate ratio of the lowest-selected group to the highest is **< 0.8**, it's considered evidence of adverse impact. A *trigger* for further analysis, not a definitive legal verdict.

### 27. Model card contents
**Front:** List the 8 standard sections of a model card.
> 1. Model details  2. Intended use  3. Factors (demographic groups)  4. Metrics (overall + per factor)  5. Training data  6. Evaluation data  7. Ethical considerations  8. Caveats and recommendations. (Mitchell et al. 2019.)

### 28. EU AI Act — key high-risk obligations
**Front:** Name three major EU AI Act obligations for a high-risk AI system.
> **Art. 9** Risk management system (continuous, documented).
> **Art. 10** Data governance (training/validation data representative, provenance documented).
> **Art. 14** Human oversight (ability to intervene, override, stop).
> Also: Art. 11 technical documentation, Art. 12 logging, Art. 13 transparency, Art. 15 accuracy/robustness, conformity assessment, post-market monitoring.

### 29. AI incident — NIST definition
**Front:** How does NIST define an "AI incident"?
> An event where AI development, use, or malfunction directly or indirectly leads to actual or potential **harm**. Snowflake incident response leverages ACCESS_HISTORY (forensic), Cortex AI Observability (prompt/response), Time Travel (recovery), Model Registry versions (rollback), and tag-driven kill switches.

### 30. AI value chain — EU Art. 25
**Front:** Who bears liability when a model is fine-tuned, under EU AI Act Art. 25?
> The party that **"substantially modifies"** a model generally takes on provider obligations. Fine-tuning typically qualifies. Downstream deployers must obtain and respect documentation from upstream providers.

---

## Domain 2 — Data Governance for AI

### 31. Horizon — what it includes
**Front:** Name five Horizon capabilities.
> **Catalog**, **Classification**, **Tagging**, **Lineage**, **Access History**, **Policies** (masking/RAP/agg/projection), **Trust Center**, **Universal Search / Copilot**, **Internal Marketplace / Data Products**. Horizon is a product bundle, not a single feature.

### 32. Classification — two axes
**Front:** Snowflake auto-classification returns two orthogonal labels. What are they?
> **privacy_category** — regulatory sensitivity: IDENTIFIER, QUASI_IDENTIFIER, SENSITIVE, INSENSITIVE.
> **semantic_category** — type of data: NAME, EMAIL, PHONE, SSN, IP_ADDRESS, URL, DATE_OF_BIRTH, AGE, GENDER, LAT/LONG, CREDIT_CARD, IBAN, VIN, etc.
> Also returns a `probability` score.

### 33. Classification invocation
**Front:** What SQL triggers classification, and what parameter writes tags automatically?
> `CALL SYSTEM$CLASSIFY('db.schema.table', {'auto_tag': TRUE})`. Auto-tag writes system tags `SNOWFLAKE.CORE.PRIVACY_CATEGORY` and `SNOWFLAKE.CORE.SEMANTIC_CATEGORY` on classified columns.

### 34. Schema-level auto-classification
**Front:** How do you enable continuous auto-classification on a schema?
> `ALTER SCHEMA db.sch SET AUTO_CLASSIFY = TRUE`. Runs on new tables, new columns, and every ~30 days. Requires `IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE`.

### 35. Tag inheritance semantics
**Front:** Along which axis do Snowflake tags inherit?
> **ACCOUNT → DATABASE → SCHEMA → TABLE/VIEW → COLUMN** (object hierarchy). A table inherits its schema's tags. **Column-level tags do NOT inherit from table-level tags in the same namespace** — column tags are independent.

### 36. TAG_REFERENCES vs. TAG_REFERENCES_WITH_LINEAGE
**Front:** You tagged a schema but not its tables. Where do you see the inherited tag?
> **TAG_REFERENCES** shows only direct tags — the table will not appear. **TAG_REFERENCES_WITH_LINEAGE** (an INFORMATION_SCHEMA table function) shows inherited tags with a `LEVEL` indicator (`SCHEMA`, `DATABASE`, etc.).

### 37. Tag-based masking policy syntax
**Front:** How do you bind a masking policy via a tag?
> `ALTER TAG <tag_name> SET MASKING POLICY <policy_name> FOR (<data_type>)`. Every column with that tag of any value automatically gets the policy. One masking policy per tag per data type.

### 38. Classification vs. tagging
**Front:** What is the governance workflow between classification and tagging?
> **Classify first** (content-based inference) → **apply custom tags** (declared policy metadata) → **bind policies to tags** (automatic coverage). Classification = empirical "what is this?"; tagging = policy "what should we do about it?"

### 39. OBJECT_DEPENDENCIES vs. GET_LINEAGE
**Front:** Which is correct for AI training-data audits: OBJECT_DEPENDENCIES or GET_LINEAGE?
> **GET_LINEAGE.** OBJECT_DEPENDENCIES captures structural (DDL-time) relationships — views depending on tables, etc. GET_LINEAGE captures runtime DML flows — CTAS, INSERT…SELECT, MERGE — which is how training sets are materialized.

### 40. Column-level lineage
**Front:** How do you request column-level lineage from GET_LINEAGE?
> Pass `OBJECT_CONSTRUCT('column_name', 'col_x')` as the final argument:
> `SELECT * FROM TABLE(SNOWFLAKE.CORE.GET_LINEAGE('db.sch.tbl', 'TABLE', 'UPSTREAM', 1, 5, OBJECT_CONSTRUCT('column_name','customer_segment')))`.

### 41. Business Glossary
**Front:** What's the AI-governance value of a Horizon Business Glossary?
> Prevents semantic drift — "revenue (gross)" vs. "revenue (net)" surfaces explicitly. Features in models link back to terms, which link to physical columns. Also feeds Cortex Analyst for natural-language queries.

### 42. Cortex Analyst governance
**Front:** When Cortex Analyst generates SQL from a natural-language prompt, under whose role does it run?
> **The caller's role.** Masking, row-access, aggregation, projection policies all apply as normal. The surface-area risk is the **semantic model (YAML)** — if it exposes a column, Analyst will reference it. Treat the YAML as a governance artifact.

### 43. ACCOUNT_USAGE vs. ORGANIZATION_USAGE
**Front:** Where do you look for cross-account credit usage and data transfer activity?
> **ORGANIZATION_USAGE** (views like CREDIT_USAGE, DATA_TRANSFER_HISTORY, REPLICATION_USAGE). Access/query/login history and policy references are per-account (ACCOUNT_USAGE); for cross-account governance, replicate ACCOUNT_USAGE to a central governance account.

### 44. Semantic categories list — identifiers
**Front:** Give 6 Snowflake semantic categories that are typically IDENTIFIERS.
> NAME, EMAIL, PHONE, SSN, CREDIT_CARD, IBAN. (Also IP_ADDRESS, VIN, MAC_ADDRESS context-dependent.)

### 45. Data metric functions
**Front:** What is a DMF and where do results land?
> Data Metric Functions compute scalar quality signals on columns/tables (NULL_COUNT, UNIQUE_COUNT, DUPLICATE_COUNT, custom). Results go to `SNOWFLAKE.LOCAL.DATA_QUALITY_MONITORING_RESULTS` and the ACCOUNT_USAGE equivalent. Schedule set via `DATA_METRIC_SCHEDULE` on the table.

### 46. Manual classification override
**Front:** How do you prevent auto-classification from overwriting a human-corrected label?
> Set the system tag directly (`SNOWFLAKE.CORE.SEMANTIC_CATEGORY`, `PRIVACY_CATEGORY`) with the corrected value, and run classification with `overwrite_existing_tags => FALSE`, or tag the column `SNOWFLAKE.CORE.EXCLUDE_FROM_CLASSIFICATION='TRUE'`.

---

## Domain 3 — Access Control & RBAC

### 47. Snowflake RBAC — core concepts
**Front:** Name the five core RBAC concepts in Snowflake.
> **User**, **Role**, **Privilege**, **Securable object**, **Grant**. Plus the DAC overlay: object OWNER can grant further at discretion.

### 48. Functional + Access roles pattern
**Front:** Describe the Snowflake-recommended two-layer role pattern.
> **Access roles** hold privileges on objects (one per object/bundle). **Functional roles** represent personas and compose access roles. Users receive **only functional roles**. New objects → add to access role → every functional role that includes it inherits.

### 49. System roles — don't do this
**Front:** What is the single most common Snowflake governance failure?
> Granting ACCOUNTADMIN broadly, especially to service accounts. ACCOUNTADMIN should be break-glass only with alerting on its use.

### 50. ORGADMIN
**Front:** What is ORGADMIN responsible for?
> Organization-level operations: creating accounts, managing replication groups across accounts, setting org-level parameters. Lives at a higher scope than ACCOUNTADMIN.

### 51. Primary vs. secondary role — ownership
**Front:** Under `USE SECONDARY ROLES ALL`, which role owns objects created in the session?
> The **primary** role. Access evaluation uses the union; ownership uses only primary. `CURRENT_ROLE()` returns primary.

### 52. Secure view differences
**Front:** Name two things a SECURE view does differently from a regular view.
> 1. **Definition is hidden** from non-owners via `GET_DDL` / `INFORMATION_SCHEMA.VIEWS`.
> 2. **Optimizer disables leaky pushdowns** that could expose predicate information through query plans / row-count side channels.

### 53. Service account auth — three methods
**Front:** Name the three Snowflake-recommended service-account auth methods and when each applies.
> **Key pair** — scheduled batch jobs, server-to-server (private key + public key registered on user).
> **OAuth** (external IdP, e.g., Okta, Azure AD) — user-impersonating apps.
> **Programmatic Access Token (PAT)** — short-lived tokens for CI/CD, migration tools.
> Passwords should not be used for services.

### 54. Future grants
**Front:** What are FUTURE grants, and at what scopes?
> Grants that apply to objects **not yet created**. Scopes: **database** (`GRANT … ON FUTURE TABLES IN DATABASE`) and **schema**. If you want new schemas to also inherit, grant at the database level too.

### 55. Managed access schemas
**Front:** What does `WITH MANAGED ACCESS` on a schema change?
> Only the **schema owner** (not individual object owners) can grant access on objects within the schema. Centralizes DAC — prevents object owners from granting to peers unilaterally.

### 56. APPLY privilege
**Front:** What does the APPLY privilege on a tag/policy allow?
> It lets a role attach the tag or policy to objects — distinct from USAGE on the tag/policy. Separation enables "security team authors policy, governance team applies it" workflows.

### 57. Network policy
**Front:** What does a NETWORK POLICY restrict?
> Allowed/blocked IP ranges for sessions. Can be assigned at account or user level. For AI workloads, pin service accounts to narrow CIDR ranges from your CI/CD or orchestrator.

### 58. Authentication policy
**Front:** What does an AUTHENTICATION POLICY control?
> Allowed auth methods (`PASSWORD`, `SAML`, `OAUTH`, `KEYPAIR`, `PROGRAMMATIC_ACCESS_TOKEN`), MFA requirements, allowed clients. Set at account or user level.

### 59. Session policy
**Front:** What does a SESSION POLICY control?
> Session timeout parameters: `SESSION_IDLE_TIMEOUT_MINS`, `SESSION_UI_IDLE_TIMEOUT_MINS`. Supports HIPAA § 164.312(a)(2)(iii) automatic logoff.

---

## Domain 4 — Data Protection & Masking

### 60. Dynamic Data Masking — what "dynamic" means
**Front:** What makes DDM "dynamic"?
> Policy evaluates at query time on the live data — no masked copies, no re-materialization. Policy changes take effect for next query immediately. Different users see different values from the same stored row.

### 61. External Tokenization — when to prefer
**Front:** Give two scenarios where External Tokenization beats DDM.
> 1. Reversibility: authorized users retrieve original via the vault's detokenize.
> 2. Removing sensitive data from Snowflake storage (only tokens stored; raw lives in external vault — may reduce PCI scope).
> 3. Format-Preserving Encryption needed for cross-system matching.

### 62. Row Access Policy — composition with masking
**Front:** Which runs first: Row Access Policy or Masking Policy?
> **Row access runs first.** Rows filtered out by RAP are never evaluated for masking. Both can share tags for consistent gating.

### 63. Projection policy — what it blocks
**Front:** A projection policy blocks projection but allows what?
> It blocks `SELECT col` (projection into result set). It **allows** use in WHERE, JOIN, GROUP BY (predicates/joins/groupings). "Use is fine, disclosure is not."

### 64. Projection constraint return
**Front:** What does a projection policy body return?
> `PROJECTION_CONSTRAINT` — e.g., `PROJECTION_CONSTRAINT(ALLOW => TRUE)` or `PROJECTION_CONSTRAINT(ALLOW => FALSE)`. **Not** BOOLEAN.

### 65. Aggregation policy — allowed-use case
**Front:** With `MIN_GROUP_SIZE => 50`, which query succeeds?
> `SELECT state, COUNT(*) FROM customers GROUP BY state` — provided each state group has ≥ 50 rows. Smaller groups are redacted (nulled out). `SELECT * FROM customers` is blocked.

### 66. Privacy policy — DP parameters
**Front:** What are the three primary parameters of Snowflake's DP policy?
> `EPSILON` (privacy budget — lower = more private), optional `DELTA` (small failure probability, e.g., 1e-6), `PRIVACY_UNIT_COLUMN` (the join key representing a single subject).

### 67. Privacy budget tracking
**Front:** What happens when the DP privacy budget is exhausted?
> Subsequent queries either receive more noise (larger scale) or are blocked. Snowflake tracks usage in `SNOWFLAKE.ACCOUNT_USAGE.PRIVACY_BUDGET_USAGE`. Plan ε allocation per analyst/role/time window.

### 68. Encryption at rest — defaults
**Front:** What encryption does Snowflake apply at rest by default?
> **AES-256**, automatic key rotation. Key hierarchy: Account Master Key → File Keys. **Tri-Secret Secure (TSS)** adds a customer-held key as a required third factor (Business Critical+).

### 69. TLS version
**Front:** Which TLS version does Snowflake enforce in transit?
> **TLS 1.2 or higher.** PrivateLink supported for cloud-vendor private connectivity (AWS PrivateLink, Azure Private Link).

### 70. Synthetic data vs. masking
**Front:** Why might you prefer synthetic data over masking for AI dev environments?
> Synthetic data: no real records (lower privacy risk), referential integrity preservable, utility for training preserved, often out-of-scope for compliance. Masking preserves originals (with redactions) — masked PII is still PII legally.

### 71. Clean room — governance payoff
**Front:** Three governance benefits of a Snowflake Data Clean Room?
> 1. Collaboration without raw data egress (satisfies purpose limitation).
> 2. Approved-query-only execution (purpose is the query).
> 3. Audit via the clean room's query log.
> Plus optional DP + aggregation floors.

### 72. Masking policy on type
**Front:** Can one masking policy apply to columns of different data types?
> **No.** A masking policy's signature is a specific type (e.g., `(val STRING) RETURNS STRING`). You need a separate policy per data type. Tag-based binding uses `FOR (<data_type>)` to route the right type.

---

## Domain 5 — Model Governance & Monitoring

### 73. Model Registry — what it stores
**Front:** Name five things the Model Registry stores per model.
> 1. Versions (immutable per version).
> 2. Metadata (training data, metrics, author, timestamps).
> 3. Tags (RISK_TIER, OWNER, APPROVED).
> 4. Artifacts (serialized bytes).
> 5. Default version pointer (aliases for atomic promotion).

### 74. Model deployment gate — pattern
**Front:** Describe a Snowflake-native deployment gate pattern for AI models.
> Required stored procedure `DEPLOY_MODEL(name, version)` reads MODEL_METRICS table; refuses deployment if required metrics (bias, drift baseline) aren't present within the last N days. Writes to DEPLOYMENT_APPROVALS table on success. Audit via ACCESS_HISTORY.

### 75. PSI thresholds
**Front:** What PSI values indicate drift severity?
> PSI < 0.1 → low. 0.1 ≤ PSI < 0.25 → moderate (investigate). PSI ≥ 0.25 → significant drift. Population Stability Index applied per feature between a baseline window and a current window.

### 76. Three drift types
**Front:** Name the three drift types and the detection signal for each.
> **Data drift (covariate shift)** — input distribution changes. KS test, PSI.
> **Concept drift** — input-output relationship changes. Prediction-vs-actual, AUC/F1 drops.
> **Performance drift** — effective metric drops regardless of cause. Ongoing precision/recall vs. baseline.

### 77. Cortex AI Observability
**Front:** What does Cortex AI Observability capture, and where does it land?
> Prompts, completions, token counts, model name/version, latency, cost, guardrail triggers, trace spans (RAG pipelines, agentic workflows), errors. Lands in an **event table** (customer-provisioned or Snowflake-managed). Queryable via SQL. OpenTelemetry integration available.

### 78. LLM eval metrics
**Front:** Define faithfulness, relevance, and groundedness for LLM output evaluation.
> **Faithfulness** — response stays true to provided context (critical for RAG).
> **Relevance** — response addresses the user's question.
> **Groundedness** — each factual claim is supported by cited sources.
> Also: context precision/recall, answer correctness, toxicity, PII leakage.

### 79. Prompt injection — layered defense
**Front:** Name five layers of prompt injection defense in Snowflake.
> 1. Input sanitization (regex/UDF checks for "ignore previous instructions" patterns).
> 2. Structured message format (separate system vs. user roles).
> 3. RAG document scanning (detect hostile instructions in retrieved content).
> 4. Cortex Guard (pre and post content safety).
> 5. Output validation + Cortex AI Observability (post-hoc analysis).
> 6. Least-privilege tooling (LLM-generated SQL runs under caller's role with policies).

### 80. Direct vs. indirect prompt injection
**Front:** What's indirect prompt injection?
> Hostile instructions **embedded in retrieved content** (e.g., a webpage ingested via RAG) that the LLM reads as part of its context and acts on. Harder to detect than direct injection because the attack doesn't appear in the user's typed prompt.

### 81. Model object as securable
**Front:** Which Snowflake privilege lets a role invoke a model's PREDICT method?
> **USAGE ON MODEL.** `GRANT USAGE ON MODEL AI_DB.MODELS.CHURN TO ROLE FR_INFERENCE_SERVICE`. OWNERSHIP is needed to register new versions or drop.

### 82. Model version immutability
**Front:** True or false: a model version in the Snowflake Model Registry can be updated.
> **False.** Model versions are immutable. To change, create a new version. You can change the **default version pointer** or alias to a different version.

### 83. Alias in Model Registry
**Front:** Why use a model alias?
> Atomic promotion: an alias (e.g., `prod`) points to a specific version. Promotions flip the alias; the underlying version is unchanged. Consumers reference the alias, enabling blue/green deployment.

### 84. Semantic model as governance artifact
**Front:** For Cortex Analyst, which configuration file is a governance artifact and why?
> The **semantic model YAML**. It defines which tables, columns, metrics, and dimensions Analyst exposes. If it exposes a column, Analyst will reference it in generated SQL (though masking/row-access still apply). Version-control the YAML; review for over-exposure.

---

## Domain 6 — Compliance & Audit

### 85. Shared responsibility — Snowflake side
**Front:** Name five things Snowflake handles under the shared responsibility model.
> Physical/network security, platform encryption (at rest default, in transit TLS 1.2+), compliance attestations (SOC 2 etc.), infra patching, platform logs (ACCOUNT_USAGE collection), key hierarchy baseline, Cortex LLM availability.

### 86. Shared responsibility — customer side
**Front:** Name five things the customer handles under shared responsibility.
> User lifecycle (SSO/SCIM/MFA), data classification, policy authorship + binding, compliance scoping of workloads, audit retention beyond 365 days + export, CMK rotation (if TSS), model governance artifacts (model cards, approvals).

### 87. Snowflake attestations — which editions
**Front:** Which Snowflake edition is required for HIPAA and PCI?
> **Business Critical** or **Virtual Private Snowflake (VPS)**. HIPAA additionally requires a signed BAA. Standard and Enterprise editions do not meet HIPAA/PCI.

### 88. FedRAMP region
**Front:** Where are Snowflake's FedRAMP-authorized accounts hosted?
> **US GovCloud** regions (e.g., AWS GovCloud, Azure Government). Not available in commercial regions. FedRAMP Moderate and High authorizations exist.

### 89. ACCOUNT_USAGE latency summary
**Front:** List typical latencies for LOGIN_HISTORY, ACCESS_HISTORY, POLICY_REFERENCES.
> **LOGIN_HISTORY** ~15 minutes. **ACCESS_HISTORY** ~45 minutes. **POLICY_REFERENCES** ~2 hours. QUERY_HISTORY ~45 minutes. Use INFORMATION_SCHEMA for near-real-time (database-scoped).

### 90. ACCOUNT_USAGE retention
**Front:** How long does Snowflake retain ACCOUNT_USAGE views?
> **365 days.** For longer retention, build an append-only landing table and schedule a task to copy new rows. Replicate to a separate account for tamper-evident archive.

### 91. Immutable audit log — key controls
**Front:** List three controls that make a customer-side audit log tamper-evident.
> 1. INSERT + SELECT grants only; no DELETE/UPDATE grants.
> 2. Replication to a separate, air-gapped Snowflake account.
> 3. Iceberg table on external storage with S3 Object Lock or equivalent versioning.
> 4. Alerts on grant changes to the audit database.

### 92. GDPR DSAR deadline
**Front:** What is the GDPR deadline to respond to a Data Subject Access Request?
> **30 days** from receipt (free of charge). Extendable by another 60 days with notice if complex. The exam has tested knowing the 30-day baseline.

### 93. GDPR Art. 17 — right to erasure quirks in Snowflake
**Front:** Name two Snowflake quirks that affect right-to-erasure fulfillment.
> 1. **Time Travel** — deleted rows remain recoverable for the retention window; shorten retention or `PURGE` if required.
> 2. **Fail-safe** — 7-day disaster recovery; not user-purgeable (requires Snowflake Support).
> 3. **Clones** — share partitions until modified; data may persist in clones.
> 4. **Model training** — models already trained have "seen" the data; retrain or document legal basis.

### 94. GDPR fines
**Front:** What are GDPR's maximum fines?
> Up to **€20 million or 4% of global annual turnover**, whichever is higher. (EU AI Act for prohibited practices is higher — 7%.)

### 95. HIPAA Technical Safeguards — key sections
**Front:** Which HIPAA sections cover access control, audit, and encryption?
> **§ 164.312(a)** Access control. **§ 164.312(b)** Audit controls. **§ 164.312(c)** Integrity. **§ 164.312(d)** Authentication. **§ 164.312(e)** Transmission security (encryption in transit).

### 96. PCI DSS and tokenization
**Front:** How does tokenization affect PCI DSS scope?
> If raw PANs are **replaced by tokens** stored only in Snowflake (plaintext PANs live in an external vault), PCI scope **may be reduced** — the Snowflake storage no longer holds cardholder data. Confirm scope reduction with a QSA; depends on token irreversibility and vault segregation.

### 97. Breach notification — GDPR timeline
**Front:** Under GDPR, how fast must a controller notify its DPA of a breach?
> **72 hours** of becoming aware (Art. 33). Snowflake customers configure Trust Center alerts + custom scanners on ACCESS_HISTORY to meet this timeline.

### 98. Legal hold pattern
**Front:** How do you implement a legal hold in Snowflake?
> Suspend deletion via (a) extending Time Travel retention, (b) copying affected tables to a dedicated hold schema with INSERT/SELECT-only grants, (c) tagging objects with `LEGAL_HOLD='TRUE'` and a policy that blocks DELETE for roles without exemption, (d) suspend tasks that would purge. Document in an audit record.

### 99. Trust Center scanners
**Front:** Name three Trust Center scanner packages.
> **CIS Benchmarks** (security configuration), **Snowflake Security Reference Architecture (SRA)**, **Threat Intelligence**. Custom scanners can be written as SQL stored procedures returning finding rows.

### 100. Compliance → feature mapping (highlights)
**Front:** Give the primary Snowflake feature for GDPR Art. 15, HIPAA § 164.312(b), PCI Req 7, SOC 2 CC6.
> **GDPR Art. 15 (DSAR)** — DATA_CLASSIFICATION_LATEST + TAG_REFERENCES + custom DSAR procedure.
> **HIPAA § 164.312(b) (Audit)** — ACCESS_HISTORY + QUERY_HISTORY + LOGIN_HISTORY.
> **PCI Req 7 (Least privilege)** — Functional + Access roles pattern, Managed Access schemas.
> **SOC 2 CC6 (Logical access)** — RBAC + Network Policy + Authentication Policy + MFA.

---

## Study Routine

- **Day 1–2:** Review flashcards 1–20 (high-frequency topics). Repeat until recall is immediate.
- **Day 3–4:** Add 21–60 (domains 1–3).
- **Day 5–6:** Add 61–100 (domains 4–6).
- **Day 7:** Full-deck shuffle, then mock exam.
- Any card you miss → re-queue for next day.

Pair with `GAS-C01_MOCK_EXAM.md` to validate recall against scenarios.
