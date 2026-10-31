# MLE-C01 Mock Exam

> **Format:** 65 questions, 115 minutes. Mirrors the real exam distribution.
> Mix: 55 single-answer, 10 multi-select (marked [MULTI]).
> Scenario-heavy, consistent with Snowflake's SnowPro Advanced style.
>
> **How to use:**
> 1. Attempt all 65 questions in **115 minutes** (no breaks). Mark your answers.
> 2. Score yourself with the answer key at the end.
> 3. Target: **≥ 75%** before sitting the real exam.
> 4. For every wrong answer, revisit the pointed `_QA.md` section.

---

## Question Distribution

| Domain | Qs | Range |
|---|---|---|
| 01 — ML Workflow Design | 12 | Q1-Q12 |
| 02 — Feature Engineering & Store | 11 | Q13-Q23 |
| 03 — Training & Registry | 13 | Q24-Q36 |
| 04 — Deployment & Inference | 13 | Q37-Q49 |
| 05 — Monitoring & Observability | 9 | Q50-Q58 |
| 06 — MLOps Automation & Governance | 7 | Q59-Q65 |

---

## Section 1 — ML Workflow Design (12 questions)

**Q1.** A data scientist runs `from snowflake.ml.modeling.xgboost import XGBClassifier`
and calls `.fit(df)` on a 500M-row Snowpark DataFrame. Where does the training
computation actually run?
- A) On the client machine, after Snowpark pulls the data via Arrow.
- B) On the Snowflake warehouse, as a Python stored procedure invoked by the SDK.
- C) On a Snowpark Container Services (SPCS) GPU pool automatically provisioned.
- D) On a Cortex-managed compute pool shared across tenants.

**Q2.** A team must choose between the Snowpark ML Modeling API and ML Jobs on SPCS
for training a 7-billion-parameter LLM via LoRA fine-tuning. Which is appropriate?
- A) Snowpark ML Modeling API on an XL warehouse.
- B) Snowpark ML Modeling API on a Snowpark-optimized warehouse.
- C) ML Jobs via `@remote` decorator on a GPU compute pool.
- D) Cortex Fine-Tuning only (LoRA is not supported outside Cortex).

**Q3.** Which pattern is best for scoring 100M customers nightly for churn probability
with no latency SLA tighter than a few hours?
- A) Model Serving SPCS service with `min_instances=4`.
- B) Dynamic Table over the feature table invoking `MODEL!PREDICT(...)` with `TARGET_LAG='3 hours'`.
- C) An External Function calling an EC2-hosted model.
- D) Streamlit app that users invoke on demand.

**Q4.** Which statement about training-serving skew prevention on Snowflake is correct?
- A) Logging a scaler separately from the model is required to avoid skew.
- B) Packaging preprocessing in a Snowpark ML Pipeline and registering the whole pipeline is the primary defense.
- C) Dynamic Tables cannot help prevent skew because they are refresh-independent of inference.
- D) Feature Store feature views cannot be used for online serving, only offline training.

**Q5.** You need < 50ms inference latency for a tabular model, 24/7, serving a mobile
app. Choose the deployment path.
- A) Warehouse UDF with auto-resume enabled.
- B) Scheduled Task writing to a predictions table consumed by the app.
- C) SPCS Model Serving service via `create_service(ingress_enabled=True, min_instances≥1)`.
- D) Snowpipe-based inference pipeline.

**Q6.** [MULTI] Which of the following are valid Snowpark ML Modeling API classes?
(Select all that apply.)
- A) `snowflake.ml.modeling.xgboost.XGBClassifier`
- B) `snowflake.ml.modeling.sklearn.RandomForestClassifier`
- C) `snowflake.ml.modeling.lightgbm.LGBMClassifier`
- D) `snowflake.ml.modeling.pipeline.Pipeline`
- E) `snowflake.ml.modeling.preprocessing.StandardScaler`

**Q7.** The business requires predictions to be no more than 15 minutes stale, with
automatic refresh and incremental processing. Which choice is best?
- A) Streams + Tasks orchestrating a MERGE every 15 minutes.
- B) Dynamic Table over the features with `TARGET_LAG = '15 minutes'`.
- C) Scheduled stored procedure every 15 minutes.
- D) External Function invoking a managed service every 15 minutes.

