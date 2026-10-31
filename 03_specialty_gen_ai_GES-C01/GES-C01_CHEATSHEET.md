# GES-C01 Cheat Sheet — Snowflake Specialty: Generative AI

Dense printable reference. Print at 100% to 2–3 pages. All syntax tested against the sources in `01–06_*_QA.md`. Confirm current rates and model availability in Snowflake docs before an exam attempt.

---

## 1. Cortex LLM Function Signatures

### COMPLETE
```sql
SNOWFLAKE.CORTEX.COMPLETE(
    model          VARCHAR,          -- or fine-tuned-model-name
    prompt_or_msgs,                  -- VARCHAR OR ARRAY of {role, content}
    options        OBJECT            -- optional; triggers OBJECT return
) RETURNS VARCHAR | OBJECT
-- Options keys: temperature, top_p, max_tokens, guardrails,
--               response_format, tools, tool_choice
-- OBJECT return (messages-form or options present):
--   { choices:[{messages:"..."}],
--     usage:{prompt_tokens, completion_tokens, total_tokens} }
```

### TRY_COMPLETE
```sql
SNOWFLAKE.CORTEX.TRY_COMPLETE(model, prompt_or_msgs [, options])
-- Same signature as COMPLETE; returns NULL on recoverable errors
-- (quota, guardrail block). Permanent errors still raise.
```

### Task-specific helpers (no model argument, Snowflake-chosen internally)
```sql
SNOWFLAKE.CORTEX.SUMMARIZE(text)                           -- generic summary
SNOWFLAKE.CORTEX.TRANSLATE(text, src_lang, tgt_lang)       -- ISO 639-1 codes
SNOWFLAKE.CORTEX.SENTIMENT(text)                           -- FLOAT in [-1,1]
SNOWFLAKE.CORTEX.EXTRACT_ANSWER(from_text, question)       -- ARRAY of {answer,score}
SNOWFLAKE.CORTEX.CLASSIFY_TEXT(text, [labels])             -- {label, confidence}
```

### AISQL family (declarative, newer)
```sql
AI_COMPLETE(text, options)                                 -- COMPLETE with task metadata
AI_CLASSIFY(text, [labels], options)                       -- multi-label, few-shot
AI_FILTER(instruction, text)                               -- BOOLEAN for WHERE
AI_AGG(instruction, column) OVER GROUP BY ...              -- aggregate over group
AI_SUMMARIZE_AGG(text) OVER GROUP BY ...                   -- group-level summary
AI_SIMILARITY(a, b)                                        -- semantic similarity
```

### Embeddings
```sql
SNOWFLAKE.CORTEX.EMBED_TEXT_768(model, text)   -- VECTOR(FLOAT, 768)
SNOWFLAKE.CORTEX.EMBED_TEXT_1024(model, text)  -- VECTOR(FLOAT, 1024)

-- Vector distance (use cosine as default)
VECTOR_COSINE_SIMILARITY(a, b)  -- higher = more similar, [-1, 1]
VECTOR_L2_DISTANCE(a, b)        -- lower  = more similar, [0, inf)
VECTOR_INNER_PRODUCT(a, b)      -- == cosine when unit-norm
```

### Document & vision
```sql
SNOWFLAKE.CORTEX.PARSE_DOCUMENT(@stage, 'file.pdf', {'mode':'LAYOUT'|'OCR'})
-- Returns OBJECT { content:"...", pages:[...], metadata:{} }

-- Vision model via COMPLETE
SNOWFLAKE.CORTEX.COMPLETE(
  'llama3.2-90b-vision',
  [{'role':'user','content':[
     {'type':'text','text':'Describe this image'},
     {'type':'image_url','image_url':{'url': BUILD_SCOPED_FILE_URL(@stage,'img.png')}}
  ]}]
);
```

### Cross-Region Inference (CRI)
```sql
ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'ANY_REGION';
-- or restrict:
ALTER ACCOUNT SET CORTEX_ENABLED_CROSS_REGION = 'AWS_US';
-- Default: DISABLED. Changes region routing but not billing account.
```

### Example COMPLETE calls

