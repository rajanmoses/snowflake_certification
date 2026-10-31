# GES-C01 Flashcards — Snowflake Specialty: Generative AI

Format: each card has one Q and one A, separated by `---`. Front-loaded by exam weight: Cortex function signatures, model catalog, embedding specs, Cortex Guard categories, semantic model YAML keys, then the long tail of RAG, Analyst, Document AI, Notebooks, patterns, and governance.

---

## Domain 1 — Cortex LLM Functions

### Q: What is the full signature of `SNOWFLAKE.CORTEX.COMPLETE`?
**A:** `COMPLETE(model VARCHAR, prompt_or_messages, options OBJECT) RETURNS VARCHAR | OBJECT`. `prompt_or_messages` is either a VARCHAR (returns bare string) or an ARRAY of `{role, content}` chat messages (returns an OBJECT with `choices`, `usage`).
---

### Q: Which three options keys most matter in COMPLETE?
**A:** `temperature` (0.0–1.0, default 0 → deterministic), `max_tokens` (default 4096, output only), and `response_format` ({type:'json', schema:{...}}) for grammar-constrained JSON decoding. Also `guardrails`, `top_p`, `tools`, `tool_choice`.
---

### Q: When does COMPLETE return an OBJECT instead of a VARCHAR?
**A:** Any messages-array prompt OR any options object makes it return an OBJECT with `choices[0].messages`, `usage.prompt_tokens`, `usage.completion_tokens`, `usage.total_tokens`.
---

### Q: What is the difference between COMPLETE and TRY_COMPLETE?
**A:** `TRY_COMPLETE` returns NULL on recoverable errors (rate limit, guardrail block, some invalid model issues) instead of aborting the statement. Use in batch pipelines. Permanent errors (missing model + CRI disabled) still raise.
---

### Q: What are the five task-specific Cortex helper functions?
**A:** `SUMMARIZE(text)`, `TRANSLATE(text, src_lang, tgt_lang)`, `SENTIMENT(text)`, `EXTRACT_ANSWER(text, question)`, `CLASSIFY_TEXT(text, [labels])`. No model argument — model is fixed internally.
---

### Q: What does SENTIMENT return and how is that different from CLASSIFY_TEXT?
**A:** SENTIMENT returns a FLOAT in [-1, 1]. CLASSIFY_TEXT returns a JSON object `{"label": "...", "confidence": ...}`. If you need a label, use CLASSIFY_TEXT, not SENTIMENT.
---

### Q: What is the return structure of EXTRACT_ANSWER?
**A:** An ARRAY of OBJECTs: `[{"answer":"...","score":0.87}]`. Always subscript `[0]:answer` for the scalar. It is extractive (returns a span from the source), not generative.
---

### Q: What are the two Cortex embedding function names and their output dims?
**A:** `EMBED_TEXT_768(model, text) → VECTOR(FLOAT, 768)` and `EMBED_TEXT_1024(model, text) → VECTOR(FLOAT, 1024)`.
---

### Q: Name the common 768-d embedding models in Cortex.
**A:** `snowflake-arctic-embed-m-v1.5` (English, retrieval-tuned, fast) and `e5-base-v2` (English baseline).
---

### Q: Name the common 1024-d embedding models in Cortex.
**A:** `snowflake-arctic-embed-l-v2.0` (multilingual flagship), `voyage-multilingual-2`, `nv-embed-qa-4` (QA-tuned).
---

### Q: Which Cortex vector distance functions exist and which is the default for embeddings?
**A:** `VECTOR_COSINE_SIMILARITY`, `VECTOR_L2_DISTANCE`, `VECTOR_INNER_PRODUCT`. Cosine is the default; higher = more similar. L2: lower = more similar.
---

### Q: What happens if you try to compute similarity between a 768-d and a 1024-d vector?
**A:** It errors out — dimensions must match. Changing embedding model after loading means re-embedding the entire corpus.
---

### Q: What model catalog "families" are available in Cortex?
**A:** Mistral (7b, large2), Llama 3.1 (8b, 70b, 405b), Llama 3.2 vision (11b-vision, 90b-vision), Snowflake Arctic, Reka (Core, Flash), Jamba (Instruct, 1.5-large), Claude via Cortex (some regions).
---

### Q: Which Cortex model has the longest context window and why?
**A:** `jamba-1.5-large` with ~256k tokens, thanks to its Mamba state-space / Transformer hybrid giving sub-quadratic long-context attention.
---

### Q: Which Cortex model is best for SQL and code generation?
**A:** `snowflake-arctic` — tuned on enterprise SQL and code; punches above its weight on structured reasoning.
---

### Q: Which Cortex models accept images?
**A:** `llama3.2-11b-vision` and `llama3.2-90b-vision`. Pass images via the messages API using `{"type":"image_url","image_url":{"url": BUILD_SCOPED_FILE_URL(@stage,'file.png')}}`.
---

### Q: What is the input-vs-output cost pattern for Cortex LLMs?
**A:** Each model has published credits-per-1M-tokens split by input vs output. Output rate is usually 1x–4x the input rate, with `llama3.1-405b` carrying an ~11x output premium. Decoding is sequential, so output tokens dominate cost and latency.
---