**Q8.** When should you use Cortex LLM functions over registering your own LLM?
- A) Always — Cortex is cheaper regardless of use case.
- B) When the task is a general-purpose foundation-model task and no custom weights are needed.
- C) When you need deterministic, pinned transformers library versions.
- D) When you need to A/B test two custom-fine-tuned open-source models.

**Q9.** Which best describes `TARGET_LAG = 'DOWNSTREAM'` on a Dynamic Table?
- A) The DT refreshes immediately whenever its source changes.
- B) The DT only refreshes when a downstream DT needs its data to meet its own lag.
- C) The DT refreshes on a 24-hour schedule.
- D) The DT is deprecated and should not be used in new pipelines.

**Q10.** You need to materialize a reproducible training snapshot that will be shuffled
and streamed to PyTorch DataLoaders on multiple GPUs. Which Snowflake abstraction fits?
- A) A Snowpark DataFrame passed directly to `torch.utils.data.DataLoader`.
- B) A `snowflake.ml.dataset.Dataset` created via `DatasetFactory.create_from_dataframe()` with multi-shard output.
- C) A `COPY INTO` to S3 and let PyTorch read S3 directly.
- D) A Model Monitor source table filtered by date.

**Q11.** Streamlit-in-Snowflake (SiS) is appropriate for which use case?
- A) Serving a public-internet consumer app with 10k QPS.
- B) An internal reviewer UI that calls a Registry model via SQL for ad-hoc scoring.
- C) Hosting a long-running inference endpoint for a mobile client.
- D) Replacing a Model Monitor's drift computation.

**Q12.** A Snowpark-optimized warehouse differs from a standard warehouse how?
- A) It is GPU-enabled.
- B) It has roughly 16x more memory per node, suited to memory-heavy single-node ML.
- C) It auto-attaches to SPCS compute pools.
- D) It is the only warehouse type that can run Snowpark ML Modeling API jobs.

---

## Section 2 — Feature Engineering & Feature Store (11 questions)

**Q13.** Which three objects comprise the Snowflake Feature Store? (Pick the best grouping.)
- A) Pipeline, Model, Registry
- B) Entity, FeatureView, Dataset
- C) Stage, Stream, Task
- D) Monitor, Metric, Alert

**Q14.** You define a FeatureView without `refresh_freq`. What happens?
- A) It's rejected — `refresh_freq` is required.
- B) The feature values compute once at registration time; it won't auto-refresh.
- C) A Dynamic Table is still created with a default lag of 1 day.
- D) Snowflake rebuilds it every time it's queried.

**Q15.** You want to pin each training spine row to feature values as-of the spine's
timestamp. What enables this in Snowflake Feature Store?
- A) A regular INNER JOIN on the entity key.
- B) `fs.generate_dataset()` with `spine_timestamp_col` set and FeatureViews that have `timestamp_col`.
- C) Snowflake Streams captured at exactly the spine timestamp.
- D) MATCH_RECOGNIZE patterns.

**Q16.** [MULTI] Which of the following cause target leakage in a feature view? (Select all.)
- A) Including a feature whose value changes *after* the label event occurs.
- B) Omitting `timestamp_col` on an event-based FeatureView used in training.
- C) Using an aggregated feature over the trailing 30 days with proper timestamp filtering.
- D) Joining on an entity key only, without temporal condition.

**Q17.** A new feature is added to a FeatureView. What is the recommended action?
- A) Modify the existing FeatureView in place and refresh.
- B) Register a new version (e.g., V2) of the FeatureView.
- C) Create a new entity to track the new feature.
- D) Delete the existing dataset and republish.

**Q18.** The Feature Store serves online features for real-time inference by which mechanism?
- A) A dedicated Redis cache populated by a Snowpipe pipeline.
- B) Querying the backing Dynamic Table (or materialized FV) by entity key with low target lag.
- C) `MODEL!PREDICT` fetches features implicitly from the FeatureView.
- D) External Functions proxying to a feature server.

**Q19.** How do you backfill a new managed FeatureView to cover historical dates not
yet in the source stream?
- A) It cannot be backfilled — only forward-looking features are supported.
- B) Materialize via `Dataset` snapshots at historical `as_of` dates, or use `ALTER DYNAMIC TABLE ... REFRESH` with historical scope.
- C) Delete the FeatureView and reload the entire warehouse history.
- D) Set `TARGET_LAG = -∞`.

