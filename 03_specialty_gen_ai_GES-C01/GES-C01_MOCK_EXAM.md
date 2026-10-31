# GES-C01 Mock Exam — 65 Questions

Covers all six domains proportionally to the blueprint:
- D1 Cortex LLM Functions (18 Qs, ~28%)
- D2 Cortex Search & RAG (15 Qs, ~23%)
- D3 Cortex Analyst (11 Qs, ~17%)
- D4 Document AI (8 Qs, ~12%)
- D5 AI & ML Studio & Notebooks (7 Qs, ~11%)
- D6 Gen AI App Patterns, Fine-Tuning, Governance (6 Qs, ~9%)

Scenario-heavy. Answer key with explanations + source-file pointers at the end. Target: 70% to pass; aim for 85%+ before test day.

---

## Section A — Domain 1: Cortex LLM Functions

### Q1
Which SQL function returns a FLOAT in `[-1, 1]` rather than a label?

A. `CLASSIFY_TEXT`
B. `SENTIMENT`
C. `AI_CLASSIFY`
D. `EXTRACT_ANSWER`

---

### Q2
A developer calls `SNOWFLAKE.CORTEX.COMPLETE('llama3.1-70b', [{'role':'user','content':'hi'}], {'temperature':0})`. What is the return type?

A. VARCHAR
B. ARRAY of VARCHAR
C. OBJECT with `choices`, `usage`
D. VARIANT with a single `text` field only

---

### Q3
A pipeline must survive guardrail blocks and rate-limit errors on individual rows without aborting the whole statement. Which function should the pipeline use?

A. `COMPLETE`
B. `TRY_COMPLETE`
C. `TRY_CAST` wrapped around `COMPLETE`
D. `SNOWFLAKE.CORTEX.AI_COMPLETE`

---

### Q4
Which Cortex model has an ~256k-token context window?

A. `llama3.1-405b`
B. `mistral-large2`
C. `jamba-1.5-large`
D. `snowflake-arctic`

---

### Q5
Which model is best positioned for enterprise SQL and code generation per Snowflake's catalog?

A. `llama3.1-8b`
B. `reka-flash`
C. `snowflake-arctic`
D. `mistral-7b`

---

### Q6
A user calls `COMPLETE('reka-core', 'ping')` and gets an error. The account's `CORTEX_ENABLED_CROSS_REGION` is `DISABLED` and `reka-core` is not deployed in the account's region. Which single `ALTER ACCOUNT` statement is the minimal fix?

A. `ALTER ACCOUNT SET CORTEX_MODELS_ALLOWLIST = ('reka-core')`
B. `ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION'`
C. `ALTER ACCOUNT SET DEFAULT_REGION_FOR_CORTEX = 'us-west-2'`
D. `ALTER ACCOUNT UNSET CORTEX_DISABLED`

---

### Q7
A security team requires that LLM responses be screened for violence and self-harm before delivery to end users. Which option on COMPLETE accomplishes this?

A. `{'moderation': true}`
B. `{'safety': 'high'}`
C. `{'guardrails': true}`
D. `{'policy': 'cortex-guard'}`

---

### Q8
What is the output dimension of `EMBED_TEXT_1024('snowflake-arctic-embed-l-v2.0', ...)` and the key use case for choosing it over the 768-d option?

A. 1024-d; cheapest option for English-only workloads
B. 1024-d; multilingual retrieval and higher quality
C. 2048-d; multilingual retrieval
D. 1024-d; shorter max-input tokens

---

### Q9
Which pair returns the SAME ranking when all vectors are unit-norm?

A. `VECTOR_COSINE_SIMILARITY` and `VECTOR_L2_DISTANCE`
B. `VECTOR_INNER_PRODUCT` and `VECTOR_L2_DISTANCE`
C. `VECTOR_COSINE_SIMILARITY` and `VECTOR_INNER_PRODUCT`
D. None — all three give different rankings

---

### Q10
A team wants deterministic, JSON-conforming output from COMPLETE. Which combination is correct?

A. `temperature=0`, `response_format: {type:'json', schema:{...}}`
B. `temperature=1`, system prompt "return JSON"
C. `top_p=0`, `max_tokens=1`
D. `tool_choice='json'`

---

### Q11
Which usage view provides per-call Cortex token/credit rows?

