# Cross-Certification Concept Index

Reverse lookup: find every Q&A file that covers a given concept across all 7 certs. Use this when the same concept shows up with different emphasis in multiple exams and you want to triangulate your understanding.

**Legend** — folder shorthand used below:
- `DEA` = `01_advanced_data_engineer_DEA-C02/`
- `ARA` = `02_advanced_architect_ARA-C01/`
- `GES` = `03_specialty_gen_ai_GES-C01/`
- `GAS` = `04_specialty_ai_governance_security_GAS-C01/`
- `COF` = `05_snowpro_core_COF-C02/`
- `MLE` = `06_advanced_mlops_engineer_MLE-C01/`
- `DAA` = `07_advanced_data_analyst_DAA-C01/`

---

## Architecture & Storage

| Concept | Files |
|---|---|
| **Three-layer architecture** (storage/compute/cloud) | COF `01` · ARA `02` |
| **Micro-partitions** (16 MB, immutable, columnar) | COF `03` · DEA `03` · ARA `02`/`06` · DAA `06` |
| **Partition pruning** | COF `06` · DEA `02` · ARA `04` · DAA `06` |
| **Clustering keys / auto-clustering** | DEA `02` · ARA `04` · COF `06` |
| **Search Optimization Service (SOS)** | DEA `02` · ARA `04` · DAA `06` |
| **Materialized views** | DEA `02` · ARA `04` · COF `06` · DAA `06` |
| **Dynamic Tables** | DEA `01`/`05` · ARA `03` · MLE `04` · DAA `06` |
| **Iceberg Tables** | DEA `01`/`03` · ARA `03` |
| **Hybrid Tables / Unistore** | DEA `03` · ARA `03` |
| **Query Acceleration Service (QAS)** | ARA `02`/`04` · DEA `02` |

## Data Movement & Pipelines

| Concept | Files |
|---|---|
| **COPY INTO** (bulk load) | COF `04` · DEA `01` |
| **Snowpipe (auto-ingest, REST)** | COF `04` · DEA `01` · ARA `03` |
| **Snowpipe Streaming** | DEA `01` · ARA `03` |
| **Streams** (standard, append-only, insert-only) | COF `04` · DEA `01` · ARA `03` |
| **Tasks + Task Graphs** | COF `04` · DEA `01` · ARA `03` |
| **External Tables** | COF `04` · DEA `01` · ARA `03` |
| **Kafka / Spark connectors** | DEA `01` · ARA `03` |
| **Stages** (user/table/named/external) | COF `04` · DEA `01` |
| **File formats + ON_ERROR** | COF `04` · DEA `01` |

## Performance & Cost

| Concept | Files |
|---|---|
| **Warehouse sizing + scaling (vertical/horizontal)** | COF `02` · DEA `02` · ARA `02`/`04` |
| **Multi-cluster warehouses + scaling policy** | COF `02` · ARA `02` |
| **Caching** (result 24h/31d, metadata, SSD, remote) | COF `06` · DEA `02` · ARA `02` |
| **Query Profile reading** | DEA `02` · ARA `04` · DAA `06` |
| **Credit math + resource monitors** | COF `02` · DEA `02` · MLE `06` |
| **Snowpark-optimized warehouses** | DEA `05` · MLE `03` |
| **Snowpark Container Services (SPCS)** | ARA `02` · MLE `03`/`04` · GES `05` |

## Data Protection & DR

| Concept | Files |
|---|---|
| **Time Travel** (1/90 days by edition) | COF `03` · DEA `03` · ARA `06` |
| **Fail-safe** (7 days, fixed) | COF `03` · DEA `03` · ARA `06` |
| **Zero-copy cloning** | COF `03` · DEA `03` · ARA `06` · GAS `06` (legal hold) |
| **Database/account replication** | DEA `03` · ARA `06` |
| **Failover groups** | DEA `03` · ARA `06` |
| **UNDROP** | COF `03` · DEA `03` · ARA `06` |
| **Encryption + Tri-Secret Secure** | ARA `01` · DEA `04` · COF `05` · GAS `04` |

## Security & RBAC

| Concept | Files |
|---|---|
| **RBAC + default roles hierarchy** | COF `05` · DEA `04` · ARA `01` · GAS `03` |
| **Functional + access role pattern** | DEA `04` · ARA `01` · GAS `03` |
| **Future grants + MANAGE GRANTS** | DEA `04` · GAS `03` |
| **Secondary roles trap** (`CURRENT_ROLE` vs `IS_ROLE_IN_SESSION`) | DEA `04` · GAS `03` |
| **Service accounts** (keypair/OAuth/PAT) | DEA `04` · ARA `01` · GAS `03` |
| **SCIM / SSO / SAML / OAuth** | ARA `01` · DEA `04` · COF `05` · GAS `03` |
| **Network policies / PrivateLink** | ARA `01` · DEA `04` · COF `05` · GAS `04` |
| **MFA / session + authentication policies** | COF `05` · ARA `01` · GAS `03` |
| **Trust Center** | GAS `03` |

## Data Governance