**Q20.** Which privilege model governs a FeatureView?
- A) FeatureViews are public by default and cannot be restricted.
- B) Standard RBAC on the FeatureStore schema and underlying DTs; masking policies on sensitive columns are honored.
- C) Only account admins may read FeatureViews.
- D) FeatureViews bypass masking policies for consistency.

**Q21.** You want to combine Snowflake Feature Store aggregations with Cortex LLM
embeddings as features. What's the best approach?
- A) Embeddings cannot be stored as features in the Feature Store.
- B) Compute embeddings via `SNOWFLAKE.CORTEX.EMBED_TEXT_768(...)` as a column in the FeatureView's SQL.
- C) Store embeddings in Pinecone and join at serve time.
- D) Only use embeddings at train time; drop them from online features.

**Q22.** [MULTI] Valid uses of FeatureView tags? (Select all.)
- A) Discoverability (searching features by domain, owner, PII sensitivity).
- B) Driving masking policies and row-access policies.
- C) Enforcing numeric thresholds on refresh latency.
- D) Supporting approval workflows (e.g., `pii=yes` requires reviewer).

**Q23.** The team has an existing Feast feature store on AWS and wants to migrate to
Snowflake Feature Store. What is a reasonable first step?
- A) Rewrite every Feast feature from scratch; there's no automation.
- B) Materialize Feast feature definitions as Snowflake tables, define matching Entities, re-create FeatureViews via SDK, and verify PIT join equivalence against a small spine.
- C) Point Feast at Snowflake as a storage backend only; no SDK migration needed.
- D) Feast and Snowflake Feature Store cannot coexist; delete Feast immediately.

---

## Section 3 — Training & Registry (13 questions)

**Q24.** The `log_model()` call does which of the following? (Pick the most complete answer.)
- A) Only writes metadata; the artifact must be uploaded separately.
- B) Serializes the artifact, captures dependencies, infers signature, registers metadata, deploys a warehouse UDF, and prepares for SPCS build.
- C) Only serializes the artifact; signature must be set manually.
- D) Logs the model but does not deploy any UDF — a separate `deploy()` call is required.

**Q25.** You log an sklearn `RandomForestClassifier` via the Registry. What must you provide?
- A) `conda_dependencies` only.
- B) `sample_input_data`, so Snowflake can infer the model signature.
- C) A fitted Snowpark ML Pipeline wrapper.
- D) A `CustomModel` subclass.

**Q26.** Which of the following is true about model versions in the Registry?
- A) Versions are mutable — you can overwrite the artifact in place.
- B) Versions are immutable; to change a version you log a new one.
- C) Only one version per model is retained.
- D) Versions are automatically deleted after 30 days.

**Q27.** An alias in the Model Registry is best described as:
- A) A reserved role for RBAC on the model.
- B) A named pointer to a version (e.g., `PROD → V17`) that can be flipped atomically.
- C) A tag with a numeric value.
- D) A deprecated synonym for version.

**Q28.** You train an XGBoost model on an L warehouse, log it via `log_model`. The UDF
fails at inference with "ModuleNotFoundError: xgboost". Likely cause?
- A) The warehouse running inference is paused.
- B) `conda_dependencies` did not include `xgboost` (or wrong version) at log time.
- C) The model was logged with `enable_explainability=False`.
- D) The signature was mis-inferred.

**Q29.** [MULTI] Which model types does the Snowflake Model Registry support natively
without `CustomModel`? (Select all.)
- A) Scikit-learn estimators
- B) XGBoost
- C) PyTorch (nn.Module)
- D) HuggingFace Transformers pipelines
- E) An arbitrary Python class with no inheritance

**Q30.** You need to serve a RAG pipeline: retrieve top-k from a vector table, then call
a fine-tuned generator. Which approach is appropriate?
- A) Log just the generator in Registry; retrieval must happen in client code.
- B) Build a `CustomModel` subclass with an `@inference_api predict()` that does retrieval + generation; register that.
- C) Not possible on Snowflake today.
- D) Use only Cortex `SEARCH_PREVIEW` with no Registry involvement.

**Q31.** [MULTI] Which hyperparameter-search options are built into Snowpark ML? (Select all.)
- A) `snowflake.ml.modeling.model_selection.GridSearchCV`
- B) `snowflake.ml.modeling.model_selection.RandomizedSearchCV`
- C) `snowflake.ml.modeling.model_selection.BayesianOptSearchCV`
- D) Custom Optuna integration via `@remote` on SPCS