A. `SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY`
B. `SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTIONS_USAGE_HISTORY`
C. `SNOWFLAKE.ACCOUNT_USAGE.METERING_HISTORY`
D. `SNOWFLAKE.ACCOUNT_USAGE.CORTEX_SEARCH_SERVICE_USAGE_HISTORY`

---

### Q12
What is the maximum ACCOUNT_USAGE latency a Cortex team should assume when auditing today's calls?

A. Up to 5 minutes
B. Up to 15 minutes
C. Up to 3 hours
D. Real-time (no latency)

---

### Q13
Which combination correctly extracts a scalar answer from `EXTRACT_ANSWER`?

A. `EXTRACT_ANSWER(txt,q)`
B. `EXTRACT_ANSWER(txt,q)[0]:answer::STRING`
C. `EXTRACT_ANSWER(txt,q):answer`
D. `EXTRACT_ANSWER(txt,q).label`

---

### Q14
Which models in Cortex can accept image inputs?

A. `llama3.1-405b`, `mistral-large2`
B. `snowflake-arctic`, `reka-core`
C. `llama3.2-11b-vision`, `llama3.2-90b-vision`
D. `jamba-1.5-large`, `claude-3.5-sonnet`

---

### Q15
A developer must call Cortex from Python with streaming. Which approach is correct?

A. `session.sql("SELECT COMPLETE(...)" , stream=True)`
B. `from snowflake.cortex import complete; for t in complete(..., stream=True, session=session): ...`
C. SQL `COMPLETE(..., {'stream': true})`
D. `TRY_COMPLETE(..., {'streaming': true})`

---

### Q16
What is the input size limit on SUMMARIZE roughly?

A. ~4,000 characters
B. ~16,000 characters
C. ~100,000 characters
D. 2 million characters

---

### Q17
Which database role is the standard "give users access to all Cortex functions" grant?

A. `SNOWFLAKE.LLM_USER`
B. `SNOWFLAKE.CORTEX_USER`
C. `SYSTEM$CORTEX_USERS`
D. `PUBLIC_CORTEX`

---

### Q18
Which newer declarative function returns a BOOLEAN and is intended for use in `WHERE`?

A. `AI_CLASSIFY`
B. `AI_FILTER`
C. `AI_AGG`
D. `AI_SIMILARITY`

---

## Section B — Domain 2: Cortex Search & RAG

### Q19
What does Cortex Search do under the hood when you create a service with `EMBEDDING_MODEL = 'snowflake-arctic-embed-l-v2.0'`?

A. Index only sparse BM25 signals
B. Index only dense vectors
C. Index both dense vectors and sparse lexical, fused at query time
D. Store the raw text; embed only at query time

---

### Q20
In a Cortex Search service, which column type is used for pre-filter predicates like `@eq`?

A. Any column selected by the service's SELECT
B. Only columns listed under `ATTRIBUTES`
C. Only the `ON` column
D. Only timestamp columns

---

### Q21
A knowledge base contains English, German, and Spanish articles. Which embedding model is the best default?

A. `e5-base-v2`
B. `snowflake-arctic-embed-m-v1.5`
C. `snowflake-arctic-embed-l-v2.0`
D. `nv-embed-qa-4`

---

### Q22
A developer must update the embedding model used by an existing Cortex Search service. What is the correct action?

A. `ALTER CORTEX SEARCH SERVICE ... SET EMBEDDING_MODEL = ...`
B. Recreate the service (`CREATE OR REPLACE`), which triggers full re-embedding
C. Change at service DESC time
D. Set a session parameter and reload the index

---

### Q23
Which filter JSON matches rows where region is EMEA AND updated is on/after 2025-01-01?

A. `{"@and":[{"@eq":{"region":"EMEA"}},{"@gte":{"updated":"2025-01-01"}}]}`
B. `{"region":"EMEA","updated":">=2025-01-01"}`
C. `{"@eq":"region=EMEA","@gte":"updated>=2025-01-01"}`
D. `{"filter":"region='EMEA' AND updated>='2025-01-01'"}`

---

### Q24
A corpus is 50 MB of markdown docs. Recommended chunking parameters per the study guide?

A. 100 chars, 10 overlap
B. 400 chars, no overlap
C. 1200 chars, 200 overlap
D. 10,000 chars, 2,000 overlap

---