### Q: Which views show Cortex token spend?
**A:** `SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTIONS_USAGE_HISTORY` (per call) and `CORTEX_FUNCTIONS_QUERY_USAGE_HISTORY` (aggregated to query_id). For near-real-time use `INFORMATION_SCHEMA.CORTEX_FUNCTIONS_USAGE_HISTORY` table function; ACCOUNT_USAGE lags up to 3 hours.
---

### Q: How do you attribute Cortex cost to an app or team?
**A:** `ALTER SESSION SET QUERY_TAG = '{"app":"x","team":"y"}'` before calls, then join `CORTEX_FUNCTIONS_QUERY_USAGE_HISTORY` to `QUERY_HISTORY` on `query_id` and parse the tag.
---

### Q: What is Cortex Guard, and how do you turn it on?
**A:** A Llama-Guard-family classifier that screens LLM output post-generation for policy violations (hate, violence, self-harm, sexual content, criminal planning, CBRN). Enable with `{'guardrails': true}` in the COMPLETE options object.
---

### Q: What are the Cortex Guard safety categories?
**A:** Hate, violence, self-harm, sexual content, criminal planning (including weapons / drug synthesis), and CBRN (chemical/biological/radiological/nuclear).
---

### Q: Does Cortex Guard screen the user's prompt?
**A:** No — Guard screens only the model's output. For input screening, use `AI_CLASSIFY` with policy labels or a dedicated prompt-injection classifier before calling COMPLETE.
---

### Q: What is Cross-Region Inference (CRI) and how do you enable it?
**A:** CRI lets a Cortex function in region A route inference to a model hosted in region B when the model is not available in A. Enable with `ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION'` or a region class like `'AWS_US'`. Default is `DISABLED`.
---

### Q: What is PARSE_DOCUMENT?
**A:** `SNOWFLAKE.CORTEX.PARSE_DOCUMENT(@stage, 'file.pdf', {'mode':'LAYOUT'|'OCR'})` returns an OBJECT with `content` (text) and optionally `pages`, `metadata`. Use LAYOUT for PDFs with embedded text; OCR for scans. It feeds EMBED_TEXT and COMPLETE downstream.
---

### Q: What is the AISQL family?
**A:** Newer declarative LLM verbs: `AI_COMPLETE`, `AI_CLASSIFY`, `AI_FILTER` (BOOLEAN usable in WHERE), `AI_AGG`, `AI_SUMMARIZE_AGG`, `AI_SIMILARITY`. Supports multi-label classification, task descriptions, and relational-algebra integration.
---

### Q: How do you force Cortex COMPLETE to return valid JSON?
**A:** Use `response_format: {type:'json', schema: {...JSON schema...}}` with `temperature: 0`. The decoder constrains logits to tokens that keep the output a valid prefix of the schema — output is guaranteed parseable.
---

### Q: Which role grants access to Cortex functions?
**A:** The database role `SNOWFLAKE.CORTEX_USER` (grant membership to your roles). PUBLIC also has usage by default; revoke if you need lockdown. Fine-grained grants on individual functions are supported.
---

### Q: How do you stream a Cortex COMPLETE response?
**A:** Streaming is only via the Python API (`from snowflake.cortex import complete; complete(..., stream=True)` returns an iterator) or REST endpoint `/api/v2/cortex/inference:complete` with SSE. SQL cannot stream.
---

### Q: What are the input character caps on helper functions?
**A:** SUMMARIZE, TRANSLATE, SENTIMENT, EXTRACT_ANSWER, CLASSIFY_TEXT: ~100k characters. COMPLETE: full model context window (128k Llama 3.1, 256k Jamba). Embeddings: ~512 tokens (truncate silently).
---

### Q: What is Cortex Fine-Tuning and which base models support it?
**A:** A managed LoRA fine-tuning service producing a custom model name callable via COMPLETE. Supported bases include Mistral 7B, Mistral-large2, and Llama 3 variants (check current docs). Trigger with `SNOWFLAKE.CORTEX.FINETUNE('CREATE', model_name, base, training_sql, validation_sql)`.
---

### Q: How is a vision model called via COMPLETE?
**A:** Pass messages with a mixed content array: `[{"type":"text","text":"..."},{"type":"image_url","image_url":{"url": BUILD_SCOPED_FILE_URL(@stage,'img.png')}}]` as a user turn. Model is `llama3.2-11b-vision` or `llama3.2-90b-vision`.
---

## Domain 2 — Cortex Search & RAG

### Q: What is Cortex Search and how is retrieval scored?
**A:** Managed hybrid retrieval service: dense ANN (embeddings, cosine) + sparse BM25 (lexical) fused by Reciprocal Rank Fusion, with an optional cross-encoder reranker. Serves via SQL `SEARCH_PREVIEW`, Python SDK, or REST.
---