**Q32.** You need GPU training for a Vision Transformer. Which path?
- A) Snowpark ML Modeling API with GPU flag.
- B) ML Jobs via `@remote(compute_pool='GPU_NV_M_POOL', pip_requirements=['torch==2.3','transformers==4.42'])`.
- C) Snowpipe + Streams pipeline with GPU compute.
- D) Cortex Fine-Tuning on text-only models.

**Q33.** Promoting a tested version to PROD atomically is best done by:
- A) Deleting the old PROD version.
- B) Calling `set_alias('PROD')` on the new version — all consumers of `MODEL:PROD` flip instantly.
- C) Logging a new model with the name `PROD` and deleting the alias.
- D) Running a batch update on the predictions table.

**Q34.** To ensure a trained model is reproducible six months later, you should pin all
of the following EXCEPT:
- A) `conda_dependencies` versions.
- B) The Dataset version used as training input.
- C) Random seeds in model and Pipeline.
- D) The warehouse size that was used.

**Q35.** You attempt to use a Snowpark ML Pipeline whose last step is a `StandardScaler`,
then call `.predict()`. What happens?
- A) It predicts using the scaler as a model.
- B) It fails — the last step must be an estimator (predictor), not a transformer.
- C) It auto-adds a dummy classifier.
- D) It falls through to the first predictor in the chain.

**Q36.** A model version has metrics `{"roc_auc": 0.87, "f1": 0.74}`. You decide to add
`{"calibration_error": 0.03}` after the fact. Can you?
- A) No — metrics are immutable once logged.
- B) Yes — `mv.set_metric("calibration_error", 0.03)` updates them.
- C) Only by relogging a new version.
- D) Only by overwriting the entire metrics dict via `mv.replace_metrics()`.

---

## Section 4 — Deployment & Inference (13 questions)

**Q37.** The two primary deployment paths for a Registry-logged model are:
- A) Snowpipe and External Functions.
- B) Warehouse UDF (invoked via SQL) and SPCS Service (via `create_service`).
- C) Streamlit and Tasks.
- D) Dynamic Tables and Streams only.

**Q38.** Which of these is a required argument to `mv.create_service()`?
- A) `region`
- B) `service_compute_pool`
- C) `warehouse_size`
- D) `signature`

**Q39.** You enable `ingress_enabled=True` on a Model Service. What does this provide?
- A) The service becomes read-only.
- B) A public HTTPS endpoint fronted by Snowflake-managed OAuth.
- C) The service shuts down on idle.
- D) The service is forced to use GPU.

**Q40.** You deploy a vectorized Python UDF for model inference. Why is "vectorized"
important?
- A) It allows GPU access.
- B) It processes rows as pandas DataFrame batches, amortizing model-load and inference overhead across many rows.
- C) It bypasses Snowflake's query optimizer.
- D) It enables external access.

**Q41.** Which pattern is correct for A/B testing two model versions?
- A) Log both, serve the one named `A` in production and the one named `B` in a sandbox; never compare in prod.
- B) Deploy both; route by stable hash (e.g., `MOD(HASH(CUSTOMER_ID), 100) < 10 → B, else A`); log which version scored each row.
- C) Use a Dynamic Table to randomly route — reads will reshuffle on refresh.
- D) Deploy a single version and manually swap the alias every hour.

**Q42.** A customer app needs < 100ms LLM response times for a 7B-param model. Which
deployment?
- A) Warehouse UDF on an XL warehouse.
- B) SPCS Model Service on a GPU compute pool (e.g., `GPU_NV_M`) with `min_instances ≥ 1` and `ingress_enabled=True`.
- C) Cortex `COMPLETE` with a 7B model from the catalog.
- D) Scheduled task writing predictions to a table.

> *Note: C may be viable **if** the needed 7B model is in Cortex's catalog. The question
> says "a 7B-param model" (custom), implying B. If the model is in Cortex catalog, C is
> cheaper. See answer key.*

**Q43.** [MULTI] Which can call a Snowflake Model Service endpoint externally? (Select all.)
- A) A mobile app with Snowflake OAuth token.
- B) An on-prem app via HTTPS with Snowflake credentials.
- C) A Snowflake UDF implicitly registered by `create_service`.
- D) An External Function.

