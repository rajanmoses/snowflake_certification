# Snowflake Specialty: Generative AI (GES-C01) — Study Guide Index

## Certification Overview

**Exam Code:** GES-C01
**Full Name:** SnowPro Specialty: Generative AI
**Target Audience:** Data engineers, ML engineers, application developers, and architects who build and deploy Gen AI workloads on Snowflake using Cortex, Cortex Search, Cortex Analyst, Document AI, Snowflake Notebooks, and Streamlit in Snowflake.
**Prerequisites:** Working SnowPro Core knowledge (RBAC, warehouses, stages, Snowpark basics). Familiarity with LLM concepts (tokens, embeddings, prompts, context windows) and SQL.
**Format:** Multiple choice / multi-select, scenario-based. ~65 scored items. Passing bar in the 70–75% range (confirm current cut on the official page).
**Duration:** Approximately 90–115 minutes.
**Delivery:** Online proctored or Pearson VUE test center.
**Validity:** 2 years, renewable via recertification.

## Blueprint Weightings (approximate)

| # | Domain | Weight | Companion File |
|---|---|---|---|
| 1 | Cortex LLM Functions (COMPLETE, helpers, embeddings, Guard, CRI) | 25–30% | `01_cortex_llm_functions_QA.md` |
| 2 | Cortex Search & RAG patterns | 20–25% | `02_cortex_search_and_rag_QA.md` |
| 3 | Cortex Analyst (text-to-SQL + semantic models) | 15–20% | `03_cortex_analyst_QA.md` |
| 4 | Document AI (Arctic-TILT, PREDICT, pipelines) | 10–15% | `04_document_ai_QA.md` |
| 5 | AI & ML Studio, Notebooks, Snowpark ML | 10–15% | `05_ai_ml_studio_and_notebooks_QA.md` |
| 6 | Gen AI app patterns, Fine-Tuning, governance | 15–20% | `06_gen_ai_app_patterns_QA.md` |

Weights above add to >100% because many exam items span two domains (e.g., a RAG scenario mentions governance and model choice at once). Plan to be strong on all six.

## Module Summaries

### 01 — Cortex LLM Functions (25–30%)
Everything in the `SNOWFLAKE.CORTEX` namespace. COMPLETE (string vs messages form, response_format, guardrails, tools), task-specific helpers (SUMMARIZE, TRANSLATE, SENTIMENT, EXTRACT_ANSWER, CLASSIFY_TEXT), embeddings (EMBED_TEXT_768, EMBED_TEXT_1024 and the Arctic-embed / Voyage / e5 model names), PARSE_DOCUMENT, AISQL family (AI_CLASSIFY, AI_FILTER, AI_AGG, AI_SUMMARIZE_AGG, AI_SIMILARITY), vision models (llama3.2-11b-vision, llama3.2-90b-vision), token/credit accounting via `CORTEX_FUNCTIONS_USAGE_HISTORY`, cross-region inference (CRI), Cortex Guard, TRY_COMPLETE, structured output (JSON schema constrained decoding), streaming via Python/REST, and governance via `SNOWFLAKE.CORTEX_USER` database role. Know the model catalog at the level of "which model wins for which workload."

### 02 — Cortex Search & RAG (20–25%)
Managed hybrid retrieval service (dense ANN + sparse BM25 + RRF fusion). Semantic chunking strategies (fixed-size, recursive, semantic, markdown-aware), `SPLIT_TEXT_RECURSIVE_CHARACTER`, embedding model options (`snowflake-arctic-embed-m-v1.5`, `snowflake-arctic-embed-l-v2.0`, `voyage-multilingual-2`, `nv-embed-qa-4`, `e5-base-v2`), service DDL with `ATTRIBUTES`, `TARGET_LAG`, `WAREHOUSE`, `EMBEDDING_MODEL`, filter JSON syntax (`@eq`, `@contains`, `@and`, `@or`, `@not`, `@gte`, `@lte`), `SEARCH_PREVIEW` SQL / Python / REST, auto-refresh via source change tracking, re-ranking cascades, multi-turn RAG with query rewriting, chunk vs document-level retrieval, cost modeling, and recall@k / NDCG eval.