```sql
-- Simple
SELECT SNOWFLAKE.CORTEX.COMPLETE('mistral-large2', 'Define RAG');

-- Chat with structured JSON output
SELECT SNOWFLAKE.CORTEX.COMPLETE(
  'llama3.1-70b',
  [
    {'role':'system','content':'Extract fields. JSON only.'},
    {'role':'user',  'content':'John ordered 2 widgets @ $9.99'}
  ],
  {
    'temperature':0,
    'max_tokens':200,
    'guardrails':true,
    'response_format':{'type':'json','schema':{
      'type':'object',
      'properties':{
        'customer':{'type':'string'},
        'items':{'type':'array','items':{
          'type':'object',
          'properties':{'name':{'type':'string'},'qty':{'type':'integer'},'price':{'type':'number'}},
          'required':['name','qty','price']}}
      },
      'required':['customer','items']
    }}
  }
);

-- Guard output screening (shortcut)
SELECT SNOWFLAKE.CORTEX.COMPLETE('llama3.1-70b', $q, {'guardrails':true});
```

---

## 2. Model Catalog

| Model | Context | Family / Note | Best for | Relative cost |
|---|---|---|---|---|
| `mistral-7b` | 32k | Mistral, dense | Cheap classify/short answer | $ |
| `mistral-large2` | 128k | Mistral, dense | Instruction following, multilingual | $$$ |
| `llama3.1-8b` | 128k | Meta, dense | Cheap bulk, classify, short gen | $ |
| `llama3.1-70b` | 128k | Meta, dense | Mid-tier reasoning, RAG default | $$ |
| `llama3.1-405b` | 128k | Meta, dense | Highest quality; 11x output premium | $$$$$ |
| `llama3.2-11b-vision` | 128k | Meta, vision | Image + text, cheap vision | $$ |
| `llama3.2-90b-vision` | 128k | Meta, vision | Image + text, better vision | $$$ |
| `snowflake-arctic` | 4k–32k | Snowflake MoE | SQL + code generation | $$ |
| `reka-flash` | ~128k | Reka | Cheap multilingual general | $ |
| `reka-core` | ~128k | Reka | High-quality multilingual | $$$$ |
| `jamba-instruct` | 256k | AI21 Mamba+Transformer | Long context cheap | $$ |
| `jamba-1.5-large` | 256k | AI21 Mamba+Transformer | Long context, strong reasoning | $$$$ |
| `claude-*` (where available) | varies | Anthropic via Cortex | Strong general / coding | varies |

Rules of thumb:
- Need vision → `llama3.2-11b-vision` or `-90b-vision`.
- Need >128k context → Jamba family.
- Need SQL/code → `snowflake-arctic`.
- Highest open-weight quality → `llama3.1-405b` (expensive on output).
- Multilingual → `mistral-large2`, `reka-core`.
- Cheap classify/extract → `llama3.1-8b` or `mistral-7b`.

Credits per million tokens (illustrative — verify current rates):

| Model | INPUT | OUTPUT |
|---|---|---|
| `snowflake-arctic` | 0.84 | 0.84 |
| `llama3.1-8b` | 0.19 | 0.19 |
| `llama3.1-70b` | 1.21 | 1.21 |
| `llama3.1-405b` | 3.00 | 11.00 (output premium) |
| `mistral-7b` | 0.12 | 0.12 |
| `mistral-large2` | 1.95 | 1.95 |
| `reka-flash` | 0.45 | 0.45 |
| `reka-core` | 5.50 | 5.50 |
| `jamba-instruct` | 0.83 | 0.83 |
| `jamba-1.5-large` | 5.00 | 5.00 |

---

## 3. EMBED_TEXT Models

| Model | Dim | Lang | Notes |
|---|---|---|---|
| `e5-base-v2` | 768 | English | Cheap generic baseline |
| `snowflake-arctic-embed-m-v1.5` | 768 | English | Retrieval-tuned, fast |
| `snowflake-arctic-embed-l-v2.0` | 1024 | Multilingual | **Default flagship** for Cortex Search |
| `voyage-multilingual-2` | 1024 | Multilingual | Strong multilingual |
| `nv-embed-qa-4` | 1024 | English | QA-tuned (query↔passage) |

Rules:
- Embedding model is **immutable** on an existing Cortex Search service — recreate to change.
- Dimensions must match for similarity ops; mixing errors.
- Embedders truncate silently past ~512 tokens; chunk before embedding.
- Billing is input-tokens-only (no decode step).

---

## 4. Semantic Model YAML Template