**Q44.** `max_batch_rows` on an SPCS service controls:
- A) The maximum number of rows the service accepts per prediction request (batching for throughput).
- B) The maximum rows in the backing table.
- C) The CPU cores allocated.
- D) The DT refresh cadence.

**Q45.** You observe that an SPCS service shows high p95 latency during request bursts.
Which parameter change is most likely to help?
- A) Decrease `max_instances`.
- B) Increase `max_instances` and ensure `min_instances` ≥ typical burst steady state.
- C) Switch to warehouse UDF.
- D) Disable `ingress_enabled`.

**Q46.** The best approach for inference on partitioned data (one model per region)?
- A) Register all region models under one name, use alias flipping to switch regions.
- B) Register separate models per region (or one model with per-region versions); in the inference SQL, use CASE/UNION ALL to route each row to its region's model UDF.
- C) Train one global model and ignore the regional distinction.
- D) Use External Functions keyed by region.

**Q47.** For 10M predictions/day with no hot-path latency SLA, cost analysis typically
favors:
- A) An always-on SPCS service with large replicas.
- B) A warehouse UDF called from a Dynamic Table or scheduled Task; the warehouse resumes, scores in a batch, and suspends.
- C) An External Function to an EKS cluster.
- D) Cortex only.

**Q48.** You decommission a model safely by:
- A) Dropping the model object immediately.
- B) First moving the `PROD` alias off it, waiting for monitoring to confirm no traffic, then dropping the model (or archiving).
- C) Pausing the Registry.
- D) Deleting the backing stage files directly.

**Q49.** A streaming inference pipeline on Snowflake typically uses:
- A) Snowpipe/Streaming → Stream → Task calling a model UDF → predictions table.
- B) Spark Streaming on an external cluster.
- C) Pub/Sub only.
- D) Dynamic Tables cannot handle streams; only batch is possible.

---

## Section 5 — Monitoring & Observability (9 questions)

**Q50.** The PSI formula is:
- A) `Σ |p_cur - p_base|`
- B) `Σ (p_cur - p_base) * ln(p_cur / p_base)` over bins
- C) `max |CDF_cur(x) - CDF_base(x)|` over x
- D) `Σ (observed - expected)² / expected`

**Q51.** [MULTI] Which drift metrics does Snowflake Model Monitor compute by default?
(Select all.)
- A) Feature distribution PSI
- B) Prediction distribution PSI
- C) Null / missing rates per feature
- D) Semantic groundedness of LLM outputs

**Q52.** PSI = 0.33. What is the typical interpretation?
- A) No shift.
- B) Minor shift, keep monitoring.
- C) Major shift — investigate / retrain.
- D) PSI cannot exceed 0.25.

**Q53.** You have a continuous-valued feature and want a distribution-free drift test
without binning. Choose:
- A) PSI with 50 bins.
- B) Kolmogorov-Smirnov (KS) test on empirical CDFs.
- C) Chi-squared test with manual bins.
- D) F-test.

**Q54.** Ground truth labels arrive 30 days late. Which design choice allows Model
Monitor to compute performance metrics correctly?
- A) Pre-populate the actuals with NULLs; backfill via MERGE as labels arrive; Monitor refresh picks them up on next cycle.
- B) Delete predictions older than 30 days and re-score.
- C) Set `actual_col=NULL` statically; performance metrics will not be computed.
- D) Only use PSI, never performance metrics for delayed-label models.

**Q55.** [MULTI] Which actions can an `ALERT` take when it fires? (Select all.)
- A) `CALL` a stored procedure (e.g., retraining).
- B) INSERT a row into a notification table.
- C) Send a Slack message via External Access Integration.
- D) Automatically overwrite the model version in Registry.

**Q56.** Chi-squared drift test is appropriate when:
- A) The feature is a numeric continuous variable.
- B) The feature is categorical with expected vs observed counts, and each cell has expected count ≥ 5.
- C) Only when `n > 100,000`.
- D) It's always preferable to PSI.

**Q57.** Cortex AI Observability is best for:
- A) Tabular model drift.
- B) LLM-specific evaluation — groundedness, faithfulness, answer relevance, context relevance via LLM-as-judge.
- C) Replacing Model Monitor for all models.
- D) Monitoring SPCS compute pool utilization.