### Q: Give the Cortex Search service DDL skeleton.
**A:**
```sql
CREATE OR REPLACE CORTEX SEARCH SERVICE svc_name
  ON content_column
  ATTRIBUTES col1, col2
  WAREHOUSE = wh_name
  TARGET_LAG = '1 hour'
  EMBEDDING_MODEL = 'snowflake-arctic-embed-l-v2.0'
  AS (SELECT ... FROM source);
```
---

### Q: What does `TARGET_LAG` mean on a Cortex Search service?
**A:** The max freshness SLA for the index relative to the source. Shorter lag = more frequent refresh = more warehouse and embedding credit spend.
---

### Q: What is the difference between `ATTRIBUTES` and other selected columns on a search service?
**A:** `ATTRIBUTES` columns are indexed and usable in filters. Non-attribute columns are returned to clients but not filterable.
---

### Q: Which filter operators does Cortex Search support?
**A:** `@eq`, `@contains`, `@gte`, `@lte`, `@gt`, `@lt`, `@and`, `@or`, `@not`, `@primarykey`. Filters are JSON expressions over ATTRIBUTE columns only.
---

### Q: How do you query a Cortex Search service in SQL?
**A:**
```sql
SELECT SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
  'svc_name',
  OBJECT_CONSTRUCT(
    'query', 'user question',
    'columns', ARRAY_CONSTRUCT('title','body'),
    'filter', OBJECT_CONSTRUCT('@eq', OBJECT_CONSTRUCT('region','EMEA')),
    'limit', 5
  )::STRING
);
```
Returns JSON with `results[]` containing selected columns plus `_score`.
---

### Q: What is RRF and why is it used in hybrid retrieval?
**A:** Reciprocal Rank Fusion: for each candidate doc, score = Σ 1/(k + rank_i) across each retrieval pass. Combines dense and sparse rankings without needing calibrated score scales — only ranks matter.
---

### Q: Why does hybrid retrieval beat pure dense retrieval?
**A:** Dense embeddings excel on synonymy (`invoice` ≈ `bill`) but lose rare tokens like product SKUs (`WDGT-4198`). Sparse BM25 nails exact matches on IDs/codes. Combining them improves recall across both query types.
---

### Q: What text splitter does Snowflake provide and what are its args?
**A:** `SNOWFLAKE.CORTEX.SPLIT_TEXT_RECURSIVE_CHARACTER(text, format, chunk_size, overlap)`. `format` can be `'none'`, `'markdown'`, etc. Typical: chunk_size=1500 chars, overlap=200.
---

### Q: Typical recommended chunk size and overlap for prose RAG?
**A:** Chunk size 1000–1500 characters, overlap 150–200. For code: 400–600 with structural boundaries. Longer chunks dilute embeddings; shorter chunks fragment context.
---

### Q: Can you change the embedding model on an existing Cortex Search service?
**A:** No — `EMBEDDING_MODEL` is immutable. Changing it requires recreating the service (and re-embedding the whole corpus). Choose carefully at creation.
---

### Q: How does Cortex Search handle updates to source rows?
**A:** Managed task runs within `TARGET_LAG` using change tracking to detect inserts/updates/deletes. Only changed rows are re-embedded. Deleted rows are purged from the index.
---

### Q: How do row access policies interact with Cortex Search?
**A:** Refresh runs under the owner's privileges and indexes everything. Serving applies the caller's row access policies, so unauthorized rows are filtered from result sets. Vectors still exist in the index — governance is correct but not zero-trust indexed form.
---

### Q: What's the standard RAG app flow on Snowflake?
**A:** Query → embed query → Cortex Search for top-k chunks → assemble prompt with context + "answer only from context, cite [title]" → `SNOWFLAKE.CORTEX.COMPLETE('llama3.1-70b', msgs, {'temperature':0,'guardrails':true})` → render with sources.
---

### Q: How do you implement multi-turn RAG with follow-up questions?
**A:** Query rewriting: use a small COMPLETE call with history + follow-up to produce a standalone query, then retrieve on that. Follow-ups like "and overseas?" otherwise lack semantic content for the embedder.
---

### Q: What retrieval metrics should you measure?
**A:** Recall@k (did we find the right chunks?), Precision@k (how much noise?), MRR (position-weighted), NDCG@k (graded relevance). Build a golden set of (query, relevant_doc_ids) and track across changes.
---

### Q: When should you use document-level vs chunk-level retrieval?
**A:** Chunk-level: local-answer QA (warranty period). Document-level: cross-section synthesis (summarize entire contract). Hierarchical (top docs → top chunks within) is a cost-aware middle ground.
---

### Q: How do you call Cortex Search from Python?
**A:**
```python
from snowflake.core import Root
root = Root(session)
svc = root.databases["DB"].schemas["SCH"].cortex_search_services["svc"]
resp = svc.search(query="...", columns=[...], filter={...}, limit=5)
for r in resp.results: ...
```
---

### Q: What are the main cost dimensions of a Cortex Search deployment?
**A:** (1) Ingest: PARSE_DOCUMENT per page + EMBED_TEXT per token. (2) Refresh: warehouse credits + embedding tokens on changed rows. (3) Per-query search credits + query embedding + COMPLETE for generation. COMPLETE output tokens usually dominate.
---