```yaml
name: enterprise_sales_v3
description: Enterprise sales analytics

custom_instructions: |
  - Exclude test accounts: WHERE customer_id NOT LIKE 'TEST%'
  - Fiscal year starts Feb 1
  - Default currency is USD

tables:
  - name: SALES
    base_table:
      database: PROD
      schema: SALES
      table: FACT_ORDERS
    primary_key:
      columns: [order_id]
    dimensions:
      - name: region
        synonyms: [area, territory, zone]
        sample_values: [AMER, EMEA, APAC, LATAM]
        expr: region_code
        data_type: VARCHAR
        description: 3-char region code
      - name: customer_name
        expr: name
        cortex_search_service:
          service: "@PROD.CRM.CUST_SRCH"
          literal_column: name
    time_dimensions:
      - name: order_date
        expr: order_dt
        data_type: DATE
        grain: day
    measures:
      - name: revenue
        expr: amount_usd
        data_type: NUMBER
        default_aggregation: sum
        description: Gross revenue in USD
      - name: margin_pct
        expr: SUM(amount_usd - cogs_usd) / NULLIF(SUM(amount_usd), 0)
        default_aggregation: none       # pre-aggregated ratio
    filters:
      - name: shipped
        expr: "status = 'SHIPPED'"
      - name: non_test
        expr: "customer_id NOT LIKE 'TEST%'"

  - name: CUSTOMERS
    base_table: {database: PROD, schema: CRM, table: DIM_CUSTOMER}
    primary_key: {columns: [customer_id]}
    dimensions:
      - name: segment
        expr: segment
        sample_values: [SMB, MID, ENT]

relationships:
  - name: sales_to_customer
    left_table: SALES
    right_table: CUSTOMERS
    relationship_columns:
      - left_column: customer_id
        right_column: customer_id
    join_type: left_outer
    relationship_type: many_to_one

verified_queries:
  - name: fy_rev_by_region
    question: "What was fiscal YTD revenue by region?"
    sql: |
      SELECT region_code, SUM(amount_usd) AS revenue
      FROM prod.sales.fact_orders
      WHERE order_dt >= DATE_FROM_PARTS(
          CASE WHEN MONTH(CURRENT_DATE) < 2
               THEN YEAR(CURRENT_DATE)-1 ELSE YEAR(CURRENT_DATE) END, 2, 1)
        AND customer_id NOT LIKE 'TEST%'
      GROUP BY region_code
    verified_at: "2025-02-01"
    verified_by: "jane@acme.com"
```

Deploy:
```sql
PUT file:///local/enterprise_sales_v3.yaml @MYDB.MYSCHEMA.MODELS AUTO_COMPRESS=FALSE OVERWRITE=TRUE;
-- Reference as @MYDB.MYSCHEMA.MODELS/enterprise_sales_v3.yaml in Analyst calls.
```

Measure aggregation rules:
- Additive (revenue, qty): `sum`
- Averages that need weighting: `none` + explicit formula
- Ratios/percentages: `none` with `SUM(num) / NULLIF(SUM(den), 0)`
- Counts: `count` or `count_distinct`

---

## 5. Cortex Search Service DDL

```sql
-- Source chunked table (populate first)
CREATE OR REPLACE TABLE chunks (
  chunk_id STRING,
  doc_id   STRING,
  title    STRING,
  category STRING,
  content  STRING,
  updated  TIMESTAMP
);

-- Service
CREATE OR REPLACE CORTEX SEARCH SERVICE support_kb
  ON content                                -- the text column to index
  ATTRIBUTES title, category, doc_id        -- filter-able columns
  WAREHOUSE = cs_wh                         -- for ingestion/refresh
  TARGET_LAG = '15 minutes'                 -- freshness SLA
  EMBEDDING_MODEL = 'snowflake-arctic-embed-l-v2.0'
  AS (SELECT chunk_id, doc_id, title, category, content, updated FROM chunks);

GRANT USAGE ON CORTEX SEARCH SERVICE support_kb TO ROLE APP_RW;

-- Query via SQL
SELECT SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
  'support_kb',
  OBJECT_CONSTRUCT(
    'query','how do I reset my password',
    'columns', ARRAY_CONSTRUCT('title','content'),
    'filter', OBJECT_CONSTRUCT(
      '@and', ARRAY_CONSTRUCT(
        OBJECT_CONSTRUCT('@eq',  OBJECT_CONSTRUCT('category','auth')),
        OBJECT_CONSTRUCT('@gte', OBJECT_CONSTRUCT('updated','2025-01-01'))
      )
    ),
    'limit', 5
  )::STRING
);

-- Alterable without re-embed: TARGET_LAG, WAREHOUSE.
-- NOT alterable: EMBEDDING_MODEL (recreate to change).

-- Health
DESC CORTEX SEARCH SERVICE support_kb;
SELECT SYSTEM$GET_CORTEX_SEARCH_SERVICE_STATUS('support_kb');
```