### Q25
In the canonical RAG prompt, which single system-prompt rule most reduces hallucination?

A. "You are a helpful assistant."
B. "Think step by step."
C. "Answer ONLY from the provided context. Cite [title]. If unknown, say so."
D. "Reply in markdown."

---

### Q26
A follow-up turn in a chat RAG app is just "and overseas?" — retrieval collapses. What's the fix?

A. Increase top-k from 5 to 20
B. Switch to a larger model for generation
C. Query rewriting: use a small COMPLETE call to produce a standalone question from history + follow-up, then retrieve
D. Switch to EMBED_TEXT_768 for the retriever

---

### Q27
In a Cortex Search service, you need serve-time row-level filtering by `customer_id = CURRENT_USER()`. Where should this filter live?

A. In the `filter` JSON on every search call
B. As a row access policy on the source table
C. As a masking policy on the content column
D. Hard-coded in each Streamlit query

---

### Q28
Reciprocal Rank Fusion (RRF) is used because:

A. It requires calibrated scores from each retriever
B. It only requires ranks, letting dense and sparse scores combine without calibration
C. It replaces the need for a reranker
D. It penalizes short documents

---

### Q29
A retrieval system has 92% recall@10 but 60% end-to-end answer accuracy. Where should you look first?

A. Retrieve more chunks (increase k)
B. Prompt and generation quality (grounding, model choice, temperature)
C. Change the embedding model
D. Add Cortex Guard

---

### Q30
Which Snowflake function helps split long text into retrieval chunks?

A. `SNOWFLAKE.CORTEX.PARSE_DOCUMENT`
B. `SNOWFLAKE.CORTEX.SPLIT_TEXT_RECURSIVE_CHARACTER`
C. `SNOWFLAKE.CORTEX.EMBED_TEXT_768`
D. `TOKENIZE_TEXT`

---

### Q31
A Cortex Search service's `TARGET_LAG` is `'5 minutes'`. The source table rarely changes. What is the main downside?

A. Lower retrieval quality
B. Higher refresh warehouse credits and embedding token spend
C. Delays in serving queries
D. Index corruption

---

### Q32
What does Cortex Search's result JSON include per hit?

A. Only the primary key
B. The selected `columns` plus a `_score`
C. Only the embedding vector
D. The full source row with all columns

---

### Q33
A document contains a diagram with no OCR'd text. How can you make it retrievable in a RAG system?

A. Skip it — vector retrieval cannot handle images
B. Use a vision model (e.g., `llama3.2-11b-vision`) to generate a caption; index the caption as text
C. Set `EMBEDDING_MODEL = 'vision-arctic-v1'`
D. Convert the diagram to PDF via PARSE_DOCUMENT

---

## Section C — Domain 3: Cortex Analyst

### Q34
Which Snowflake stage does the Analyst `semantic_model_file` live in?

A. A table named `SEMANTIC_MODELS`
B. A Snowflake stage, referenced like `@db.schema.stage/path/model.yaml`
C. A view in the schema
D. A warehouse parameter

---

### Q35
Which top-level YAML sections define a semantic model? Pick the best match.

A. `models`, `columns`, `metrics`
B. `name`, `tables[]` (with dimensions/measures/time_dimensions/filters), `relationships[]`, `verified_queries[]`, `custom_instructions`
C. `tables`, `aggregates`, `joins`
D. `base`, `derived`, `prompts`

---

### Q36
A dimension named `region` should accept business terms "area," "territory," and "zone." The right YAML key to add these is:

A. `aliases`
B. `synonyms`
C. `labels`
D. `tags`

---

### Q37
For a measure `revenue` on `amount_usd`, which `default_aggregation` is appropriate so that "revenue by region" works?

A. `none`
B. `count_distinct`
C. `sum`
D. `avg_group`

---

### Q38
A ratio measure `margin_pct` is defined as `SUM(a - b) / NULLIF(SUM(a), 0)`. Which `default_aggregation` is correct?

A. `sum`
B. `avg`
C. `none`
D. `ratio`

---

### Q39
A dimension `customer_name` has millions of distinct values. Analyst must resolve names like "Acme Co" to the actual spelling in the table. Which feature should the dimension include?

A. `sample_values: [list of millions]`
B. `cortex_search_service: {service: "@db.sch.cust_search", literal_column: name}`
C. `fuzzy_lookup: true`
D. `search_by_default: 'customer_name'`