## Domain 3 — Cortex Analyst

### Q: What is Cortex Analyst and how is it different from RAG?
**A:** A REST service that turns NL into SQL grounded by a semantic model (YAML). RAG retrieves unstructured text; Analyst generates SQL over structured tables. Analyst returns SQL; your app executes it.
---

### Q: What is the REST endpoint for Analyst?
**A:** `POST /api/v2/cortex/analyst/message` with payload including `messages`, `semantic_model_file` (path to YAML in a stage), optional `stream`.
---

### Q: Top-level sections of a semantic model YAML?
**A:** `name`, `description`, `custom_instructions`, `tables[]` (with `base_table`, `primary_key`, `dimensions`, `time_dimensions`, `measures`, `filters`), `relationships[]`, `verified_queries[]`.
---

### Q: Where does the Analyst semantic model file live?
**A:** In a Snowflake **stage** (not a table), referenced as `@db.schema.stage/path/model.yaml`. Pair with Git integration for versioning.
---

### Q: Difference between dimensions, measures, and time_dimensions?
**A:** Dimensions = categorical columns you GROUP BY. Measures = numeric columns you AGGREGATE (have `default_aggregation`). Time_dimensions = date/timestamp columns eligible for period-over-period logic.
---

### Q: What is `default_aggregation` and why is it required?
**A:** Tells Analyst how to aggregate a measure when the user says "revenue" without specifying. Values: `sum`, `avg`, `count`, `min`, `max`, `none` (for pre-aggregated expressions like ratios).
---

### Q: What are `synonyms` and `sample_values` on a dimension?
**A:** `synonyms` = alternate business names ("area","territory" for "region"). `sample_values` = short list of representative values (`["AMER","EMEA","APAC"]`) so the model can match entity mentions in questions.
---

### Q: What are verified queries and why do they matter?
**A:** SME-approved `(question, sql)` pairs stored under `verified_queries[]`. Analyst uses them as few-shot exemplars. They are the single highest-ROI accuracy lever, especially for window functions, nested CTEs, and company-specific logic (fiscal calendars).
---

### Q: How do you declare a join in a semantic model?
**A:** Under top-level `relationships[]` with `left_table`, `right_table`, `relationship_columns`, `join_type` (`inner`, `left_outer`), `relationship_type` (`many_to_one`, `one_to_one`, etc.). Analyst only joins tables where a relationship is declared.
---

### Q: What goes in `custom_instructions`?
**A:** Free-text business rules that apply to every query (fiscal calendar, test exclusions, default currency). Centralize rules here instead of duplicating in every verified_query. Keep it tight.
---

### Q: How do you handle a high-cardinality text dimension in Analyst?
**A:** Attach a `cortex_search_service` to the dimension so Analyst can fuzzy-resolve literals at query time, rather than enumerate every value in `sample_values`.
---

### Q: Does Analyst execute the SQL it generates?
**A:** No. It returns SQL in the response; your app executes it under the caller's role. Row access and masking apply at execution time. Streamlit chat UIs often execute automatically.
---

### Q: What is the content array in an Analyst response?
**A:** An array under `message.content` with items of type `text` (NL commentary), `sql` (`{type:"sql", statement:"..."}`), and `suggestions` (follow-up questions).
---

### Q: How do you implement multi-turn Analyst conversations?
**A:** Append the full history to the `messages` array including the prior assistant SQL and text turns. The service rewrites follow-ups like "and by product?" against prior context.
---

### Q: How do you evaluate Analyst accuracy?
**A:** Keep a golden set of (question, expected_result_set). Run each through Analyst, execute the SQL, compare result sets under set semantics (not string match). Target >90% match rate before deploying a new YAML version.
---

### Q: How do you version a semantic model safely?
**A:** Store YAML with version suffix in a stage (`sales_v1.yaml`, `sales_v2.yaml`) or Git-integrated repo. Point production app at a specific version; promote via staging → eval → prod.
---

### Q: What happens when a user asks something outside the semantic model's scope?
**A:** Analyst returns a text content block saying it cannot answer, often with `suggestions`. The app should surface this honestly, not fabricate SQL. Collect refusals as feedback for YAML extension.
---

## Domain 4 — Document AI

### Q: What model powers Document AI?
**A:** Arctic-TILT (Text-Image-Layout Transformer) — a multimodal model that jointly encodes OCR'd text tokens, their 2D bounding-box coordinates, and visual features. Layout-aware so "Total" and "$429.50" being adjacent is a strong signal.
---

### Q: How is Document AI different from PARSE_DOCUMENT?
**A:** PARSE_DOCUMENT extracts raw text (layout or OCR mode). Document AI extracts structured named fields via a trained model with a schema you define. Different jobs — one gives you a blob, the other gives you rows.
---

### Q: What is the Document AI PREDICT syntax?
**A:** `<db>.<schema>.<model_name>!PREDICT(document_input, model_version)` where `document_input` is typically `GET_PRESIGNED_URL(@stage, 'file.pdf', 3600)` and `model_version` is an integer. Returns an OBJECT.
---

