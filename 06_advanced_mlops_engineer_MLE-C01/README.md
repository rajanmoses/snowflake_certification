# Snowflake Advanced MLOps Engineer (MLE-C01) — Study Guide Index

> Exam-focused study pack for the Snowflake **Advanced MLOps Engineer** certification.
> Each `_QA.md` file in this directory contains 25 scenario-grade questions with
> deep answers, first-principles reasoning, ASCII diagrams, labs, and exam traps.
> This README is your navigation layer.

---

## Certification at a Glance

| Attribute | Value |
|---|---|
| **Exam code** | MLE-C01 |
| **Full name** | SnowPro Advanced: Machine Learning Engineer / MLOps Engineer |
| **Format** | Proctored, computer-based (SnowPro delivery) |
| **Questions** | ~65 multiple-choice / multi-select |
| **Duration** | 115 minutes |
| **Passing score** | ~750 / 1000 (scaled) |
| **Prerequisite** | SnowPro Core (active) |
| **Validity** | 2 years from pass date (recertification required) |
| **Primary stack** | Snowpark ML, Model Registry, Feature Store, Model Monitor, Cortex AI, SPCS, Dynamic Tables, Snowflake CLI |

---

## Blueprint Weights (Approximate)

Snowflake rebalances blueprint weights periodically — treat these as directional, not
authoritative. The Q&A modules are sized proportionally so study hours match the exam.

| # | Domain | Weight | Primary file |
|---|---|---|---|
| 1 | **ML Workflow Design** | 18-22% | `01_ml_workflow_design_QA.md` |
| 2 | **Feature Engineering & Feature Store** | 15-18% | `02_feature_engineering_and_store_QA.md` |
| 3 | **Model Training & Registry** | 18-22% | `03_model_training_and_registry_QA.md` |
| 4 | **Model Deployment & Inference** | 18-22% | `04_model_deployment_inference_QA.md` |
| 5 | **Monitoring & Observability** | 12-15% | `05_monitoring_and_observability_QA.md` |
| 6 | **MLOps Automation & Governance** | 15-18% | `06_mlops_automation_governance_QA.md` |

Total deep questions: **150** (25 × 6). Plus a 65-question mock exam and 100+ flashcards
in the companion files below.

---

## Module Summaries

### 01 — ML Workflow Design (18-22%)
End-to-end lifecycle on Snowflake: ingestion → curation → features → train → register →
deploy → monitor → retrain. Snowpark ML vs classic Snowpark. Warehouse vs SPCS vs ML Jobs
decision factors. Batch vs real-time inference patterns. Training-serving skew prevention.
Dynamic Tables as the ML pipeline primitive. Streamlit-in-Snowflake for UX. Cortex vs
custom-registered LLMs. **Big-picture architecture questions live here.**

### 02 — Feature Engineering & Feature Store (15-18%)
`snowflake.ml.feature_store.FeatureStore`, Entity, FeatureView (managed vs static),
Dataset. Point-in-time joins via `generate_dataset()`. Online vs offline serving.
Feature view versioning + schema evolution. Backfills, time-travel, feature tagging.
Preventing target leakage via timestamp discipline. Governance of feature views (RBAC,
masking, row access). **Hinges on PIT join mechanics and schema evolution rules.**

### 03 — Model Training & Registry (18-22%)
Snowpark ML Modeling API (`modeling.xgboost`, `modeling.lightgbm`, `modeling.pipeline`,
`modeling.preprocessing`, HP tuning). Distributed/GPU training via SPCS + `jobs.remote`.
`snowflake.ml.registry.Registry` — `log_model`, versions, aliases (DEFAULT, PROD),
tags, metrics, signatures. CustomModel for ensembles/business logic. Dependency pinning.
Reproducibility (Dataset versions + git_commit tags). **Deep API-surface questions.**

### 04 — Model Deployment & Inference (18-22%)
Warehouse UDF vs SPCS Service vs Cortex vs External Function. `mv.create_service()`
with compute pools, image repos, ingress, min/max replicas, GPU requests. Dynamic Tables
for batch inference. Vectorized Python UDFs. Scheduled Tasks. A/B testing & shadow
deployments. Autoscaling. External access from mobile/web. Cost comparison modeling.
**Deployment decision matrix is a near-guaranteed exam topic.**

### 05 — Monitoring & Observability (12-15%)
Snowflake Model Monitor — baseline, drift metrics (PSI, KS, chi²), performance metrics
with actuals, scheduled refresh. Cortex AI Observability for LLMs (groundedness,
faithfulness). Prediction log schema design. Delayed/async ground truth. Alerts firing
retraining SPROCs. Event Tables. Lineage via ACCESS_HISTORY. **Formulas and threshold
interpretations are testable.**

### 06 — MLOps Automation & Governance (15-18%)
`snow` CLI + GitHub Actions / Jenkins pipelines. `CREATE GIT REPOSITORY` + `ALTER GIT
REPOSITORY FETCH`. RBAC on models/versions/services. Approval workflows via tags +
SPROCs. Cost chargeback via QUERY_TAG and warehouse metering. Secrets + External Access
Integrations. PII handling (masking, row policies, Horizon Catalog). Audit via
ACCOUNT_USAGE. Dev → stage → prod promotion patterns. **Scenario-heavy governance
questions.**

---

## Recommended Study Order

**Weeks 1-2: Foundation (modules 01 + 02)**
Read `01_ml_workflow_design_QA.md` cover-to-cover. Internalize the lifecycle diagram.
Then `02_feature_engineering_and_store_QA.md` with a focus on Q4 (PIT joins), Q6 (FV
versioning), and Q11 (leakage prevention).