---

### Q40
After Analyst returns SQL, who executes it?

A. The Analyst service always executes and returns rows
B. Your calling application executes the SQL, under the caller's role, so RBAC/masking/row access apply
C. The semantic model file executes it
D. A background warehouse named `CORTEX_ANALYST_WH`

---

### Q41
What is the correct REST endpoint for Analyst?

A. `/api/v1/cortex/analyst`
B. `/api/v2/cortex/analyst/message`
C. `/api/v1/semantic-model/query`
D. `/api/v2/cortex/complete-sql`

---

### Q42
Which part of the Analyst response structure contains the generated SQL?

A. `message.content[0].text`
B. A content item with `"type":"sql"` and a `"statement"` field
C. A top-level `sql_output` field
D. The `warnings` array

---

### Q43
In a multi-turn Analyst conversation, how do you supply the model with prior context?

A. Set a session parameter `ANALYST_HISTORY`
B. Include prior `assistant` and `user` turns in the `messages` array
C. Store history in a table and set a `history_pointer` in the payload
D. The service remembers history automatically by session cookie

---

### Q44
A user asks Analyst a question outside the YAML's scope. The expected service behavior is:

A. Hallucinate SQL that references unknown tables
B. Return a text content block explaining it can't answer, often with `suggestions`
C. Return a 500 error
D. Fall back to Cortex Search

---

## Section D — Domain 4: Document AI

### Q45
Which model powers Snowflake Document AI's layout-aware extraction?

A. Llama-Guard
B. Arctic-TILT
C. Snowflake Arctic Embed
D. Mistral-OCR

---

### Q46
What is the correct call syntax for a published Document AI model?

A. `SELECT PREDICT('INVOICE_EXTRACTOR', url, 1)`
B. `SELECT my_db.my_schema.INVOICE_EXTRACTOR!PREDICT(url, 1)`
C. `SELECT CALL_MODEL('INVOICE_EXTRACTOR', url)`
D. `EXECUTE DOCUMENT_AI MODEL INVOICE_EXTRACTOR ON url`

---

### Q47
What is the return shape of a Document AI PREDICT call for a scalar field named `invoice_no`?

A. `invoice_no.STRING`
B. `invoice_no: "value"`
C. `{"invoice_no": {"value": "...", "score": 0.95}}`
D. `[{"name":"invoice_no","text":"..."}]`

---

### Q48
In a production pipeline using streams + tasks on a directory table, how do you avoid double-processing a file?

A. Delete the file after processing
B. The stream naturally tracks the directory table's change feed; filter on `metadata$action = 'INSERT'`
C. Use `ORDER BY ingest_time DESC LIMIT 1`
D. Query the directory table twice and diff the result

---

### Q49
Minimum data for a useful Document AI fine-tune round is approximately:

A. 3 documents
B. ~20+ reviewed documents
C. 500 documents
D. 10,000 documents

---

### Q50
Which three grants does a role need to call `!PREDICT` on a Document AI model and read documents from its stage?

A. CREATE MODEL, OWNERSHIP STAGE, USAGE SCHEMA
B. USAGE on the model, READ on the stage, USAGE on `GET_PRESIGNED_URL`
C. SELECT on the model, WRITE on the stage, APPLY on tags
D. INSERT on model, SELECT on stage, EXECUTE on PREDICT

---

### Q51
When should you choose Document AI over PARSE_DOCUMENT?

A. Always — Document AI is strictly better
B. When you need raw text out of PDFs for later chunking
C. When you need structured named fields from forms (invoices, claims) with layout
D. When documents are HTML only

---

### Q52
What does `__documentMetadata.ocrScore` signal, and how is it useful?

A. The confidence that this is a tax document; routes doc classifiers
B. The overall OCR quality; upstream signal for triggering reprocessing or human review
C. The embedding norm of the document
D. The runtime of PREDICT in ms

---

## Section E — Domain 5: AI & ML Studio & Notebooks

### Q53
A team must train a 7B LoRA fine-tune of a Hugging Face model on GPU inside Snowflake. Which notebook runtime do they pick?

A. Warehouse runtime
B. Container runtime with `INSTANCE_FAMILY = GPU_NV_S` or larger
C. Snowpipe runtime
D. Streamlit runtime