### 03 — Cortex Analyst (15–20%)
Text-to-SQL grounded by a YAML semantic model stored in a stage. YAML sections: `name`, `description`, `tables[]` with `base_table`, `primary_key`, `dimensions` (with `synonyms`, `sample_values`, `cortex_search_service`), `measures` (with `default_aggregation`), `time_dimensions`, `filters`, `relationships` (`join_type`, `relationship_type`), `verified_queries`, `custom_instructions`. REST API at `/api/v2/cortex/analyst/message` with messages/tool/content array response structure. Multi-turn chat, Streamlit integration via `_snowflake.send_snow_api_request`, governance passthrough (RBAC/masking/row access at execution time), versioning via stage files + Git integration, result-set-equivalence eval harnesses, hybrid Analyst+Search for high-cardinality name resolution.

### 04 — Document AI (10–15%)
Arctic-TILT (Text-Image-Layout Transformer) for layout-aware extraction. Build workflow: upload samples → define schema (field = NL question + type) → review predictions → fine-tune (LoRA) → publish version. The `<db>.<schema>.<model>!PREDICT(url, version)` syntax, `GET_PRESIGNED_URL` for stage files, OBJECT return shape with `{value, score}` per field, `__documentMetadata.ocrScore`. Pipeline pattern: directory table + stream + task + extract table + confidence-gated review UI. Billing per processed page. Governance via model object USAGE + stage READ. Comparison with PARSE_DOCUMENT (text only) and 3rd parties (Textract/Form Recognizer).

### 05 — AI & ML Studio & Notebooks (10–15%)
Snowflake Notebooks with warehouse runtime vs container runtime (CPU/GPU compute pools like `GPU_NV_S`, `GPU_NV_M`). AI & ML Studio tiles for Analyst, Search, Document AI, Fine-Tuning, Playground. Snowpark ML Model Registry (`log_model`, `!PREDICT`), Feature Store with entities, feature views, and point-in-time join. Package management (Anaconda channel vs pip in containers), Git repository integration, scheduled notebooks via `EXECUTE NOTEBOOK` tasks, Streamlit in-notebook prototyping, Hugging Face model deployment via Registry or Snowpark Container Services, secret management via SECRET + External Access Integration, GPU LoRA training, resource monitors.

### 06 — Gen AI App Patterns, Fine-Tuning, Governance (15–20%)
Prompt engineering hierarchy (zero-shot, few-shot, chain-of-thought, self-consistency, reflexion), Cortex Guard details, prompt-injection defenses (input classifier + system prompt layering + Guard + least privilege), Cortex Agents (planner LLM over tools: `cortex_search`, `cortex_analyst_text_to_sql`, `generic`), Cortex Fine-Tuning (data prep, LoRA, job submission via `SNOWFLAKE.CORTEX.FINETUNE`), Streamlit chat patterns, Snowflake Native Apps distribution, rate-limit handling with backoff, cost controls (caching, max_tokens, model downsize, QUERY_TAG), conversation memory (short-term, summary, semantic), structured JSON output, function calling via COMPLETE `tools` option, continuous eval harnesses, defense-in-depth checklist.

## Suggested Study Order

1. **01 — Cortex LLM Functions** first. Every other domain calls COMPLETE/EMBED/etc. under the hood. Get the function signatures and model catalog memorized before anything else.
2. **02 — Cortex Search & RAG** next. The retrieval primitives feed everything downstream (RAG, Analyst hybrid dim lookup, Agents).
3. **03 — Cortex Analyst**. The semantic model YAML is where exam traps live; author one by hand before taking the exam.
4. **04 — Document AI**. Smallest surface area; study after you understand the stage/stream/task plumbing.
5. **05 — AI & ML Studio & Notebooks**. Mostly operational; most exam questions here are "where do I build X."
6. **06 — Gen AI App Patterns & Governance**. Synthesizes everything. Save for last; it is also the chapter with the most scenario questions.

Use this repo in three passes:
- **Pass 1 (concept pass):** read each `_QA.md` top to bottom, skipping lab exercises.
- **Pass 2 (practice pass):** run the lab exercises in a scratch Snowflake account; take flashcards and the mock exam.
- **Pass 3 (polish pass):** re-read "Gotchas / Exam Traps" in each file; drill the cheat sheet; retake mock exam.

## Companion Links