Filter JSON operators: `@eq`, `@contains`, `@gte`, `@lte`, `@gt`, `@lt`, `@and`, `@or`, `@not`, `@primarykey`. Filters are JSON over ATTRIBUTE columns only (not non-attribute columns).

### SPLIT_TEXT helper (for chunking before indexing)

```sql
SELECT
  d.id AS doc_id,
  c.index::INT AS chunk_idx,
  c.value::STRING AS chunk_text
FROM docs d,
     LATERAL FLATTEN(
       input => SNOWFLAKE.CORTEX.SPLIT_TEXT_RECURSIVE_CHARACTER(
         d.body,
         'markdown',   -- format: 'none', 'markdown'
         1500,         -- chunk_size (chars)
         200           -- overlap   (chars)
       )
     ) c;
```

---

## 6. Cortex Analyst REST API Payload

Endpoint: `POST /api/v2/cortex/analyst/message`
Auth: Snowflake OAuth or key-pair JWT.

```json
{
  "messages": [
    {"role":"user","content":[
      {"type":"text","text":"What was Q3 revenue by region last year?"}
    ]}
  ],
  "semantic_model_file": "@MYDB.MYSCHEMA.MODELS/sales_v3.yaml",
  "stream": false
}
```

Response:
```json
{
  "request_id":"uuid",
  "message":{
    "role":"assistant",
    "content":[
      {"type":"text","text":"Here is Q3 revenue by region..."},
      {"type":"sql","statement":"SELECT region, SUM(amount) ..."},
      {"type":"suggestions","suggestions":["Compare to last quarter","Break down by product"]}
    ]
  },
  "warnings":[]
}
```

Multi-turn — append prior assistant turns to `messages`:
```json
{
  "messages":[
    {"role":"user","content":[{"type":"text","text":"revenue by region last quarter"}]},
    {"role":"assistant","content":[
      {"type":"sql","statement":"SELECT region, SUM(amount) ..."},
      {"type":"text","text":"Here is Q3 revenue by region..."}
    ]},
    {"role":"user","content":[{"type":"text","text":"and by product"}]}
  ],
  "semantic_model_file":"@..."
}
```

Streamlit-in-Snowflake call:
```python
import _snowflake, json
resp = _snowflake.send_snow_api_request(
    "POST", "/api/v2/cortex/analyst/message",
    {}, {},
    {"messages": st.session_state.msgs,
     "semantic_model_file": "@MYDB.MYSCHEMA.MODELS/sales_v3.yaml"},
    None, 30000)   # timeout ms
body = json.loads(resp["content"])
msg = body["message"]
sql = next((c["statement"] for c in msg["content"] if c["type"]=="sql"), None)
if sql:
    df = session.sql(sql).to_pandas()  # app executes under caller role
```

---

## 7. Document AI PREDICT Syntax

```sql
-- One file
SELECT my_db.my_schema.INVOICE_EXTRACTOR!PREDICT(
  GET_PRESIGNED_URL(@docs_stage, 'invoice_sample.pdf', 3600),  -- 1-hr URL
  1                                                             -- model version
) AS extracted;

-- Many files via directory table
WITH files AS (
  SELECT relative_path,
         GET_PRESIGNED_URL(@docs_stage, relative_path, 3600) AS url
  FROM DIRECTORY(@docs_stage)
  WHERE relative_path ILIKE '%.pdf'
)
SELECT f.relative_path,
       my_db.my_schema.INVOICE_EXTRACTOR!PREDICT(f.url, 1) AS r,
       r:invoice_no.value::STRING AS inv_no,
       r:invoice_no.score::FLOAT  AS inv_no_conf,
       r:total.value::NUMBER(18,2) AS total,
       r:__documentMetadata.ocrScore::FLOAT AS ocr_conf
FROM files f;
```

Return shape:
```json
{
  "__documentMetadata": {"ocrScore": 0.97},
  "invoice_no": {"value":"12345", "score":0.98},
  "vendor":     {"value":"Acme Corp", "score":0.94},
  "date":       {"value":"2025-10-02", "score":0.91},
  "total":      {"value":429.50, "score":0.96},
  "line_items": [
    {"description":{"value":"Widget","score":0.92},
     "qty":{"value":3,"score":0.98}}
  ]
}
```

Required grants:
```sql
GRANT USAGE   ON MODEL   my_db.my_schema.INVOICE_EXTRACTOR TO ROLE AP_READ;
GRANT READ    ON STAGE   my_db.my_schema.docs_stage        TO ROLE AP_READ;
GRANT USAGE   ON FUNCTION GET_PRESIGNED_URL(VARCHAR,VARCHAR,NUMBER) TO ROLE AP_READ;
```