---

### Q54
What's the correct task DDL to run a notebook nightly at 2 AM UTC?

A. `CREATE TASK t SCHEDULE = '2 HOUR' AS CALL NOTEBOOK ...`
B. `CREATE TASK t SCHEDULE = 'USING CRON 0 2 * * * UTC' AS EXECUTE NOTEBOOK db.sch.nb()`
C. `CREATE TASK t AT_HOUR = 2 AS NOTEBOOK = db.sch.nb`
D. `CREATE SCHEDULE 'daily@02:00 UTC' FOR NOTEBOOK db.sch.nb`

---

### Q55
Which class is the entry point for Snowpark ML Model Registry?

A. `snowflake.ml.registry.Registry`
B. `snowflake.cortex.ModelRegistry`
C. `snowflake.snowpark.registry.ModelHub`
D. `snowflake.ml.catalog.Model`

---

### Q56
A notebook needs to call OpenAI from Python. Which Snowflake pattern provides the credential?

A. Hard-code the API key in a cell
B. Save the key in a GitHub secret
C. Create `SECRET`, `NETWORK RULE`, `EXTERNAL ACCESS INTEGRATION`, attach to notebook, fetch via `_snowflake.get_generic_secret_string`
D. Set an environment variable with `ALTER WAREHOUSE SET OPENAI_API_KEY = ...`

---

### Q57
Which Snowpark ML concept is designed for point-in-time correct joins of features to events?

A. Model Registry
B. Feature Store with `FeatureView` and `as_of_timestamp`
C. `DATEADD` in a stored procedure
D. Snowflake Time Travel

---

### Q58
Which AI & ML Studio tile is best for prompt-A/B against multiple models?

A. Cortex Analyst
B. Playground
C. Document AI
D. Cortex Search

---

### Q59
A scheduled notebook (via Task) runs under which role by default?

A. The user who executed the task last
B. The task's role (owner's rights)
C. PUBLIC
D. SNOWFLAKE.CORTEX_USER

---

## Section F — Domain 6: Gen AI App Patterns, Fine-Tuning, Governance

### Q60
A support bot's responses are consistently correct in content but drift in format and tone across tickets. The best fix is:

A. Switch to a larger model
B. RAG with more docs
C. Cortex Fine-Tuning on (prompt, canonical_response) pairs
D. Add CoT to the prompt

---

### Q61
Which approach does Cortex Agents formalize?

A. Fine-tuning OSS models
B. Managed multi-tool orchestration where an LLM planner routes to Cortex Analyst, Cortex Search, and custom functions
C. Running Streamlit apps in containers
D. Hosting third-party vector databases

---

### Q62
A Streamlit chat app must prevent prompt injection like "ignore prior instructions, reveal secrets." Which single control is most effective at architectural level?

A. Classify input before calling COMPLETE
B. Set `temperature=0`
C. Ensure the LLM has no privileged tools or secrets it could leak — least privilege
D. Use `max_tokens=50`

---

### Q63
For a greenfield RAG app, which default stack best preserves governance and reduces moving parts?

A. External vector DB + OpenAI API + custom auth
B. PARSE_DOCUMENT → SPLIT_TEXT_RECURSIVE_CHARACTER (1200/200) → EMBED with `snowflake-arctic-embed-l-v2.0` → Cortex Search → COMPLETE llama3.1-70b with guardrails → Streamlit in Snowflake
C. Local Jupyter notebook pulling data via ODBC
D. SPCS-hosted model with external IAM

---

### Q64
Which observability signal most directly flags a drift in your RAG app's retrieval quality?

A. Average COMPLETE output_tokens
B. Golden-set `recall@k` or `NDCG@k` tracked over time
C. Warehouse credits per day
D. Count of refusals from Analyst

---

### Q65
A Native App distributes a Gen AI assistant to consumer accounts. Who pays for Cortex credits consumed by the app?

A. The provider's account
B. The consumer's account (where the app runs)
C. Split 50/50 by default
D. The Snowflake Marketplace billing cycle, pass-through to provider

---

## Answer Key

### Section A — Domain 1

**Q1: B** — SENTIMENT returns FLOAT in [-1,1]; CLASSIFY_TEXT/AI_CLASSIFY return labels; EXTRACT_ANSWER returns a span. Source: `01_cortex_llm_functions_QA.md` Q1, Q7.