- `01_cortex_llm_functions_QA.md` — Cortex LLM function reference deep-dive.
- `02_cortex_search_and_rag_QA.md` — Cortex Search + canonical RAG pipeline.
- `03_cortex_analyst_QA.md` — Semantic model authoring + REST API.
- `04_document_ai_QA.md` — Arctic-TILT + extraction pipelines.
- `05_ai_ml_studio_and_notebooks_QA.md` — Notebooks, runtimes, Registry, Feature Store.
- `06_gen_ai_app_patterns_QA.md` — Prompting, Fine-Tuning, Agents, Streamlit, governance.
- `GES-C01_FLASHCARDS.md` — 80–120 spaced-repetition flashcards across all six domains.
- `GES-C01_MOCK_EXAM.md` — 65-question mock exam mirroring the real blueprint.
- `GES-C01_CHEATSHEET.md` — dense printable reference for the week before the exam.

## Tips for a Cortex-Heavy Exam

**Memorize function signatures.** The exam loves "which argument is wrong" questions. Know `SNOWFLAKE.CORTEX.COMPLETE(model, prompt_or_messages, options)`, the two return shapes (VARCHAR vs OBJECT), the options keys (`temperature`, `top_p`, `max_tokens`, `guardrails`, `response_format`, `tools`), and that `TRY_COMPLETE` returns NULL on error.

**Keep the model catalog in your head.** Mistral family (7b, large2), Llama 3.1 family (8b, 70b, 405b), Llama 3.2 vision (11b-vision, 90b-vision), Snowflake Arctic (SQL/code-tuned), Reka (Core, Flash), Jamba (Instruct, 1.5-large with 256k context), Claude via Cortex. Know who wins at long context (Jamba), vision (Llama 3.2 vision), SQL/code (Arctic), raw quality (Llama 3.1 405b).

**Embedding model and dimension pairs.** `snowflake-arctic-embed-m-v1.5` → 768-d, English, fast. `snowflake-arctic-embed-l-v2.0` → 1024-d, multilingual, best default. `voyage-multilingual-2` → 1024-d multilingual. `nv-embed-qa-4` → 1024-d QA-tuned. `e5-base-v2` → 768-d English baseline. Mixing dims in one similarity op errors out.

**Cortex Guard categories.** Hate, violence, self-harm, sexual content, criminal planning, CBRN. Guard screens output, is activated by `{'guardrails': true}` on COMPLETE, and blocked calls still consume tokens.

**Semantic model YAML keys.** `name`, `description`, `tables[].base_table`, `primary_key`, `dimensions` (`name`, `synonyms`, `sample_values`, `expr`, `data_type`, `cortex_search_service`), `measures` (`default_aggregation`), `time_dimensions`, `filters`, `relationships` (`left_table`, `right_table`, `relationship_columns`, `join_type`, `relationship_type`), `verified_queries`, `custom_instructions`.

**Governance mantra.** Masking + row access policies apply at **execution** time, so the LLM/Analyst/Search never bypasses them. The semantic model YAML's `sample_values` CAN leak otherwise-masked literals; choose non-sensitive examples.

**Observability views.** `SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTIONS_USAGE_HISTORY`, `CORTEX_FUNCTIONS_QUERY_USAGE_HISTORY`, `DOCUMENT_AI_USAGE_HISTORY`, `QUERY_HISTORY`, `ACCESS_HISTORY`. For near-real-time, use the `INFORMATION_SCHEMA` table functions.

**Cost model.** Input tokens cheap, output tokens 1x–4x more expensive (and 405b has an 11x output premium). Decode is sequential and dominates latency and cost; prompt tokens prefill in parallel. Streaming improves perceived latency, not cost.

**Scenario answer heuristic.** For "which approach" questions, ask: (1) is the data structured or unstructured; (2) does the task need the base model's knowledge or your proprietary knowledge; (3) do we need format change (fine-tune) or knowledge grounding (RAG) or just prompting; (4) is governance in play (always say yes). Snowflake-native over external APIs by default.

**Watch for the "I don't know" option.** Analyst refuses out-of-scope questions by design. Treat refusals as a feature, not a bug. Logging + feedback drives the next YAML update.

**Pin versions, always.** Model names, semantic model files, Document AI model versions, Registry model versions. "latest" references are a prod outage waiting to happen.

**Do the labs.** The hands-on exercises in each `_QA.md` file take ~2 hours total and will cement the APIs that scenario questions ask about.

Good luck.