Pipeline pattern (directory table + stream + task):
```sql
CREATE OR REPLACE STAGE incoming_docs
  DIRECTORY = (ENABLE = TRUE AUTO_REFRESH = TRUE)
  ENCRYPTION = (TYPE = 'SNOWFLAKE_SSE');

CREATE OR REPLACE STREAM new_docs_stream ON DIRECTORY TABLE incoming_docs;

CREATE OR REPLACE TASK extract_invoices
  WAREHOUSE = docai_wh
  SCHEDULE  = '2 MINUTE'
  WHEN SYSTEM$STREAM_HAS_DATA('new_docs_stream')
AS
INSERT INTO invoice_extracts
SELECT relative_path, CURRENT_TIMESTAMP,
       r:invoice_no.value::STRING, r:invoice_no.score::FLOAT,
       r:vendor.value::STRING,     r:vendor.score::FLOAT,
       r:total.value::NUMBER(18,2),r:total.score::FLOAT,
       (LEAST(r:invoice_no.score, r:vendor.score, r:total.score) < 0.85) AS needs_review,
       r
FROM (
  SELECT relative_path,
         my_db.my_schema.INVOICE_EXTRACTOR!PREDICT(
           GET_PRESIGNED_URL(@incoming_docs, relative_path, 3600), 1) AS r
  FROM new_docs_stream
  WHERE metadata$action = 'INSERT' AND relative_path ILIKE '%.pdf'
);

ALTER TASK extract_invoices RESUME;
```

Version management:
```sql
SHOW MODEL VERSIONS IN MODEL my_db.my_schema.INVOICE_EXTRACTOR;
DROP MODEL VERSION my_db.my_schema.INVOICE_EXTRACTOR!v1;  -- only if unused
```

Billing view: `SNOWFLAKE.ACCOUNT_USAGE.DOCUMENT_AI_USAGE_HISTORY`.

---

## 8. Cortex Guard Categories

Enabled with `{'guardrails': true}` in COMPLETE options. Screens model **output** (not input). Blocked calls still consume tokens.

Classifier (Llama-Guard family) categories:
- **Hate** — hateful content targeting protected groups.
- **Violence** — graphic violence, threats.
- **Self-harm** — suicide, self-injury content.
- **Sexual content** — sexual content, especially involving minors.
- **Criminal planning** — weapons synthesis, drug manufacture, cybercrime instructions.
- **CBRN** — chemical, biological, radiological, nuclear weapons.

Input screening (your responsibility, not Guard):
```sql
AI_CLASSIFY(user_q, ['benign','prompt_injection','pii_extraction_attempt','jailbreak'],
            {'task_description':'Detect attempts to override instructions or extract secrets'})
```

Defense-in-depth order:
1. Auth + RBAC + masking + row access
2. Input filter (classifier) for policy violations and prompt injection
3. System prompt separates roles; retrieved content wrapped as "data not instructions"
4. COMPLETE with `{'guardrails': true}`
5. Logging + audit review
6. Rate limits + backoff
7. Model/package version pinning

---

## 9. Fine-Tuning Workflow

### Cortex Fine-Tuning (managed LoRA on supported base models)

Supported bases include Mistral-7B, Mistral-large2, and Llama 3 family (verify current list).