| Concept | Files |
|---|---|
| **Dynamic masking policies** | DEA `04` · GAS `04` · COF `05` |
| **Row access policies** | DEA `04` · GAS `04` |
| **Aggregation policies** (min group size) | GAS `04` |
| **Projection policies** | GAS `04` |
| **External tokenization** | GAS `04` |
| **Tags + tag-based policies + inheritance** | DEA `04` · GAS `02` |
| **Auto-classification (`SYSTEM$CLASSIFY`)** | DEA `04` · GAS `02` |
| **Horizon Catalog + lineage** (`GET_LINEAGE`) | GAS `02` |
| **ACCESS_HISTORY + base_objects_accessed** | DEA `04` · GAS `02`/`06` · MLE `06` |
| **Data Metric Functions (DMFs)** | GAS `02` |
| **Policy evaluation order** (RAP→mask→proj→agg) | GAS `04` |

## Compliance & Audit

| Concept | Files |
|---|---|
| **ACCOUNT_USAGE view catalog** | GAS `06` · DEA `04` |
| **GDPR / HIPAA / PCI / SOC2 / FedRAMP mapping** | GAS `04`/`06` |
| **NIST AI RMF / EU AI Act / ISO 42001** | GAS `01` |
| **Shared responsibility model** | GAS `06` |
| **DSAR + right to erasure** | GAS `06` |
| **Legal hold via zero-copy clone** | GAS `06` |

## Data Sharing & Marketplace

| Concept | Files |
|---|---|
| **Secure Data Sharing (zero-copy)** | ARA `05` · DAA `06` |
| **Reader Accounts** | ARA `05` |
| **Snowflake Marketplace + Private Listings** | ARA `05` · DAA `06` |
| **Data Clean Rooms** | ARA `05` · DAA `06` |
| **Native Apps** | ARA `05` · GES `06` · MLE `06` · GAS `06` |
| **Cross-region/cross-cloud share replication** | ARA `05`/`06` |
| **Secure views + masking at share boundary** | ARA `05` |

## SQL & Analytics

| Concept | Files |
|---|---|
| **Window functions + frames (ROWS vs RANGE)** | COF `07` · DAA `02` |
| **QUALIFY** | COF `07` · DAA `02` |
| **GROUPING SETS / ROLLUP / CUBE** | DAA `02` |
| **PIVOT / UNPIVOT** | DAA `02` |
| **MATCH_RECOGNIZE** (sessionization, gaps-and-islands) | DAA `02` |
| **VARIANT / OBJECT / ARRAY** | COF `07` · DAA `03` |
| **LATERAL FLATTEN + params** | COF `07` · DAA `03` |
| **UDFs (SQL / Python / JS / Java / Scala)** | COF `07` · DEA `05` |
| **UDTFs + vectorized UDFs** | DEA `05` · MLE `04` |
| **Stored procedures + Snowflake Scripting** | COF `07` · DEA `05` |
| **Secure views / UDFs** | DEA `04`/`05` · ARA `05` |

## Cortex AI (SQL Functions)

| Concept | Files |
|---|---|
| **COMPLETE / SUMMARIZE / TRANSLATE / SENTIMENT** | GES `01` |
| **EMBED_TEXT + embedding models** | GES `01`/`02` |
| **AI_CLASSIFY / AISQL / TRY_COMPLETE** | GES `01` |
| **Cortex Search service (hybrid dense+sparse)** | GES `02` |
| **Cortex Analyst (text-to-SQL + semantic model)** | GES `03` |
| **Document AI (Arctic-TILT + !PREDICT)** | GES `04` |
| **Cortex Guard** | GES `01`/`06` · GAS `05` |
| **Cortex Fine-Tuning (LoRA)** | GES `01`/`06` |
| **Cortex Agents + function calling** | GES `06` |
| **FORECAST / ANOMALY / CLASSIFICATION / TOP_INSIGHTS** | DAA `05` |

## Snowpark & ML

| Concept | Files |
|---|---|
| **Snowpark DataFrame (lazy eval)** | DEA `05` · DAA `05` |
| **Snowpark ML Modeling API** | MLE `03` |
| **Snowflake Feature Store + PIT joins** | MLE `02` |
| **Snowflake Model Registry + aliases/versions** | MLE `03`/`04` |
| **Model deployment (WH UDF / SPCS / ML Jobs / Cortex)** | MLE `04` |
| **Model Monitor + drift (PSI/KS/chi²)** | MLE `05` |
| **Snowflake Notebooks (WH vs container runtime)** | GES `05` · MLE `01`/`03` |
| **Streamlit in Snowflake** | GES `05`/`06` · MLE `04` · DAA `04` |
| **Git integration + Snow CLI (CI/CD)** | DEA `05` · MLE `06` |

## Visualization & Apps

| Concept | Files |
|---|---|
| **Snowsight dashboards + filters + scheduling** | DAA `04` |
| **Streamlit (`get_active_session`, `st.cache_*`, `st.session_state`)** | DAA `04` · GES `05`/`06` |
| **Native Apps for AI/data** | ARA `05` · GES `06` |

---

## How to use this index
1. See a concept pop up in two certs? Open both files side-by-side — different exams emphasize different angles of the same feature.
2. Reviewing weak areas? Grep this file for the concept, then drill all referenced files.
3. The `_QA.md` files repeat concepts intentionally — that's reinforcement, not redundancy. Each file hits the concept from its own exam's angle.