### Q: What is the return shape of PREDICT?
**A:** OBJECT with `__documentMetadata.ocrScore` plus one entry per schema field. Scalars return `{"value":..., "score":...}`. Table fields return an array of row-objects.
---

### Q: Walk through the Document AI build workflow.
**A:** 1) Create build in Snowsight → AI & ML → Document AI. 2) Upload 10–100 sample docs. 3) Define schema (fields = NL questions + types). 4) Review model's baseline predictions. 5) Train (fine-tune) with ~20+ reviewed docs. 6) Publish v1 (callable via `!PREDICT`). 7) Iterate.
---

### Q: How does Document AI integrate with pipelines?
**A:** Pattern: stage with directory table → stream on INSERT → task runs every N minutes → `INSERT ... SELECT ... !PREDICT(...)` → extract table with `needs_review` flag by confidence threshold. Low-confidence rows go to a Streamlit review UI.
---

### Q: What data types does a Document AI field support?
**A:** STRING (default), NUMBER, DATE, BOOLEAN. Model outputs values coerced to the declared type (or NULL on failure).
---

### Q: How are Document AI models versioned?
**A:** Each publish is an incrementing integer version. Reference with `!PREDICT(url, 3)` for version 3. Drop old versions via `DROP MODEL VERSION model!v1`. Pin production to a known-good version.
---

### Q: How does Document AI billing work?
**A:** Per processed page for both training and inference, plus warehouse credits for pipeline tasks and stage storage. A 20-page PDF = 20 units regardless of how many fields you extract (one forward pass).
---

### Q: Which usage view tracks Document AI consumption?
**A:** `SNOWFLAKE.ACCOUNT_USAGE.DOCUMENT_AI_USAGE_HISTORY`.
---

### Q: What grants are needed to call PREDICT on a published Document AI model?
**A:** USAGE on the model, READ on the backing stage, and USAGE on the `GET_PRESIGNED_URL` function. Example: `GRANT USAGE ON MODEL db.sch.M TO ROLE R; GRANT READ ON STAGE db.sch.S TO ROLE R;`.
---

### Q: How does Document AI fine-tuning work internally?
**A:** Parameter-efficient fine-tuning (LoRA-style adapters) over Arctic-TILT using your reviewed (document, field, correct-answer) triples. Minimum ~20 reviewed docs for a stable gradient. Train in minutes–tens-of-minutes.
---

## Domain 5 — AI & ML Studio & Notebooks

### Q: Snowflake Notebook: what are the two runtime types?
**A:** Warehouse runtime (runs against a virtual warehouse, Anaconda-curated packages, no GPU) and Container runtime (Snowpark Container Services, CPU or GPU compute pools, full pip + custom Docker). Pick container for Hugging Face / GPU / deep learning.
---

### Q: What is `SYSTEM$GPU_BASIC_RUNTIME`?
**A:** The built-in GPU-capable notebook runtime image. Used with `RUNTIME_NAME = 'SYSTEM$GPU_BASIC_RUNTIME'` and a GPU compute pool (`INSTANCE_FAMILY = GPU_NV_S` etc.).
---

### Q: How do you schedule a notebook?
**A:** Via a Task: `CREATE TASK t SCHEDULE = 'USING CRON 0 2 * * * UTC' AS EXECUTE NOTEBOOK db.sch.nb()`. The notebook must have a LIVE VERSION set. Pass params with `EXECUTE NOTEBOOK nb(OBJECT_CONSTRUCT('key','value'))`.
---

### Q: What are the AI & ML Studio tiles?
**A:** Cortex Analyst (semantic model builder), Cortex Search (service wizard), Document AI (build/review/train), Cortex Fine-Tuning (job builder), Playground (model/prompt A/B).
---

### Q: What is the Snowpark ML Model Registry?
**A:** `snowflake.ml.registry.Registry` — a schema-level catalog for ML models. `log_model(model, model_name, version_name, conda_dependencies)` persists a model; it's then callable as a SQL method like `MY_MODEL!PREDICT(...)`.
---

### Q: What is the Snowpark ML Feature Store?
**A:** A schema-level service for defining `Entity`, `FeatureView`, and point-in-time correct feature joins. `fs.register_feature_view(fv, version='v1')` materializes features with scheduled refresh. Useful for hybrid LLM+ML systems.
---

### Q: How do you handle secrets for external API calls from a notebook?
**A:** Create a `SECRET`, a `NETWORK RULE` (egress), an `EXTERNAL ACCESS INTEGRATION`, grant USAGE on the integration + READ on the secret, then `ALTER NOTEBOOK nb SET EXTERNAL_ACCESS_INTEGRATIONS = (eai) SECRETS = ('key'=secret)`. Fetch inside Python via `_snowflake.get_generic_secret_string('key')`.
---

### Q: How do you integrate Git with Snowflake Notebooks?
**A:** Create API integration (`API_PROVIDER = git_https_api`), create `GIT REPOSITORY ... ORIGIN='https://github.com/...'`, `ALTER GIT REPOSITORY ... FETCH`, then reference `@repo/branches/main/path/notebook.ipynb` when creating the notebook.
---