**Q58.** How is a Snowflake Event Table typically used in ML pipelines?
- A) It replaces the Model Registry.
- B) It receives structured logs/metrics/traces from UDFs, SPROCs, and services for debugging and operational observability.
- C) It is the backing table for a Model Monitor.
- D) It cannot be queried directly.

---

## Section 6 — MLOps Automation & Governance (7 questions)

**Q59.** The command to sync a Git repository with Snowflake after setup is:
- A) `CREATE GIT REPOSITORY SYNC`
- B) `ALTER GIT REPOSITORY repo_name FETCH`
- C) `GIT PULL repo_name`
- D) `MERGE INTO repo_name`

**Q60.** [MULTI] Snowflake-native MLOps automation pillars include which of the following?
(Select all.)
- A) Source control via Git Repository object
- B) CI/CD via Snowflake CLI (`snow`) + GitHub Actions/Jenkins
- C) Reproducibility via Model Registry versions and Dataset versions
- D) Governance via RBAC, Object Tags, Masking Policies
- E) Observability via Model Monitor + Event Table + QUERY_HISTORY

**Q61.** Which Snowflake view attributes credits consumed by workload tag?
- A) `ACCOUNT_USAGE.LOGIN_HISTORY`
- B) `ACCOUNT_USAGE.QUERY_HISTORY` + `QUERY_TAG` column (and `WAREHOUSE_METERING_HISTORY` for credits).
- C) `INFORMATION_SCHEMA.TABLES`
- D) `ACCOUNT_USAGE.SESSIONS`

**Q62.** A proper model approval workflow should:
- A) Allow any data scientist to flip the `PROD` alias.
- B) Use an Object Tag (e.g., `MODEL_STATE`) with RBAC restricting who can set it to APPROVED, plus a promotion SPROC that checks the tag before flipping aliases.
- C) Rely on verbal sign-off only.
- D) Use git branch protection rules alone, with no in-Snowflake gate.

**Q63.** Which view captures which tables/views were read by each query (for lineage)?
- A) `ACCOUNT_USAGE.ACCESS_HISTORY`
- B) `ACCOUNT_USAGE.LOGIN_HISTORY`
- C) `INFORMATION_SCHEMA.TABLE_PRIVILEGES`
- D) `ACCOUNT_USAGE.STAGES`

**Q64.** You rotate an API key stored as a Snowflake Secret used by a running SPCS
service. What's required for the new key to take effect?
- A) Drop and recreate the service.
- B) `ALTER SECRET SET SECRET_STRING = '...'`; the service picks up the new value on next read (no restart required if the service re-fetches per call).
- C) Modify the compute pool manually.
- D) Rotate the master encryption key.

**Q65.** Dev → stage → prod promotion typically relies on which pattern?
- A) Single Registry schema with no separation; names like `MODEL_DEV`, `MODEL_PROD`.
- B) Separate Registry schemas (e.g., `REG_DEV`, `REG_STG`, `REG_PROD`); CI promotes by re-logging / cross-schema reference, with git-tagged DDL controlling the handoff.
- C) Promotion is unnecessary; models deploy directly to prod.
- D) Only use external MLflow tracking for promotion.

---

---

# Answer Key & Explanations

> Score: 1 pt per correct single-answer, 1 pt per MULTI (all-or-nothing).
> 65 total points. Pass ≥ 49/65 (~75%).

**Q1 — B.** Snowpark ML Modeling API pushes training down to the warehouse as a
Python stored procedure. The client only orchestrates. See `01_*` Q1, Q3. **A** is
wrong — Snowpark ML never pulls full training tables to the client. **C** happens only
when you use `jobs.remote` or SPCS explicitly. **D** describes Cortex, not Modeling API.

**Q2 — C.** Fine-tuning a 7B LLM needs GPU, custom CUDA, and persistent runtime —
exactly SPCS/ML Jobs territory. See `01_*` Q3, `03_*` Q14-Q15. **A/B** are CPU
warehouses; even Snowpark-optimized ones have no GPU. **D** is wrong — you can bring
your own LoRA via `@remote`.

**Q3 — B.** Batch at scale, no tight SLA, Dynamic Table is the native Snowflake
pattern. See `01_*` Q6, `04_*` Q3. **A** wastes money (always-on SPCS). **C** adds
unneeded external cost. **D** is user-driven, wrong workload.