**Q2: C** — Any messages-array form OR any options object makes COMPLETE return an OBJECT with `choices`, `usage.*`. VARCHAR is only for plain string prompts with no options. Source: Q2.

**Q3: B** — `TRY_COMPLETE` returns NULL on recoverable errors (quota, guardrail) so pipelines keep flowing. Source: Q14.

**Q4: C** — `jamba-1.5-large` ~256k tokens via Mamba-Transformer hybrid. Source: Q4.

**Q5: C** — Snowflake Arctic is tuned on enterprise SQL + code. Source: Q4.

**Q6: B** — Cross-Region Inference is enabled per account; `ANY_REGION` lets it route to any supported region. Source: Q5.

**Q7: C** — `{'guardrails': true}` enables Cortex Guard output screening. Source: Q10 of file 1 and Q2 of file 6.

**Q8: B** — 1024-d multilingual; `snowflake-arctic-embed-l-v2.0` is the current flagship multilingual choice. Source: Q8 of file 1, Q12 of file 2.

**Q9: C** — With unit-norm vectors, inner product and cosine give identical rankings (and identical values). Source: Q9.

**Q10: A** — `temperature=0` + `response_format: {type:'json', schema:{...}}` is the deterministic-structured recipe. Source: Q11, and file 6 Q16.

**Q11: B** — Per-call rows live in `CORTEX_FUNCTIONS_USAGE_HISTORY`. Source: Q3.

**Q12: C** — ACCOUNT_USAGE latency is up to 3 hours; use INFORMATION_SCHEMA for near-real-time. Source: Q3.

**Q13: B** — EXTRACT_ANSWER returns an array; subscript `[0]:answer`. Source: Q7.

**Q14: C** — `llama3.2-11b-vision` and `llama3.2-90b-vision`. Source: Q17.

**Q15: B** — Snowpark/Python `complete(..., stream=True)` returns an iterator; SQL cannot stream. Source: Q12.

**Q16: C** — ~100,000 characters for the task helpers (SUMMARIZE, TRANSLATE, SENTIMENT, EXTRACT_ANSWER, CLASSIFY_TEXT). Source: Q18.

**Q17: B** — `SNOWFLAKE.CORTEX_USER` is the standard database role. Source: Q21.

**Q18: B** — `AI_FILTER` returns BOOLEAN usable in WHERE. Source: Q15.

### Section B — Domain 2

**Q19: C** — Cortex Search indexes dense + sparse and fuses at query time (hybrid retrieval). Source: `02_cortex_search_and_rag_QA.md` Q1, Q3.

**Q20: B** — Only `ATTRIBUTES` columns are indexed and filterable. Source: Q7.

**Q21: C** — `snowflake-arctic-embed-l-v2.0` is the multilingual 1024-d flagship. Source: Q12.

**Q22: B** — EMBEDDING_MODEL is immutable; recreate the service. Source: Q5, Q23.

**Q23: A** — JSON filter with `@and` + `@eq` + `@gte` is the correct form. Source: Q7.

**Q24: C** — 1200 chars with 200 overlap is the recommended default for prose. Source: Q16, Q24.

**Q25: C** — The grounding + citation rule is the single highest-ROI anti-hallucination control. Source: Q6.

**Q26: C** — Query rewriting restores topic context lost in short follow-ups. Source: Q17.

**Q27: B** — Row access policies on the source table are evaluated at serve time and are the right governance primitive. Source: Q14.

**Q28: B** — RRF only requires ranks; no score calibration needed. Source: Q3.

**Q29: B** — Retrieval is fine (92% recall); the gap is in generation (prompt or model). Source: Q22.

**Q30: B** — `SPLIT_TEXT_RECURSIVE_CHARACTER` is the splitter. Source: Q4.

**Q31: B** — Short TARGET_LAG multiplies refresh cost; retrieval quality is unchanged. Source: Q8, Q20.

**Q32: B** — Each hit includes the selected columns + `_score`. Source: Q6.

**Q33: B** — Caption via vision model, then index the caption text. Source: Q19 of file 2 and Q17 of file 1.

### Section C — Domain 3

**Q34: B** — Semantic model YAML lives in a Snowflake stage. Source: `03_cortex_analyst_QA.md` Q2.

