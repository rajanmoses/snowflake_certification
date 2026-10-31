# MLE-C01 Flashcards

> **How to use:** Cover answers, read question, answer out loud, then uncheck.
> Each card is intentionally atomic — one fact, one answer. Harder cards marked [HARD].
> Front-loaded: Snowpark ML API surface, Feature Store, Registry, Monitor, SPCS vs
> warehouse decisions, PIT joins, drift metrics.

---

## A. Snowpark ML API Surface (Cards 1-20)

**[1]** Which Python package provides Snowpark ML, and what package is it layered on?
> `snowflake-ml-python`, layered on `snowflake-snowpark-python`.

**[2]** Import path for the Model Registry class?
> `from snowflake.ml.registry import Registry`

**[3]** Import path for the Feature Store class?
> `from snowflake.ml.feature_store import FeatureStore, Entity, FeatureView`

**[4]** Import path for Snowpark ML's XGBoost classifier?
> `from snowflake.ml.modeling.xgboost import XGBClassifier`

**[5]** Import path for a scikit-style preprocessing scaler in Snowpark ML?
> `from snowflake.ml.modeling.preprocessing import StandardScaler`

**[6]** Import path to assemble multi-step ML pipelines in Snowpark ML?
> `from snowflake.ml.modeling.pipeline import Pipeline`

**[7]** Which decorator turns a Python function into a remote SPCS ML job?
> `from snowflake.ml.jobs import remote` → `@remote(compute_pool=..., stage_name=..., pip_requirements=[...])`

**[8]** Import path for materialized ML datasets?
> `from snowflake.ml.dataset import DatasetFactory` (and `Dataset`).

**[9]** Import path for a custom-model abstraction (ensembles, business logic)?
> `from snowflake.ml.model import custom_model` → `custom_model.CustomModel` + `@custom_model.inference_api`.

**[10]** [HARD] Three required args to `Registry(...)` constructor?
> `session`, `database_name`, `schema_name`.

**[11]** [HARD] Call to log a trained Snowpark ML model with explicit version and metrics?
> `reg.log_model(model, model_name="CHURN", version_name="V1", metrics={"roc_auc":0.87}, conda_dependencies=["xgboost==1.7.*"], sample_input_data=df.limit(10))`

**[12]** How do you retrieve a model version object from the Registry?
> `reg.get_model("CHURN").version("V1")` (or `.default` for the DEFAULT alias).

**[13]** Which method deploys a model version to SPCS as a service?
> `mv.create_service(service_name=..., service_compute_pool=..., image_repo=..., ingress_enabled=True|False, min_instances=, max_instances=, gpu_requests=)`

**[14]** Which method creates a Model Monitor on a registered version?
> `mv.create_monitor(monitor_name=..., source_table=..., prediction_col=..., actual_col=..., timestamp_col=..., refresh_frequency=...)`

**[15]** What is the difference between `.fit()` returning in Snowpark ML vs sklearn?
> Snowpark ML `.fit()` runs training as a stored procedure on the warehouse and returns a Snowpark-ML model object (not a native xgboost `Booster` or sklearn estimator).

**[16]** Convert a Snowpark ML XGB model to the native xgboost object?
> `model.to_xgboost()` (similarly `.to_sklearn()`, `.to_lightgbm()`).

**[17]** How do you call a Registry-logged model as a UDF in pure SQL?
> `SELECT ML_DB.REG.CHURN!PREDICT(OBJECT_CONSTRUCT('AGE', AGE, 'INCOME', INCOME))`

**[18]** Where does Snowpark ML Modeling training run by default — client, warehouse, or SPCS?
> **Warehouse** (as a Python stored procedure). SPCS is opt-in via `jobs.remote` or explicit services.

**[19]** [HARD] Why does Snowpark ML push training to the warehouse?
> Data stays in place. Pulling the training table to a client would be slow and costly; SPROC pushdown keeps the full table in Snowflake.

**[20]** What is the Snowpark ML equivalent of sklearn's `GridSearchCV`?
> `from snowflake.ml.modeling.model_selection import GridSearchCV` (and `RandomizedSearchCV`), with the same `.fit()`-on-DataFrame interface.

---

## B. Feature Store (Cards 21-38)

**[21]** Three primary Feature Store object types?
> **Entity**, **FeatureView**, **Dataset**.