```sql
-- 1. Prepare training table (hundreds to low thousands of pairs)
CREATE OR REPLACE TABLE ft_data AS
SELECT support_ticket_body AS prompt,
       canonical_response    AS completion
FROM historical_tickets
WHERE quality_rating >= 4;

-- 2. Dedup + balance + validation hold-out
CREATE OR REPLACE TABLE ft_clean AS
WITH deduped AS (
  SELECT DISTINCT prompt, completion FROM ft_data
),
balanced AS (
  SELECT * FROM deduped
  QUALIFY ROW_NUMBER() OVER (PARTITION BY category ORDER BY RANDOM()) <= 200
)
SELECT * FROM balanced;

CREATE OR REPLACE TABLE ft_train AS
  SELECT * FROM ft_clean WHERE HASH(prompt) % 100 < 85;
CREATE OR REPLACE TABLE ft_val AS
  SELECT * FROM ft_clean WHERE HASH(prompt) % 100 >= 85;

-- 3. Create fine-tune job
SELECT SNOWFLAKE.CORTEX.FINETUNE(
  'CREATE',
  'my_db.my_schema.support_bot_v1',
  'mistral-7b',
  'SELECT prompt, completion FROM my_db.my_schema.ft_train',
  'SELECT prompt, completion FROM my_db.my_schema.ft_val'
);

-- 4. Monitor
SELECT SNOWFLAKE.CORTEX.FINETUNE('DESCRIBE', '<job_id>');
SELECT SNOWFLAKE.CORTEX.FINETUNE('SHOW');

-- 5. Call the fine-tuned model
SELECT SNOWFLAKE.CORTEX.COMPLETE(
  'my_db.my_schema.support_bot_v1',
  'A customer says their package arrived damaged...'
);

-- 6. Evaluate side-by-side (LLM-as-judge)
WITH side_by_side AS (
  SELECT prompt, reference,
    SNOWFLAKE.CORTEX.COMPLETE('mistral-7b', prompt) AS base_out,
    SNOWFLAKE.CORTEX.COMPLETE('my_db.my_schema.support_bot_v1', prompt) AS tuned_out
  FROM eval_set
)
SELECT prompt,
  SNOWFLAKE.CORTEX.COMPLETE('mistral-large2',
    CONCAT('Which better matches reference?\nREF: ', reference,
           '\nA: ', base_out, '\nB: ', tuned_out, '\nReply A/B/Tie.'),
    {'temperature':0}) AS verdict
FROM side_by_side;
```

### Document AI fine-tuning (Snowsight UI)

1. Build → upload samples → define schema (fields = NL questions + types)
2. Review predictions on each sample, correct errors
3. Click "Train" (minimum ~20 reviewed docs)
4. Compare metrics on hold-out
5. Publish next version
6. Pin pipeline to new version number; keep previous for rollback

### When to fine-tune vs other options

| Symptom | Primary fix |
|---|---|
| Model doesn't know your product catalog | **RAG** (not fine-tune) |
| Format / style drift | **Fine-tune** |
| Needs specialized domain terminology | **Fine-tune** + glossary in prompt |
| Model slow / expensive | **Smaller model + fine-tune** |
| New business rule weekly | **Prompt eng** (modular) |
| Dates / math wrong | **CoT prompt** or **tool use** |

Fine-tuning does NOT inject knowledge well — hallucinations get baked in. Use RAG for knowledge; fine-tune for behavior.

---

## 10. RAG App Skeleton (Streamlit in Snowflake)

```python
import streamlit as st
import json
from snowflake.snowpark.context import get_active_session
from snowflake.cortex import complete

session = get_active_session()

st.set_page_config(page_title="Support Copilot", page_icon=":robot_face:")
st.title("Support Copilot")

# --- Chat memory ---
if "chat" not in st.session_state:
    st.session_state.chat = []

# --- Render existing turns ---
for msg in st.session_state.chat:
    with st.chat_message(msg["role"]):
        st.markdown(msg["content"])
        if msg.get("sources"):
            with st.expander("Sources"):
                for s in msg["sources"]:
                    st.markdown(f"**{s['title']}** (score {s['_score']:.2f})")
                    st.text(s["content"][:300] + "...")

# --- New question ---
if q := st.chat_input("Ask a question"):
    st.session_state.chat.append({"role": "user", "content": q})

    # 1. Retrieve top-5 chunks from Cortex Search
    hits_json = session.sql(
        """SELECT SNOWFLAKE.CORTEX.SEARCH_PREVIEW(
             'support_kb',
             OBJECT_CONSTRUCT(
               'query', ?,
               'columns', ARRAY_CONSTRUCT('title','content'),
               'limit', 5
             )::STRING
           )""",
        params=[q],
    ).collect()[0][0]
    hits = json.loads(hits_json).get("results", [])

    # 2. Assemble grounded prompt
    context = "\n\n---\n\n".join(
        f"[{h['title']}]\n{h['content']}" for h in hits
    )
    messages = [
        {"role": "system", "content":
            "Answer ONLY from the provided context. Cite [title] after each claim. "
            "If the context does not contain the answer, say you don't know."},
        {"role": "user", "content": f"Context:\n{context}\n\nQ: {q}"},
    ]

    # 3. Generate with guardrails and deterministic temperature
    reply = complete(
        model="llama3.1-70b",
        prompt=messages,
        options={"guardrails": True, "temperature": 0, "max_tokens": 800},
        session=session,
    )

    # 4. Append and render
    st.session_state.chat.append(
        {"role": "assistant", "content": reply, "sources": hits}
    )
    st.rerun()

# --- Logging (recommended, async if possible) ---
# INSERT INTO llm_trace (ts, user_name, app, model, user_prompt,
#                        retrieved_context, response, guardrails_triggered)
# VALUES (CURRENT_TIMESTAMP, CURRENT_USER(), 'support-copilot', 'llama3.1-70b',
#         :q, :hits, :reply, FALSE);
```