### Q: What are typical governance grants for notebooks and ML objects?
**A:** `GRANT USAGE ON NOTEBOOK`, `OPERATE ON NOTEBOOK` (to run), `USAGE ON COMPUTE POOL`, `USAGE ON MODEL` (registry), `USAGE ON INTEGRATION`, `READ ON SECRET`.
---

### Q: How do you call Cortex from Python idiomatically?
**A:** `from snowflake.cortex import complete, summarize, translate, sentiment, extract_answer, embed_text_1024` and call `complete(model, prompt, session=session)`. For streaming: `complete(..., stream=True)` returns an iterator.
---

### Q: What are common cost controls on compute pools?
**A:** Set `AUTO_SUSPEND_SECS` short (60–300s), cap `MAX_NODES`, attach resource monitors to warehouses, drop old registry model versions, retain logging tables with TTLs, tag queries with `QUERY_TAG` for showback.
---

## Domain 6 — Gen AI App Patterns, Fine-Tuning, Governance

### Q: Prompt engineering hierarchy from cheapest to most expensive?
**A:** Zero-shot → few-shot → chain-of-thought (CoT) → self-consistency (CoT ensemble, majority vote) → reflexion (answer → self-critique → revise). Pick the lowest adequate tier.
---

### Q: When is CoT inappropriate?
**A:** On well-defined extraction tasks (use structured output), when you can't afford long outputs (CoT adds tokens), or when temperature 0 is required for diversity (CoT with temp 0 is a deterministic single path).
---

### Q: What is prompt injection and name three defenses?
**A:** Malicious user input that tries to override the system prompt. Defenses: (1) distinct system/user roles with system prioritization, (2) input classifier (AI_CLASSIFY for injection labels), (3) output filter (Cortex Guard), (4) least privilege / no secrets accessible to the LLM, (5) wrap retrieved RAG content in delimiters labeled as data not instructions.
---

### Q: What is a Cortex Agent?
**A:** A managed service at `/api/v2/cortex/agent:run` that orchestrates tool use (Cortex Analyst, Cortex Search, custom generic functions) under an LLM planner. Config: `model`, `messages`, `tools[]`, `tool_resources{}`. Handles tool dispatch, chat loop, streaming.
---

### Q: Three tool types usable in a Cortex Agent?
**A:** `cortex_analyst_text_to_sql` (structured SQL), `cortex_search` (unstructured retrieval), `generic` (user-defined function with input_schema; app executes and returns).
---

### Q: Cortex Fine-Tuning workflow in SQL?
**A:**
```sql
SELECT SNOWFLAKE.CORTEX.FINETUNE(
  'CREATE', 'db.sch.custom_model', 'mistral-7b',
  'SELECT prompt, completion FROM train_pairs',
  'SELECT prompt, completion FROM val_pairs'
);
-- Monitor
SELECT SNOWFLAKE.CORTEX.FINETUNE('DESCRIBE','<job_id>');
-- Use
SELECT SNOWFLAKE.CORTEX.COMPLETE('db.sch.custom_model', 'prompt');
```
---

### Q: Fine-tune vs RAG vs prompt engineering — the heuristic?
**A:** Missing knowledge → RAG. Bad format/style → Fine-tune. Capability or clarity gap → better prompt / better model. Fine-tuning doesn't inject knowledge well; RAG does.
---

### Q: How much training data for Cortex Fine-Tuning typically?
**A:** Hundreds to a few thousand (prompt, completion) pairs. Below ~100 is unstable; past ~10k is diminishing returns per example. Reserve 10–20% as hold-out validation.
---

### Q: Streamlit-in-Snowflake chat skeleton?
**A:**
```python
import streamlit as st
from snowflake.cortex import complete
from snowflake.snowpark.context import get_active_session
session = get_active_session()
if "chat" not in st.session_state: st.session_state.chat = []
for m in st.session_state.chat:
    with st.chat_message(m["role"]): st.markdown(m["content"])
if q := st.chat_input("Ask..."):
    st.session_state.chat.append({"role":"user","content":q})
    reply = complete("llama3.1-70b", q, options={"guardrails":True,"temperature":0}, session=session)
    st.session_state.chat.append({"role":"assistant","content":reply})
    st.rerun()
```
---

### Q: How do you distribute a Gen AI app to other Snowflake accounts?
**A:** As a Snowflake Native App: create `APPLICATION PACKAGE`, add manifest.yml + setup.sql + Streamlit code + stored procs to a stage, publish version, list on Marketplace or share privately. Consumer installs; Cortex calls bill to consumer.
---

### Q: What are three conversation memory tiers?
**A:** Short-term (last N turns in session_state), summary memory (periodically summarize older turns and prepend), semantic memory (embed turns into a vector store, retrieve relevant history each turn).
---

### Q: Three biggest cost levers for a Gen AI app?
**A:** (1) Smaller model when quality allows, (2) shorter prompts (retrieval over stuffing, chat history pruning, max_tokens cap), (3) caching responses by prompt hash + model + options.
---