**[22]** What is an Entity in Snowflake Feature Store?
> A named grouping of join keys (e.g., `CUSTOMER` with `join_keys=["CUSTOMER_ID"]`). Defines *what* joins to what.

**[23]** What is a FeatureView?
> A named, versioned collection of features bound to one or more entities, defined by a Snowpark DataFrame expression. Optional timestamp column enables PIT joins.

**[24]** What is a Dataset in the Feature Store context?
> A materialized, immutable, versioned training set produced by joining a spine DF (keys + timestamp + labels) with one or more feature views.

**[25]** Difference between managed and static FeatureView?
> **Managed** = `refresh_freq` set → backed by a Dynamic Table, auto-refreshes. **Static** = `refresh_freq=None` → materialized once; must be refreshed manually or dropped/recreated.

**[26]** Call to register an Entity in the store?
> `fs.register_entity(Entity(name="CUSTOMER", join_keys=["CUSTOMER_ID"]))`

**[27]** Call to register a FeatureView with a managed refresh?
> `fs.register_feature_view(fv, version="V1")` where `fv = FeatureView(name=..., entities=[...], feature_df=..., refresh_freq="1 hour", timestamp_col="EVENT_TS")`

**[28]** How does Snowflake Feature Store back managed feature views under the hood?
> Dynamic Tables with the configured `TARGET_LAG`.

**[29]** [HARD] What is a point-in-time (PIT) join?
> For each spine row at time T, pick the latest feature value with timestamp ≤ T. Prevents future data leaking into training labels.

**[30]** Which Feature Store call generates a training set with PIT semantics?
> `fs.generate_dataset(spine_df=..., spine_timestamp_col="EVENT_TS", spine_label_cols=["CHURNED"], features=[fv1, fv2], name="CHURN_V1")`

**[31]** Why is PIT join essential for ML correctness?
> A naive JOIN picks whichever feature row the engine selects (often the latest), which may contain post-event state. That leaks future information and inflates training accuracy; production craters.

**[32]** When do you create a new FeatureView **version** vs modify in place?
> New version whenever you add/change/remove features or change computation semantics. Modify in-place only for cosmetic metadata (description, tags).

**[33]** Can two FeatureView versions share the same underlying Dynamic Table?
> No — each version creates (or references) its own DT. Versions are isolated snapshots of the definition.

**[34]** [HARD] How do you serve a feature view online for real-time scoring?
> Query the FV's underlying DT by join key: `fs.retrieve_feature_values(spine_df, [fv])` or direct SQL `SELECT ... FROM FS.FV_CUSTOMER_30D WHERE CUSTOMER_ID=:id`. Managed FV with low `refresh_freq` keeps data fresh.

**[35]** What timestamp field does a FeatureView use for PIT join eligibility?
> The `timestamp_col` passed to `FeatureView(..., timestamp_col="EVENT_TS")`. Without it, FV supports only non-temporal joins.

**[36]** How to prevent target leakage inside a FeatureView definition?
> (1) Use the FV `timestamp_col`; (2) ensure the SQL filters on events ≤ `timestamp_col` only; (3) do not reference labels or label-correlated post-event fields.

**[37]** Can you share FeatureViews across databases or accounts?
> Across **databases** — yes, by registering the FeatureStore in the target DB. Across **accounts** — via Snowflake Data Sharing on the backing tables, but the FV catalog itself is per-account.

**[38]** How do you backfill a new FeatureView over historical dates?
> Register with managed refresh, then issue `ALTER DYNAMIC TABLE ... REFRESH` or set `refresh_mode='INCREMENTAL'` with a historical scope. Alternatively, materialize via Dataset at the required `as_of` dates.

---

## C. Model Registry (Cards 39-55)

**[39]** What is the top-level object managing versions in the Registry?
> **Model** (a named, logical model). It contains versions, aliases, tags.

**[40]** What is a "version" in the Registry?
> A specific logged artifact — e.g., `CHURN:V1`, with its own weights, dependencies, signature, metrics.

**[41]** What is an **alias**?
> A named pointer to a version. Common conventions: `DEFAULT` (auto, latest logged unless overridden), `PROD`, `STAGING`, `SHADOW`.