Trace log table:
```sql
CREATE OR REPLACE TABLE llm_trace (
  trace_id STRING DEFAULT UUID_STRING(),
  ts TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  user_name STRING,
  app STRING,
  model STRING,
  system_prompt STRING,
  user_prompt STRING,
  retrieved_context ARRAY,
  response STRING,
  latency_ms INT,
  prompt_tokens INT,
  completion_tokens INT,
  cost_credits NUMBER,
  guardrails_triggered BOOLEAN,
  feedback_score INT,
  feedback_notes STRING
);
```

### Cortex Agent payload (multi-tool orchestration)

```python
import _snowflake, json

payload = {
    "model": "llama3.1-70b",
    "messages": st.session_state.msgs,
    "tools": [
        {"tool_spec": {"type": "cortex_analyst_text_to_sql", "name": "sales"}},
        {"tool_spec": {"type": "cortex_search",             "name": "complaints"}},
        {"tool_spec": {"type": "generic",                   "name": "call_vendor_api",
            "description": "Fetch vendor shipping status by order id.",
            "input_schema": {"type":"object",
                             "properties":{"order_id":{"type":"string"}},
                             "required":["order_id"]}}},
    ],
    "tool_resources": {
        "sales":      {"semantic_model_file": "@MY.SEM/sales.yaml"},
        "complaints": {"name": "MY.SEARCH.COMPLAINTS_SRCH"},
    },
}
r = _snowflake.send_snow_api_request(
    "POST", "/api/v2/cortex/agent:run", {}, {}, payload, None, 60000)
```

---

## 11. Observability & Governance Quick Reference

### Usage views
```sql
-- Cortex LLM per-call rows
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTIONS_USAGE_HISTORY
WHERE start_time >= DATEADD(day, -7, CURRENT_DATE);

-- Cortex LLM aggregated to query_id (join to QUERY_HISTORY for QUERY_TAG)
SELECT PARSE_JSON(qh.query_tag):app::STRING AS app,
       SUM(ch.token_credits)                AS credits
FROM SNOWFLAKE.ACCOUNT_USAGE.CORTEX_FUNCTIONS_QUERY_USAGE_HISTORY ch
JOIN SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY qh USING (query_id)
WHERE ch.start_time >= DATEADD(day, -30, CURRENT_DATE)
GROUP BY 1 ORDER BY credits DESC;

-- Document AI
SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.DOCUMENT_AI_USAGE_HISTORY
WHERE start_time >= DATEADD(day, -7, CURRENT_DATE);

-- Near-real-time (no 3h lag)
SELECT * FROM TABLE(INFORMATION_SCHEMA.CORTEX_FUNCTIONS_USAGE_HISTORY(
    DATE_RANGE_START=>DATEADD(hour,-1,CURRENT_TIMESTAMP),
    DATE_RANGE_END=>CURRENT_TIMESTAMP));
```

### Showback setup
```sql
ALTER SESSION SET QUERY_TAG = '{"app":"support-copilot","team":"cs","env":"prod"}';
-- all subsequent Cortex calls carry this tag
```

### Resource monitor on an ML warehouse
```sql
CREATE OR REPLACE RESOURCE MONITOR ml_rm
  CREDIT_QUOTA = 500 FREQUENCY = MONTHLY
  START_TIMESTAMP = CURRENT_TIMESTAMP
  TRIGGERS
    ON 80 PERCENT DO NOTIFY
    ON 100 PERCENT DO SUSPEND;
ALTER WAREHOUSE ml_wh SET RESOURCE_MONITOR = ml_rm;
```