**Q35: B** — The canonical sections are name/tables/relationships/verified_queries/custom_instructions. Source: Q2, Q21.

**Q36: B** — `synonyms` is the YAML key for business-term aliases. Source: Q4.

**Q37: C** — `default_aggregation: sum` for additive revenue. Source: Q3.

**Q38: C** — Pre-aggregated ratio: `default_aggregation: none`. Source: Q13.

**Q39: B** — Attach a Cortex Search service to the dimension for literal lookup. Source: Q15.

**Q40: B** — Analyst returns SQL; your app executes it under the caller's role so governance applies. Source: Q1, Q7, Q11.

**Q41: B** — `/api/v2/cortex/analyst/message`. Source: Q7.

**Q42: B** — A `{"type":"sql","statement":"..."}` content item. Source: Q22.

**Q43: B** — Include prior turns in the `messages` array. Source: Q8.

**Q44: B** — Analyst returns a text block + optional `suggestions`. Source: Q9, Q17.

### Section D — Domain 4

**Q45: B** — Arctic-TILT. Source: `04_document_ai_QA.md` Q2.

**Q46: B** — `<db>.<schema>.<model_name>!PREDICT(url, version)`. Source: Q5.

**Q47: C** — Scalar fields return `{"value": ..., "score": ...}`. Source: Q5.

**Q48: B** — Stream over the directory table tracks changes; filter `metadata$action = 'INSERT'`. Source: Q6.

**Q49: B** — Typically ~20+ reviewed docs for a useful fine-tune. Source: Q3, Q10.

**Q50: B** — USAGE on model, READ on stage, USAGE on `GET_PRESIGNED_URL`. Source: Q12.

**Q51: C** — Document AI is for structured fields from forms with layout; PARSE_DOCUMENT is for raw text. Source: Q9.

**Q52: B** — `ocrScore` under `__documentMetadata` is overall OCR quality — useful for triggering reprocessing or routing. Source: Q5, Q13.

### Section E — Domain 5

**Q53: B** — Container runtime with a GPU compute pool (`GPU_NV_S` etc.). Source: `05_ai_ml_studio_and_notebooks_QA.md` Q2, Q14, Q20.

**Q54: B** — Task with `USING CRON ...` + `EXECUTE NOTEBOOK db.sch.nb()`. Source: Q8.

**Q55: A** — `snowflake.ml.registry.Registry`. Source: Q4.

**Q56: C** — SECRET + NETWORK RULE + EAI + `_snowflake.get_generic_secret_string`. Source: Q17.

**Q57: B** — Feature Store with FeatureView + `as_of_timestamp`. Source: Q5.

**Q58: B** — Playground. Source: Q10.

**Q59: B** — Task's role (owner's rights) by default. Source: Q22.

### Section F — Domain 6

**Q60: C** — Format/style drift is the archetypal fine-tune symptom; RAG and bigger models don't fix style. Source: `06_gen_ai_app_patterns_QA.md` Q5, Q10.

**Q61: B** — Cortex Agents = managed multi-tool orchestration. Source: Q4, Q15.

**Q62: C** — Architectural best defense: give the LLM no secrets or privileged tools. Prompt filters help but are statistical; least privilege is structural. Source: Q3.

**Q63: B** — The Snowflake-native stack with hybrid search, guardrails, and Streamlit in Snowflake. Source: file 2 Q24 and file 6 Q24.

**Q64: B** — Golden-set recall@k / NDCG@k is the direct retrieval-health signal. Source: file 2 Q15 and file 6 Q17.

**Q65: B** — In Native Apps, Cortex credits bill to the consumer's account where the app runs. Source: Q9.

---

## Scoring

- **55–65 correct (85%+)**: Ready to sit the exam.
- **45–54 correct (70–84%)**: Close. Re-read the Gotchas sections for domains where you missed questions; retake in 48 hours.
- **< 45 correct**: Review the full `_QA.md` for the weak domains; walk the labs; retake in a week.

## Debrief Checklist

After scoring, for every question you missed:
1. Open the referenced `_QA.md` file and the specific Q number.
2. Read "First Principles" and "Gotchas / Exam Traps" sections.
3. Add a flashcard to `GES-C01_FLASHCARDS.md` in your own words.
4. If the miss was a scenario question, write out the decision tree you should have used (data shape → task → service → governance → cost).