**[42]** Can you overwrite an existing model version?
> **No** — versions are immutable. To change, log a new version. (You can delete + relog, but doing so breaks lineage.)

**[43]** What does the `DEFAULT` alias point to if you don't set it manually?
> The most recently logged version.

**[44]** Six things `log_model()` captures/does?
> 1) Serialize model artifact to stage, 2) Infer signature from `sample_input_data`, 3) Capture `conda_dependencies` / `pip_requirements`, 4) Register metadata (version, metrics, tags, comment), 5) Deploy a warehouse UDF, 6) Prepare for SPCS deployment.

**[45]** [HARD] What's the minimum required to log a non-Snowpark-ML model (e.g., sklearn)?
> You must pass `sample_input_data` so Snowflake can infer the model signature.

**[46]** How are model signatures used at inference?
> They validate the input DataFrame schema and infer output types; mismatches error at deploy or call time rather than silently producing bad predictions.

**[47]** Which Registry types are supported natively (no CustomModel wrapper)?
> Snowpark ML Modeling API, scikit-learn, XGBoost, LightGBM, CatBoost, PyTorch, TensorFlow, HuggingFace Transformers, MLflow models.

**[48]** When do you reach for `CustomModel`?
> Non-standard inference logic: ensembles across versions, external API calls, RAG chains, custom pre/post-processing, business rules on top of a model.

**[49]** Which decorator marks a user method as the inference entry point in CustomModel?
> `@custom_model.inference_api`

**[50]** Difference between tags and metrics on a model version?
> **Metrics** = numeric (e.g., `roc_auc=0.87`), free-form, for comparison.
> **Tags** = governed, pre-registered key/value strings tied to Object Tag objects, used for policy/approval workflow.

**[51]** Can metrics be updated after logging?
> **Yes** via `mv.set_metric(name, value)`. Metrics are mutable; artifact + signature are not.

**[52]** How do you add a comment / description to a version?
> `mv.comment = "..."` (property) or at log time via `log_model(..., comment=...)`.

**[53]** [HARD] How do you promote a model version to PROD atomically?
> `reg.get_model("CHURN").version("V2").set_alias("PROD")` — alias move is atomic from the serving side. Downstream SQL that references `CHURN:PROD` or `CHURN!PREDICT` on the default/aliased version flips instantly.

**[54]** [HARD] How is `MODEL!PREDICT` (bang syntax) different from calling a UDF?
> `MODEL!PREDICT` invokes the model's registered **method** on the DEFAULT version. It's shorthand sugar over the versioned UDF. You can also call specific-version UDFs created implicitly per version.

**[55]** Why pin `conda_dependencies` at log time?
> Serving must match training; pickled models may fail to deserialize across library versions. Pinning (`xgboost==1.7.5`) prevents silent breakage.

---

## D. SPCS vs Warehouse (Cards 56-70)

**[56]** Default compute for a Registry-logged model UDF?
> **Warehouse** (serverless, autoscale-via-size, cold-start seconds).

**[57]** When to deploy to SPCS via `create_service`?
> Real-time sub-100ms latency, high QPS, GPU inference, custom OS libraries, model > ~500MB, persistent-container runtime needs.

**[58]** When to train on SPCS or ML Jobs instead of warehouse?
> GPU training (PyTorch, TF, XGBoost-GPU), multi-node distributed training, custom CUDA/OS deps, long-running jobs (hours to days), Ray/Dask clusters.

**[59]** Instance family keyword for GPU compute pools?
> `INSTANCE_FAMILY = GPU_NV_S` (and `GPU_NV_M`, `GPU_NV_L` depending on size).

**[60]** Which has lower cold-start latency: warehouse or SPCS service?
> **SPCS service** (always-on container). Warehouse resume adds seconds.

**[61]** Which is cheaper for a nightly batch of 100M predictions: warehouse UDF or SPCS?
> **Warehouse UDF** — serverless, scales down to zero, amortizes one columnar scan over all rows.

**[62]** Which is cheaper for a 24/7 real-time API serving 50 QPS at 50ms SLA: warehouse or SPCS?
> **SPCS** — warehouse cold start + per-call overhead would dominate; SPCS pays flat for always-on compute pool.

**[63]** [HARD] What's the role of an **image repository** in SPCS deployment?
> `CREATE IMAGE REPOSITORY` holds the OCI images built from the model + dependencies. Images are cached across versions when deps are unchanged.