### Common grants
```sql
GRANT DATABASE ROLE SNOWFLAKE.CORTEX_USER TO ROLE ANALYST;

GRANT USAGE ON CORTEX SEARCH SERVICE support_kb TO ROLE APP_RW;

GRANT USAGE ON MODEL  my_db.my_schema.INVOICE_EXTRACTOR TO ROLE AP_READ;
GRANT READ  ON STAGE  my_db.my_schema.incoming_docs     TO ROLE AP_READ;

GRANT USAGE   ON NOTEBOOK my_db.ml.training TO ROLE DS_READER;
GRANT OPERATE ON NOTEBOOK my_db.ml.training TO ROLE DS_RUNNER;
GRANT USAGE   ON COMPUTE POOL ml_gpu        TO ROLE DS_RUNNER;

-- External Access for 3rd-party APIs
CREATE OR REPLACE SECRET openai_key TYPE = GENERIC_STRING SECRET_STRING = 'sk-...';
CREATE OR REPLACE NETWORK RULE openai_rule TYPE = HOST_PORT
  VALUE_LIST = ('api.openai.com:443') MODE = EGRESS;
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION openai_eai
  ALLOWED_NETWORK_RULES = (openai_rule)
  ALLOWED_AUTHENTICATION_SECRETS = (openai_key)
  ENABLED = TRUE;
GRANT USAGE ON INTEGRATION openai_eai TO ROLE DS;
GRANT READ  ON SECRET openai_key      TO ROLE DS;
ALTER NOTEBOOK my_db.ml.external_calls
  SET EXTERNAL_ACCESS_INTEGRATIONS = (openai_eai)
      SECRETS = ('openai_key' = openai_key);
```

---

## 12. Scenario Decision Trees

### "Which service?"
```mermaid
flowchart TD
    Q[Does task need YOUR warehouse data?]
    Q -->|No| V[Well-known verb?<br/>summarize/translate/sentiment]
    V -->|Yes| H[Cortex helper:<br/>SUMMARIZE / TRANSLATE / SENTIMENT /<br/>EXTRACT_ANSWER / CLASSIFY_TEXT / AI_CLASSIFY]
    V -->|No| CMP[COMPLETE with prompt engineering]

    Q -->|Yes| S[Data structured?<br/>tables, metrics]
    S -->|Yes| AN[Cortex Analyst<br/>+ semantic model YAML]
    S -->|No| F[Forms with fixed layout?<br/>invoices, claims]
    F -->|Yes| DA[Document AI<br/>schema -> !PREDICT]
    F -->|No| RAG[Cortex Search + COMPLETE - RAG]
    RAG --> FT[Need style/format change? -> + Fine-tune]
    RAG --> AG[Multi-source blend? -> Cortex Agents]
```

### "Which model for COMPLETE?"

| Need | Pick |
|---|---|
| Vision | `llama3.2-11b-vision` / `llama3.2-90b-vision` |
| >128k context | `jamba-instruct` / `jamba-1.5-large` |
| SQL / code generation | `snowflake-arctic` |
| Multilingual general | `mistral-large2` / `reka-core` |
| Highest open-weight quality | `llama3.1-405b` (watch 11x output premium) |
| Mid-tier RAG default | `llama3.1-70b` |
| Bulk classify / short gen | `llama3.1-8b` / `mistral-7b` / `reka-flash` |
| Snowflake fine-tunable | `mistral-7b` / `mistral-large2` / `llama3` family |

### "Pipeline cost dominators?"

| Component | Notes |
|---|---|
| `embed_tokens * embed_rate` | One-time + refresh on CDC |
| `per_query search_credits` | Small per call |
| `prompt_tokens * model_input_rate` | Retrieval payload + system/user |
| `output_tokens * model_output_rate` | Dominant for long outputs; 11x on 405b |
| `warehouse_credit_seconds` | Orchestration; small |
| `guardrails_classifier_pass` | Small |
| `CRI_network_hop` | 50-150ms, no extra credit |

---

## 13. Last-Minute Exam Reminders

1. **Masking + row access policies apply at execution** — the LLM never bypasses them.
2. **Analyst returns SQL**; your app executes it. **Agents** execute tool calls on their own.
3. **CRI is ACCOUNT level**; off by default; `'ANY_REGION'` is the maximum opening.
4. **Guard screens OUTPUT only.** Blocked calls still consume tokens.
5. **Embedding model is IMMUTABLE** on a search service — recreate to change.
6. **EMBEDDING truncates at ~512 tokens** silently. Chunk first.
7. **Semantic model YAML lives in a stage**, not a table.
8. **sample_values can leak masked literals** — use non-sensitive examples.
9. **Fine-tune for behavior, RAG for knowledge** — do not confuse.
10. **Pin versions** everywhere: model name, YAML version, Document AI version, Registry version.
11. **ACCOUNT_USAGE latency up to 3h**; INFORMATION_SCHEMA table functions for near-real-time.
12. **COMPLETE return type** flips to OBJECT whenever you pass messages array OR any options.
13. **TRY_COMPLETE** returns NULL on recoverable errors; use in batch pipelines.
14. **VECTOR distance** semantics: cosine higher=similar, L2 lower=similar — opposite sorts.
15. **Native App Cortex credits bill to the CONSUMER** account.