**Q4 — B.** The dominant defense is packaging preprocessing *inside* the registered
Pipeline so the same transforms run at serve time. See `01_*` Q5, `03_*` Q3. **A** is
backwards. **C** is false — DTs ensure feature consistency between pipelines. **D** is
false — Feature Store serves both offline and online.

**Q5 — C.** Sub-100ms, 24/7, mobile app → SPCS service with ingress. See `01_*` Q4,
`04_*` Q1, Q5-Q6. Warehouse UDF (A) adds resume latency. Task (B) is batch. Snowpipe
(D) is ingestion, not inference.

**Q6 — A, C, D, E.** All real Snowpark ML paths. **B** is wrong — Snowpark ML does
not have `modeling.sklearn.*` imports; sklearn estimators are logged via the Registry
as external models. (See `03_*` Q1-Q2.)

**Q7 — B.** Dynamic Table with `TARGET_LAG='15 minutes'` is the native, incremental
pattern. See `01_*` Q6, Q19. **A/C** work but are more code and prone to bugs. **D** is
irrelevant.

**Q8 — B.** Standard tasks + foundation models → Cortex. See `01_*` Q10, `03_*` Q8.
**A** is overbroad. **C/D** require Registry (custom weights, deterministic env).

**Q9 — B.** `DOWNSTREAM` propagates the staleness contract from end-of-chain backward.
See `01_*` Q6. **A** is immediate refresh (not downstream). **C** fixed cron is wrong.
**D** false.

**Q10 — B.** `Dataset` is the exact abstraction. See `01_*` Q8. **A** re-executes
SQL per epoch. **C** moves data out of Snowflake unnecessarily. **D** is unrelated.

**Q11 — B.** Internal UI, Snowflake-auth, SQL-backed. See `01_*` Q9. **A** needs
external hosting. **C** too, plus always-on. **D** irrelevant.

**Q12 — B.** Snowpark-optimized = more memory per node. See `01_*` Q11. No GPU (A).
Not auto-SPCS (C). Not required (D).

**Q13 — B.** Entity, FeatureView, Dataset. See `02_*` Q2.

**Q14 — B.** Static FV — materialized once, no auto-refresh. See `02_*` Q3.
**A** is false; `refresh_freq` is optional. **C** no default DT if not set. **D** false.

**Q15 — B.** This is the PIT join. See `02_*` Q4, Q8. **A** causes leakage.
**C/D** are not PIT mechanisms.

**Q16 — A, B, D.** All leak. **A** is post-event mutation. **B** omits temporal filter.
**D** no time pin. **C** is correct behavior, not leakage.

**Q17 — B.** Create a new FV version. See `02_*` Q6. In-place mutation (A) breaks
reproducibility. **C/D** are wrong scopes.

**Q18 — B.** Query the backing DT by entity key. See `02_*` Q5, Q18. **A** is not
Snowflake-native. **C** is false — `MODEL!PREDICT` doesn't auto-pull features.
**D** is unnecessary.

**Q19 — B.** Dataset snapshots + REFRESH options. See `02_*` Q14. **A** is false.
**C/D** are wrong/invalid.

**Q20 — B.** Standard RBAC + masking + row policies all apply. See `02_*` Q24. Others
are false.

**Q21 — B.** Compute embeddings in SQL and store as a feature column. See `02_*` Q19.
**A/C/D** are wrong.

**Q22 — A, B, D.** Tags drive discovery (A), policies (B), approvals (D). **C** is
not what tags do — thresholds live in alerts/policies, not tags.

**Q23 — B.** Standard migration path. See `02_*` Q20. **A/C/D** are false.

**Q24 — B.** All six steps. See `03_*` Q6. **A** is false (artifact is included).
**C** false (signature inferred). **D** describes an older pattern.

**Q25 — B.** `sample_input_data` is mandatory for non-Snowpark-ML models. See `03_*`
Q8. Others are helpful but not required for sklearn specifically.

**Q26 — B.** Versions are immutable. See `03_*` Q10.

**Q27 — B.** Alias = named version pointer. See `03_*` Q11.

**Q28 — B.** Missing deps at log time → serving UDF lacks the library. See `03_*`
Q6, Q13, Q17. Others don't cause ModuleNotFoundError.

**Q29 — A, B, C, D.** All four are natively supported. **E** needs `CustomModel`.
See `03_*` Q8-Q9.

**Q30 — B.** CustomModel is the escape hatch for RAG + custom logic. See `03_*` Q9.