**[64]** What parameters control SPCS service autoscaling in `create_service()`?
> `min_instances`, `max_instances` — Snowflake scales replicas between them based on load (CPU, request rate).

**[65]** What is `ingress_enabled=True` on `create_service()`?
> Exposes a public HTTPS endpoint through Snowflake-managed Ingress (auth via Snowflake OAuth). `False` keeps the service in-DB only (callable via UDF).

**[66]** [HARD] What is `max_batch_rows` on an SPCS service?
> Max rows the service accepts per prediction request — tune to balance latency vs throughput, especially for GPU batching.

**[67]** Can a Dynamic Table call a SPCS service for inference?
> Yes — the Service also registers an in-DB UDF, callable in DT SELECTs (subject to standard UDF restrictions).

**[68]** Difference between ML Jobs API and raw SPCS Services?
> **ML Jobs** = higher-level abstraction for *ad-hoc* training/jobs (`@remote` decorator) on compute pools; auto-manages service lifecycle. **SPCS Services** = long-running services with full control.

**[69]** What is a Snowpark-optimized warehouse and when to use it for ML?
> A warehouse variant with ~16x more memory per node. Use when single-node training/inference exceeds standard warehouse memory (large pandas, big XGBoost in-memory, Spark-ML pushdown).

**[70]** Can a warehouse UDF import a model artifact larger than ~500MB?
> Not reliably — warehouse UDF has artifact-size limits. For large (LLM, big TF) models, deploy via SPCS service.

---

## E. Dynamic Tables & Inference Patterns (Cards 71-82)

**[71]** What is `TARGET_LAG` in a Dynamic Table?
> Upper bound on data staleness. The engine schedules refreshes to keep the DT within `TARGET_LAG` of the source. Not a cron schedule.

**[72]** `TARGET_LAG = 'DOWNSTREAM'` meaning?
> Refresh is deferred until a downstream DT needs the data. Useful in multi-stage DAGs so the end-of-chain lag drives the whole chain.

**[73]** Can `MODEL!PREDICT(...)` be used in a Dynamic Table SELECT?
> **Yes.** Registry UDFs are pure; DTs allow them, enabling continuously-refreshed predictions.

**[74]** Can an external-access UDF be used in a Dynamic Table?
> **No.** DTs require deterministic / pure UDFs. External-access or mutable UDFs are prohibited.

**[75]** When to use a scheduled Task over a Dynamic Table for inference?
> When you need imperative orchestration (branching, error handling, side effects like notifications) or when the inference is a one-off backfill rather than continuous.

**[76]** Vectorized Python UDF — what is it and why?
> A UDF declared with `@pandas_udf` / `@vectorized` that receives a pandas DataFrame batch instead of row-by-row. For model inference it amortizes model-load + inference-setup across many rows → 10-100x throughput.

**[77]** What's the typical vectorized UDF batch size?
> Snowflake decides, usually ~1000-50000 rows per batch depending on data types and memory.

**[78]** [HARD] A/B testing pattern on Snowflake?
> Deploy both versions. Route via a Dynamic Table or SPROC that picks the version by a stable hash of a key (e.g., `MOD(ABS(HASH(CUSTOMER_ID)), 100) < 10` → B, else A). Log the version per row for later evaluation.

**[79]** Shadow deployment pattern?
> Serve PROD normally; also score every request with a shadow version in parallel, log both. No user impact; compare offline to decide promotion.

**[80]** Streaming inference pattern on Snowflake?
> Snowpipe/Streaming ingests events → stream object captures changes → Task scores via UDF → write to predictions table. Micro-batch between batch and real-time.

**[81]** Which Snowflake product serves per-token-billed LLM inference without custom ops?
> **Snowflake Cortex** (`SNOWFLAKE.CORTEX.COMPLETE`, `EMBED_TEXT_768`, `SUMMARIZE`, `TRANSLATE`, `SENTIMENT`, `EXTRACT_ANSWER`, etc.).

**[82]** When to register a custom LLM in the Registry vs use Cortex?
> Register when: fine-tuned/custom weights, non-catalog HF model, deterministic env needs, custom governance. Use Cortex when: general-purpose task, foundation model, serverless preferred.