### Q: How do you handle Cortex rate limits?
**A:** Catch error, retry with exponential backoff + jitter, use `TRY_COMPLETE` in batch pipelines, spread workload across models (per-model quotas), request quota increase for predictable bursts.
---

### Q: What belongs in an LLM observability trace table?
**A:** `trace_id`, `ts`, `user_name`, `app`, `model`, `system_prompt`, `user_prompt`, `retrieved_context`, `response`, `latency_ms`, `prompt_tokens`, `completion_tokens`, `cost_credits`, `guardrails_triggered`, `feedback_score`, `feedback_notes`. Apply masking + retention policies to the log itself.
---

### Q: How does masking policy interact with a Cortex call?
**A:** Masking applies at execution before the data reaches the Cortex function. So the LLM only sees masked values in the prompt or retrieved context. Governance preserved end-to-end; the LLM has no bypass.
---

### Q: Why pin model names/versions in production?
**A:** Prevents silent regressions when a model is upgraded or deprecated. Applies to `COMPLETE('llama3.1-70b', ...)`, semantic model file versions, Document AI model versions, and Registry model versions.
---

### Q: What's the structured-JSON output recipe?
**A:** Use `response_format: {type:'json', schema:{...}}` + system prompt "Reply with ONLY JSON" + `temperature: 0` + downstream `PARSE_JSON` + schema validation + retry on failure. Grammar-constrained decoding guarantees parseable output.
---

### Q: What's the "defense in depth" security checklist (abbreviated)?
**A:** Auth, RBAC (least privilege), masking policies, row access policies, input filter (classifier), structured prompt roles, output filter (Guard), rate limits, logging, audit review, secrets via EAI + SECRET, pinned model/package versions, observability dashboards.
---

### Q: What is function calling in COMPLETE?
**A:** Supported models (llama3.1 family, mistral-large2) accept `tools: [{type:"function", function:{name, description, parameters:JSONSchema}}]` plus `tool_choice: 'auto'`. Model may respond with a tool_call; app executes, feeds result back as a `tool` role message, calls COMPLETE again until final answer.
---

### Q: Default stack recommendation for a greenfield enterprise RAG app on Snowflake?
**A:** Snowpipe → PARSE_DOCUMENT → `SPLIT_TEXT_RECURSIVE_CHARACTER` (1200/200) → EMBED with `snowflake-arctic-embed-l-v2.0` → Cortex Search with TARGET_LAG=1h and ATTRIBUTES on category/region/date → COMPLETE with `llama3.1-70b` temp 0 + guardrails → Streamlit UI with sources → QUERY_TAG + CORTEX_FUNCTIONS_USAGE_HISTORY monitoring.
---

### Q: Final scenario heuristic for exam?
**A:** Ask: (1) data shape? structured → Analyst; unstructured text → RAG; forms with layout → Document AI. (2) Task? QA / classify / extract / generate. (3) Constraint? cost / latency / governance / scale. Pick the lowest-friction Snowflake-native tool that fits. External APIs are usually wrong when a Cortex service covers the use case.
---

### Q: Three cross-cutting concerns the exam loves?
**A:** Governance (RBAC, masking, row access, audit — unchanged for AI), observability (QUERY_HISTORY, CORTEX_FUNCTIONS_USAGE_HISTORY, DOCUMENT_AI_USAGE_HISTORY, custom traces), and cost/latency/quality trade-offs.
---

### Q: What happens if you call a Cortex model that isn't available in your region and CRI is DISABLED?
**A:** The call errors out (no silent fallback). To fix: `ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION'` or a region class — but be aware prompts leave the region, which may matter for GDPR/PCI.
---

### Q: When does an LLM-as-judge evaluation make sense?
**A:** When human eval is expensive and you need to compare two models/prompts at scale. Rules: use a third, stronger model as judge; different family than candidates (avoid self-preference); temperature 0; randomize A/B order per row to kill position bias; aggregate win-rate with CI.
---

### Q: Why does chunk size matter for retrieval precision?
**A:** Too small → fragments lose context, retrieved chunks are partial, LLM must reassemble. Too large → embedding averages multiple topics, retrieval precision drops. 1000–1500 chars for prose is the sweet spot; measure with recall@k on your golden set.
---