**Q31 — A, B, D.** Native (A, B); Optuna via `@remote` is a common pattern (D).
**C** is not a Snowpark ML class. See `03_*` Q4.

**Q32 — B.** GPU + custom deps → `@remote` on GPU compute pool. See `03_*` Q14-Q15.

**Q33 — B.** Alias flip. See `03_*` Q11, `04_*` Q16.

**Q34 — D.** Warehouse size affects speed/cost, not reproducibility. See `03_*` Q22.

**Q35 — B.** Last step must be an estimator. See `03_*` Q3.

**Q36 — B.** Metrics are mutable via `set_metric`. See `03_*` Q12.

**Q37 — B.** The two Snowflake-native paths. See `04_*` Q1.

**Q38 — B.** `service_compute_pool` is required. See `04_*` Q5. **A/C** aren't
service-level args. **D** is inferred from the model.

**Q39 — B.** Public HTTPS endpoint with Snowflake OAuth. See `04_*` Q5-Q6.

**Q40 — B.** Vectorized = batched pandas inputs → amortized inference. See `04_*`
Q4, Q15.

**Q41 — B.** Deterministic hash routing + version logging. See `04_*` Q16.

**Q42 — B.** Custom 7B → GPU SPCS with ingress. See `04_*` Q5, Q23. If the model
is actually in Cortex's catalog, C would be more cost-efficient; the question's wording
("a 7B-param model") implies custom.

**Q43 — A, B, C, D.** All four work. See `04_*` Q6-Q8. External apps use the HTTPS
endpoint (A, B); internal UDF (C); External Function proxy (D).

**Q44 — A.** Max rows per request → throughput/latency tuning. See `04_*` Q5, Q15.

**Q45 — B.** Scale out replicas and keep hot steady state. See `04_*` Q10.

**Q46 — B.** Register per-region models and route in SQL. See `04_*` Q13.

**Q47 — B.** Low-SLA batch → warehouse UDF. See `04_*` Q17.

**Q48 — B.** Alias off, drain, confirm, drop. See `04_*` Q24, `06_*` Q24.

**Q49 — A.** Streams+Tasks on top of streaming ingest. See `04_*` Q19.

**Q50 — B.** PSI formula. See `05_*` Q2, cheatsheet.

**Q51 — A, B, C.** Native monitors. **D** is Cortex AI Observability, not basic
Model Monitor. See `05_*` Q1, Q6.

**Q52 — C.** 0.33 ≥ 0.25 → major shift. See `05_*` Q2.

**Q53 — B.** KS test is distribution-free, needs no bins. See `05_*` Q3.

**Q54 — A.** Backfill actuals via MERGE. See `05_*` Q17.

**Q55 — A, B, C.** All valid alert actions. **D** is incorrect — alerts can't
overwrite versions (versions are immutable; promotion needs explicit SPROC). See
`05_*` Q10-Q11.

**Q56 — B.** Chi² needs categorical + sufficient expected counts. See `05_*` Q3.

**Q57 — B.** LLM-specific evaluation via Cortex AI Observability. See `05_*` Q6, Q14.

**Q58 — B.** Event Tables = structured telemetry. See `05_*` Q12.

**Q59 — B.** `ALTER GIT REPOSITORY ... FETCH`. See `06_*` Q2.

**Q60 — A, B, C, D, E.** All five pillars. See `06_*` Q1.

**Q61 — B.** QUERY_HISTORY + QUERY_TAG + warehouse metering. See `06_*` Q8.

**Q62 — B.** Tag-based gating + RBAC + SPROC. See `06_*` Q5.

**Q63 — A.** ACCESS_HISTORY for lineage. See `06_*` Q7.

**Q64 — B.** `ALTER SECRET` and re-read. See `06_*` Q16.

**Q65 — B.** Separate schemas + CI-driven promotion. See `06_*` Q10.

---

## Scoring Guide

| Score | Readiness |
|---|---|
| 55-65 | Exam-ready; schedule the test. |
| 49-54 | Pass-zone; review wrong answers, then retest with a shuffle. |
| 40-48 | Marginal; re-read `_QA.md` files for weak domains and retake in a week. |
| < 40 | Not ready; return to the modules — focus on the 2-3 weakest domains. |

Weak-domain diagnostic: tally wrong answers per section. More than 25% wrong in any
section → re-read that `_QA.md` end-to-end before re-attempting.