---

## F. Drift Metrics & Monitoring (Cards 83-96)

**[83]** PSI formula?
> `PSI = Σ (p_cur - p_base) * ln(p_cur / p_base)` over each bin, where `p_cur`, `p_base` are proportions.

**[84]** PSI interpretation thresholds (industry defaults)?
> **< 0.1** — no shift. **0.1-0.25** — minor shift, monitor. **≥ 0.25** — major shift, investigate / retrain.

**[85]** How many bins for PSI by convention?
> 10 bins (equal-frequency or equal-width) for numeric; one bin per category for categorical.

**[86]** [HARD] What test is best for comparing two continuous distributions, non-binned?
> **Kolmogorov-Smirnov (KS) test** — compares empirical CDFs; statistic = max absolute CDF gap.

**[87]** KS statistic interpretation?
> Larger KS = larger distribution gap. P-value from the KS table tells you significance. For drift, often threshold `KS > 0.1` with `p < 0.05`.

**[88]** [HARD] Which drift test for categorical features with expected vs observed counts?
> **Chi-squared (χ²) test.** `χ² = Σ (observed - expected)² / expected` over categories.

**[89]** When does chi² break down?
> With very low expected counts (< 5 per cell). Use Fisher's exact or Monte-Carlo variants in that regime.

**[90]** What does Snowflake Model Monitor compute automatically?
> Feature distribution drift (PSI/KS), prediction distribution drift, null/missing rates, performance metrics (AUC, F1, RMSE etc. when `actual_col` supplied), scheduled refresh.

**[91]** Required args to `create_monitor()`?
> `monitor_name`, `source_table`, `prediction_col`, `timestamp_col`, `refresh_frequency`. `actual_col` optional (needed for performance metrics). `baseline` optional (defaults to training set if logged).

**[92]** How are "actuals" / ground truth joined to predictions in Model Monitor?
> You update the `source_table` with the `actual_col` (often via a late-arriving-labels UPDATE/MERGE). Monitor re-runs performance metrics on refresh.

**[93]** [HARD] How do you handle delayed ground truth (e.g., 30-day churn label)?
> Design the predictions table with a nullable `ACTUAL` column. A separate pipeline (Task/DT) back-fills labels when they arrive. Monitor refresh picks them up next cycle. Alerts on performance metrics must tolerate NULLs or window to only matured labels.

**[94]** What is Cortex AI Observability?
> Snowflake's LLM-specific observability — traces prompts/responses, computes groundedness, faithfulness, answer relevance, context relevance using LLM-as-judge. Attaches to Cortex calls.

**[95]** What is an Event Table and its role for ML?
> `CREATE EVENT TABLE` — a structured table receiving logs/traces/metrics from UDFs, stored procedures, and services. Used for debugging ML pipelines and tracing calls.

**[96]** How does an Alert object fire a retraining pipeline?
> `CREATE ALERT` with a SQL `IF` condition (e.g., `PSI > 0.25`) and an action — typically a `CALL procedure()` that enqueues retraining. Scheduled via a `WAREHOUSE` + `SCHEDULE`.

---

## G. Governance & MLOps Automation (Cards 97-110)

**[97]** How does Snowflake Git integration work?
> `CREATE GIT REPOSITORY REPO_NAME ORIGIN = 'https://...' API_INTEGRATION = ...`. Then `ALTER GIT REPOSITORY REPO_NAME FETCH`. Branches/commits appear as stage paths (`@REPO_NAME/branches/main/path/file.sql`).

**[98]** `snow` CLI command to run SQL against your account?
> `snow sql -q "..."` for inline, `snow sql -f path/to/file.sql` for file.

**[99]** `snow` CLI command to deploy a Streamlit app?
> `snow streamlit deploy` from a project with `snowflake.yml`.

**[100]** `snow` CLI command to run a stored procedure?
> `snow sql -q "CALL ML_DB.PROC.FOO(1,2)"` (there is no dedicated `snow procedure call` yet; use `snow sql`).

**[101]** [HARD] How do you track model lineage from source data to predictions on Snowflake?
> `ACCOUNT_USAGE.ACCESS_HISTORY` captures tables accessed per query; combined with QUERY_TAG, Model Registry version metadata, and Dataset lineage, you can reconstruct source→feature→training→model→predictions.