**Weeks 3-4: Build (modules 03 + 04)**
`03_model_training_and_registry_QA.md` — memorize the `Registry.log_model()` signature
and the alias/version model. Then `04_model_deployment_inference_QA.md` — build the
mental matrix of UDF vs SPCS vs Cortex.

**Week 5: Operate (modules 05 + 06)**
`05_monitoring_and_observability_QA.md` — be able to compute PSI by hand from a bin
table. `06_mlops_automation_governance_QA.md` — walk through a dev→prod promotion with
RBAC + tags.

**Week 6: Rehearse (companions)**
- Day 1-2: Burn through `MLE-C01_FLASHCARDS.md` (both directions).
- Day 3: Pass the `MLE-C01_MOCK_EXAM.md` under time pressure.
- Day 4-5: Review wrong answers against the source `_QA.md` files.
- Day 6: Final skim of `MLE-C01_CHEATSHEET.md`.
- Day 7: Rest, then take the real exam.

---

## Companion Documents

| File | Purpose | When to use |
|---|---|---|
| `MLE-C01_FLASHCARDS.md` | 100+ atomic Q↔A cards | Daily spaced repetition; last-week rapid review |
| `MLE-C01_MOCK_EXAM.md` | 65 scenario questions + answer key | Week 6 timed practice; identify weak domains |
| `MLE-C01_CHEATSHEET.md` | Dense API/formula/pattern reference | Night-before skim; exam-day warmup |
| `01_ml_workflow_design_QA.md` ... `06_mlops_automation_governance_QA.md` | Deep conceptual mastery | First-pass learning; targeted re-reads |

---

## Tips for MLOps Scenario Questions

Snowflake MLE-C01 is heavily scenario-based. Patterns that recur:

1. **"A team wants X latency / Y throughput / Z cost — which path?"**
   Train the UDF-vs-SPCS-vs-Cortex matrix until it's reflexive. See cheatsheet
   "Deployment matrix" and `04_*` Q1, Q5, Q17.

2. **"Which Snowflake service for this phase?"**
   The lifecycle in `01_*` Q2 is a key diagram — memorize the phase→service mapping.

3. **"The team sees training/production accuracy divergence — root cause?"**
   Nearly always a skew issue. Check: feature view used in both pipelines?
   Preprocessing baked into model? PIT join on training spine? (`01_*` Q5, `02_*` Q4+Q11.)

4. **"Which drift metric for this feature type?"**
   PSI = both numeric (binned) and categorical, business-friendly thresholds.
   KS = numeric continuous, pure distribution test.
   Chi² = categorical with expected-vs-observed counts. (`05_*` Q2+Q3.)

5. **"How do you promote a model to PROD safely?"**
   Alias flip (atomic) + tag-based approval gate + rollback path. Never overwrite a
   version. Keep a shadow deployment for comparison. (`03_*` Q11, `06_*` Q5+Q10+Q18.)

6. **"Compliance / audit / lineage question"**
   `ACCESS_HISTORY`, `QUERY_HISTORY`, Object Tags, Horizon Catalog, Model Registry tags.
   (`06_*` Q7, Q11, Q20.)

7. **Multi-answer questions on "which of the following are true about X" —** read
   each option as a standalone claim. Snowflake loves to put one subtle lie among four
   truths (e.g., "Dynamic Tables support external access UDFs" — false).

8. **When two answers look right, pick the one that aligns with Snowflake-native
   abstractions.** "Build it yourself with Streams+Tasks" vs "use Dynamic Table" —
   the native-abstraction answer wins unless the question explicitly rules it out.

9. **Beware of "always" / "never" / "only" in distractors** — Snowflake rarely has
   absolutes. Services and APIs typically have escape hatches.

10. **Watch for API staleness** — some older patterns (`snowflake.ml.model.ModelDeployment`,
    `deploy()` method) have been superseded by `create_service()` + Registry. If an
    answer references deprecated API, it's usually a distractor.

---

## External References

- Snowflake docs: [Snowpark ML](https://docs.snowflake.com/en/developer-guide/snowpark-ml/index)
- Snowflake docs: [Model Registry](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/overview)
- Snowflake docs: [Feature Store](https://docs.snowflake.com/en/developer-guide/snowflake-ml/feature-store/overview)
- Snowflake docs: [Model Monitor](https://docs.snowflake.com/en/developer-guide/snowflake-ml/model-registry/model-observability)
- Snowflake docs: [Snowpark Container Services](https://docs.snowflake.com/en/developer-guide/snowpark-container-services/overview)
- Snowflake docs: [Cortex AI](https://docs.snowflake.com/en/user-guide/snowflake-cortex/overview)
- Snowflake docs: [Snowflake CLI](https://docs.snowflake.com/en/developer-guide/snowflake-cli/index)

---

## Checklist Before Exam Day

- [ ] All six `_QA.md` modules read at least once; weak domains re-read
- [ ] Flashcards: ≥90% recall on the Snowpark ML API surface cards
- [ ] Mock exam attempted under 115-minute time limit, score ≥ 75%
- [ ] Cheatsheet reviewed, especially: deployment matrix, PSI formula,
      Registry workflow, Monitor DDL, `create_service()` kwargs
- [ ] Account credentials/ID practice setup confirmed (SnowPro delivery portal)
- [ ] 8 hours of sleep. No last-minute cramming on exam morning — rely on
      spaced repetition, not panic.

Good luck.