### Q: How do you keep a Cortex Search service's cost under control?
**A:** Choose a sensible `TARGET_LAG` (don't go to 1 minute if source changes hourly), set warehouse `AUTO_SUSPEND` short, pre-filter with ATTRIBUTE filters, keep the embedding model as small as quality allows (768-d if English-only), and cache COMPLETE outputs on identical prompts.
---

### Q: Which Snowflake object serves as the gate between Snowflake and an external HTTP API?
**A:** `EXTERNAL ACCESS INTEGRATION`, paired with one or more `NETWORK RULE` objects and `SECRET` objects (grants USAGE/READ to the role). Attached to a notebook, stored proc, or Streamlit app.
---

### Q: Why can't you simply paste an API key in a notebook cell?
**A:** Secrets in notebook text are version-controlled, visible in audit, and shareable with whoever has USAGE on the notebook object. Use Snowflake `SECRET` + EAI; fetch at runtime via `_snowflake.get_generic_secret_string(...)`.
---

### Q: What distinguishes a verified_query from a few-shot example in a regular prompt?
**A:** verified_queries live in the semantic model and are retrieved (semantic + lexical) only when a user's question matches them — not stuffed into every prompt. They are curated SME-signed exemplars, versioned with the model file.
---

### Q: Default aggregation for a ratio measure in a semantic model?
**A:** `default_aggregation: none` because the expression is already an aggregate (e.g., `SUM(a) / NULLIF(SUM(b),0)`). Never SUM a percent.
---

### Q: How do you prevent dividing by zero in a measure expression?
**A:** Wrap denominators with `NULLIF(x, 0)`: `SUM(a) / NULLIF(SUM(b), 0)` returns NULL instead of raising.
---

### Q: What's the trade-off of lowering a Document AI confidence threshold?
**A:** More rows auto-approve (less review burden) but more errors slip through. Keep the threshold high (0.90+) for critical fields like invoice_no; lower (0.70) is acceptable for noisy free-text fields.
---

### Q: What's the risk of putting PII sample_values in a semantic model?
**A:** The YAML is embedded in Analyst prompts and may be visible to any role with stage USAGE. Sample_values should be non-sensitive representative values, never real PII.
---

### Q: What's the difference between warehouse runtime "External Access" and container runtime network?
**A:** Warehouse runtime: outbound limited to configured External Access Integrations. Container runtime: full container network capability, still subject to Snowflake network policies and compute pool configuration. Container is more flexible but requires image management.
---

### Q: What's the Snowpark Container Services pattern for serving a custom LLM?
**A:** Build Docker image with model + FastAPI/Triton inference server → push to Snowflake image repo → `CREATE SERVICE` in a GPU compute pool → wrap as a Service Function (`CREATE FUNCTION ... SERVICE = ... ENDPOINT = ... MAX_BATCH_ROWS = ...`) → call from SQL.
---

### Q: Why would you choose Snowpark Container Services over Snowpark ML Model Registry for an OSS LLM?
**A:** SPCS is better for always-on inference servers with autoscaling, batching, and custom inference stacks. Registry is better for simpler model-as-function patterns with lower infra overhead. SPCS adds a service layer; Registry is just an object.
---

### Q: Describe the end-to-end billing for a RAG query hitting Cortex Search + COMPLETE.
**A:** (1) Query embed cost (small input tokens). (2) Cortex Search per-query credits. (3) COMPLETE input tokens for retrieved chunks + query + system prompt. (4) COMPLETE output tokens (usually dominant). (5) Warehouse credits for orchestrating SQL. Guard adds a classifier pass but minor.
---

### Q: Why is "lost in the middle" a reason to limit retrieved chunks?
**A:** LLMs attend less to tokens in the middle of long contexts — a correct answer buried mid-prompt may be ignored. Smaller k (3–7 chunks) often beats larger k on QA tasks. Reduce k before adding more model size.
---

### Q: What is `BUILD_SCOPED_FILE_URL` used for?
**A:** Generates a stage-aware URL that Cortex vision models and Document AI PREDICT can read. Scoped to the caller's session and privileges. Alternative: `GET_PRESIGNED_URL` with explicit expiration seconds.
---

### Q: What's the fastest way to compare two models' outputs for a prompt without writing eval harness code?
**A:** Open AI & ML Studio → Playground, paste the prompt, select two models, inspect side-by-side responses with token counts and credits. Promote to a SQL + QUERY_TAG eval harness once you pick a winner.
---

### Q: When does a `many_to_many` relationship in a semantic model become a problem?
**A:** Analyst doesn't model m2m directly — you must introduce a bridge table (join table) and declare two many_to_one relationships through it. Otherwise aggregations fan out and double-count.
---

### Q: What's the difference between `caller's rights` and `owner's rights` for a scheduled notebook?
**A:** Scheduled notebooks (via Task) run under the task's role by default (owner's rights). Caller's rights (where the user who triggers it is used) is more appropriate for user-facing dynamic work. Choose with intent; minimize privilege.
---

### Q: Which usage view is for query-level Cortex rollup vs per-call rows?
**A:** `CORTEX_FUNCTIONS_USAGE_HISTORY` = per call row (model, function, tokens, credits). `CORTEX_FUNCTIONS_QUERY_USAGE_HISTORY` = aggregated to `query_id` — join to `QUERY_HISTORY` for QUERY_TAG and user context.
---

### Q: Final mental model for the GES-C01 exam?
**A:** Three layers: (1) serverless LLM primitives — COMPLETE, helpers, embeddings, PARSE_DOCUMENT; (2) managed services — Cortex Search, Analyst, Document AI, Fine-Tuning, Agents; (3) dev/deploy surface — Notebooks, AI/ML Studio, Streamlit, Native Apps. Three cross-cutting concerns: governance (RBAC + masking + row access), observability (usage views + custom logs), cost (model size + prompt length + caching).
---