**[102]** Role of `QUERY_TAG` for cost chargeback?
> Set per-session or per-statement; `ACCOUNT_USAGE.QUERY_HISTORY` groups credit consumption by tag. Attach to every ML workload: `ALTER SESSION SET QUERY_TAG='churn-training-v2'`.

**[103]** Which ACCOUNT_USAGE view attributes credits to warehouses?
> `WAREHOUSE_METERING_HISTORY` for warehouse credits; `SERVICES_METERING_HISTORY` (or `METERING_DAILY_HISTORY`) for SPCS; combine with `QUERY_TAG` for workload-level chargeback.

**[104]** How do you implement a model approval workflow in Snowflake?
> (1) Pre-register an Object Tag `MODEL_STATE` with allowed values (DEV, STAGING, APPROVED, PROD). (2) RBAC: only approver role can set tag to APPROVED. (3) Promotion SPROC checks tag before `set_alias('PROD')`.

**[105]** [HARD] RBAC privileges needed to call a Registry model UDF?
> `USAGE` on database + schema + model, `IMPORTED PRIVILEGES`-equivalent for underlying stages. Not `OWNERSHIP`. Service calls additionally need `USAGE` on the service and compute pool.

**[106]** How do you handle PII in training data?
> (1) Masking policies on source tables, (2) Row Access Policies for tenancy, (3) Horizon Catalog classification tags surfaced to data scientists, (4) Only aggregate features used in FeatureView if raw PII is sensitive.

**[107]** What is an External Access Integration (EAI)?
> A Snowflake object granting a UDF/SPROC/Service outbound network access to specific endpoints (via Network Rules) and credentials (via Secrets). Required for any external API call from Snowflake.

**[108]** How do you rotate a secret used by a running SPCS service?
> `ALTER SECRET SET SECRET_STRING = '...'`. The service picks up the new value on next `GET` from Snowflake Secret API — no restart required if the service re-reads per call.

**[109]** [HARD] Dev → stage → prod promotion pattern?
> Separate Registry schemas: `REG_DEV`, `REG_STG`, `REG_PROD`. CI promotes a tested version by **re-logging** it in the next schema (or by cross-schema reference + alias). Database Change-Management-style with git-tagged commits controls the DDL + log calls.

**[110]** How do you ensure deterministic training across time?
> Pin: (1) `conda_dependencies` versions, (2) Dataset version (immutable snapshot), (3) random seeds in model + Pipeline, (4) git_commit tag on the model version, (5) a stored procedure wrapping training (same SQL, same code).

---

## H. Rapid-Fire Gotchas (Cards 111-120)

**[111]** `TARGET_LAG` on a DT — is it a cron?
> **No**, it's a staleness upper bound; the engine decides when to refresh.

**[112]** Can you overwrite a model version in Registry?
> **No** — versions are immutable; log a new version instead.

**[113]** DEFAULT alias behavior if you log V2 without manually setting an alias?
> DEFAULT auto-flips to V2 (latest logged). Pin DEFAULT explicitly if you don't want this.

**[114]** Does `fs.register_feature_view` create a Dynamic Table?
> Only if `refresh_freq` is set (managed FV). Otherwise it's static.

**[115]** Does a FeatureView auto-detect PIT eligibility?
> Only if `timestamp_col` is passed to the FeatureView definition.

**[116]** Can a Dynamic Table use a stored procedure?
> No — SPROCs are not callable in DT SELECTs (they're imperative). UDFs only.

**[117]** Can Cortex functions run in Dynamic Tables?
> **Yes** — Cortex SQL functions are deterministic UDFs and compose in DTs.

**[118]** Can you register an MLflow model directly?
> Yes — the Registry accepts MLflow Model format and packages it like native types.

**[119]** Where does `log_model` put the serialized artifact?
> An internal stage within the Registry schema (abstracted — you reference by model name + version).

**[120]** Do SPCS compute pools auto-suspend by default?
> **No** — you must configure `AUTO_SUSPEND_SECS`. A common cost pitfall is leaving pools running overnight.

---

**End of flashcards. Total: 120 cards.**

> **Review cadence:** Day 1 — read all; day 3 — redo weak domains; day 7 — all;
> day 14 — missed-only; day before exam — missed-only skim.
