# Snowflake Lab Environment Setup

One-time bootstrap for a clean trial account so every lab in the Q&A files runs without "missing warehouse/role/stage" errors. Run this once as `ACCOUNTADMIN`, then switch to the lab roles.

## Prerequisites
- Snowflake trial account (any edition — Enterprise+ recommended for extended Time Travel, replication, SOS, MVs)
- Snowsight access + optional SnowSQL / Snow CLI installed
- (Optional) An external S3 bucket you own, for stage/pipe labs

---

## 1. Role & User Setup

```sql
USE ROLE ACCOUNTADMIN;

-- Lab roles (two-tier: functional + access roles)
CREATE ROLE IF NOT EXISTS LAB_ADMIN;
CREATE ROLE IF NOT EXISTS LAB_ENGINEER;
CREATE ROLE IF NOT EXISTS LAB_ANALYST;
CREATE ROLE IF NOT EXISTS LAB_ML_ENGINEER;
CREATE ROLE IF NOT EXISTS LAB_AI_DEVELOPER;
CREATE ROLE IF NOT EXISTS LAB_GOVERNANCE;

GRANT ROLE LAB_ADMIN       TO ROLE SYSADMIN;
GRANT ROLE LAB_ENGINEER    TO ROLE LAB_ADMIN;
GRANT ROLE LAB_ANALYST     TO ROLE LAB_ADMIN;
GRANT ROLE LAB_ML_ENGINEER TO ROLE LAB_ADMIN;
GRANT ROLE LAB_AI_DEVELOPER TO ROLE LAB_ADMIN;
GRANT ROLE LAB_GOVERNANCE  TO ROLE SECURITYADMIN;

-- Grant lab roles to your user (replace YOUR_USER)
GRANT ROLE LAB_ADMIN TO USER YOUR_USER;
```

## 2. Warehouses

```sql
USE ROLE SYSADMIN;

-- Right-size per workload. Default to XS to keep costs minimal.
CREATE WAREHOUSE IF NOT EXISTS LAB_WH
  WAREHOUSE_SIZE = XSMALL
  AUTO_SUSPEND   = 60
  AUTO_RESUME    = TRUE
  INITIALLY_SUSPENDED = TRUE
  COMMENT = 'Default lab compute';

-- Larger warehouse for load/perf labs (keep suspended)
CREATE WAREHOUSE IF NOT EXISTS LAB_LOAD_WH
  WAREHOUSE_SIZE = MEDIUM
  AUTO_SUSPEND   = 60
  AUTO_RESUME    = TRUE
  INITIALLY_SUSPENDED = TRUE;

-- Snowpark-optimized for ML labs (memory-heavy)
CREATE WAREHOUSE IF NOT EXISTS LAB_ML_WH
  WAREHOUSE_SIZE = MEDIUM
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED'
  AUTO_SUSPEND   = 60
  AUTO_RESUME    = TRUE
  INITIALLY_SUSPENDED = TRUE;

GRANT USAGE, OPERATE ON WAREHOUSE LAB_WH      TO ROLE LAB_ADMIN;
GRANT USAGE, OPERATE ON WAREHOUSE LAB_LOAD_WH TO ROLE LAB_ENGINEER;
GRANT USAGE, OPERATE ON WAREHOUSE LAB_ML_WH   TO ROLE LAB_ML_ENGINEER;
```

## 3. Database & Schema Layout

```sql
USE ROLE SYSADMIN;

CREATE DATABASE IF NOT EXISTS LAB_DB
  DATA_RETENTION_TIME_IN_DAYS = 7;   -- Enterprise+ allows up to 90

-- Schemas by domain
CREATE SCHEMA IF NOT EXISTS LAB_DB.RAW;        -- landing / stages / Snowpipe targets
CREATE SCHEMA IF NOT EXISTS LAB_DB.STAGING;    -- transformations / streams
CREATE SCHEMA IF NOT EXISTS LAB_DB.ANALYTICS;  -- marts / dynamic tables / MVs
CREATE SCHEMA IF NOT EXISTS LAB_DB.ML;         -- feature store, model registry
CREATE SCHEMA IF NOT EXISTS LAB_DB.GEN_AI;     -- Cortex search services, semantic models
CREATE SCHEMA IF NOT EXISTS LAB_DB.GOVERNANCE; -- tags, policies

GRANT USAGE ON DATABASE LAB_DB TO ROLE LAB_ADMIN;
GRANT USAGE ON ALL SCHEMAS IN DATABASE LAB_DB TO ROLE LAB_ADMIN;
GRANT ALL ON ALL SCHEMAS IN DATABASE LAB_DB TO ROLE LAB_ADMIN;
```

## 4. Sample Data Access

Snowflake ships two sample databases you can use directly — no load required:

```sql
-- SNOWFLAKE_SAMPLE_DATA is auto-mounted on every account via a share
USE DATABASE SNOWFLAKE_SAMPLE_DATA;
SHOW SCHEMAS;
-- TPCH_SF1 (small), TPCH_SF10, TPCH_SF100, TPCDS_SF10TCL
SELECT COUNT(*) FROM TPCH_SF1.LINEITEM;   -- 6M rows, good for perf labs
SELECT COUNT(*) FROM TPCH_SF10.LINEITEM;  -- 60M rows
```

For ML/Gen-AI labs, you'll usually create your own small datasets inline (see individual _QA.md files).

## 5. External Stage (optional, for Snowpipe labs)

Only needed if you have an S3/Azure/GCS bucket:

```sql
USE ROLE ACCOUNTADMIN;

CREATE STORAGE INTEGRATION IF NOT EXISTS LAB_S3_INT
  TYPE = EXTERNAL_STAGE
  STORAGE_PROVIDER = 'S3'
  ENABLED = TRUE
  STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::<your-account>:role/snowflake-lab-role'
  STORAGE_ALLOWED_LOCATIONS = ('s3://your-lab-bucket/');

DESC INTEGRATION LAB_S3_INT;
-- Copy STORAGE_AWS_IAM_USER_ARN and STORAGE_AWS_EXTERNAL_ID, update IAM trust policy

USE ROLE SYSADMIN;
GRANT USAGE ON INTEGRATION LAB_S3_INT TO ROLE LAB_ENGINEER;

USE SCHEMA LAB_DB.RAW;
CREATE STAGE IF NOT EXISTS lab_s3_stage
  URL = 's3://your-lab-bucket/'
  STORAGE_INTEGRATION = LAB_S3_INT
  FILE_FORMAT = (TYPE = CSV);
```

Skip this section entirely if you're staying with internal stages — `CREATE STAGE lab_internal_stage;` works out of the box.

## 6. Governance Objects

```sql
USE ROLE ACCOUNTADMIN;
USE SCHEMA LAB_DB.GOVERNANCE;

-- Shared tags for lab exercises
CREATE TAG IF NOT EXISTS pii_level
  ALLOWED_VALUES 'none','low','medium','high','restricted';
CREATE TAG IF NOT EXISTS cost_center
  ALLOWED_VALUES 'engineering','analytics','ml','gen_ai';
CREATE TAG IF NOT EXISTS environment
  ALLOWED_VALUES 'dev','test','prod';

GRANT APPLY ON TAG pii_level     TO ROLE LAB_GOVERNANCE;
GRANT APPLY ON TAG cost_center   TO ROLE LAB_GOVERNANCE;
GRANT APPLY ON TAG environment   TO ROLE LAB_ADMIN;
```

## 7. Cortex / Gen-AI Prerequisites

```sql
USE ROLE ACCOUNTADMIN;

-- Cortex functions require CORTEX_USER (default: granted to PUBLIC in most regions)
GRANT DATABASE ROLE SNOWFLAKE.CORTEX_USER TO ROLE LAB_AI_DEVELOPER;

-- Quick smoke-test
USE ROLE LAB_AI_DEVELOPER;
USE WAREHOUSE LAB_WH;
SELECT SNOWFLAKE.CORTEX.COMPLETE('mistral-7b','Say hi in 5 words.');
```

If this errors with "region not supported", switch to a Cortex-enabled region (AWS us-west-2, us-east-1; Azure East US 2; etc.).

## 8. Resource Monitor (cost safety)

```sql
USE ROLE ACCOUNTADMIN;

CREATE RESOURCE MONITOR IF NOT EXISTS LAB_RM
  WITH CREDIT_QUOTA = 50
  FREQUENCY = MONTHLY
  START_TIMESTAMP = IMMEDIATELY
  TRIGGERS
    ON 75 PERCENT DO NOTIFY
    ON 90 PERCENT DO SUSPEND
    ON 100 PERCENT DO SUSPEND_IMMEDIATE;

ALTER WAREHOUSE LAB_WH      SET RESOURCE_MONITOR = LAB_RM;
ALTER WAREHOUSE LAB_LOAD_WH SET RESOURCE_MONITOR = LAB_RM;
ALTER WAREHOUSE LAB_ML_WH   SET RESOURCE_MONITOR = LAB_RM;
```

Protects your trial credits — monitor suspends warehouses at 90% of the 50-credit monthly quota.

---

## Daily Lab Session Flow

```sql
-- Pick the role that matches the module you're studying
USE ROLE LAB_ENGINEER;          -- DEA-C02, ARA-C01 modules
-- USE ROLE LAB_ANALYST;        -- DAA-C01 modules
-- USE ROLE LAB_ML_ENGINEER;    -- MLE-C01 modules
-- USE ROLE LAB_AI_DEVELOPER;   -- GES-C01 modules
-- USE ROLE LAB_GOVERNANCE;     -- GAS-C01 modules

USE WAREHOUSE LAB_WH;
USE DATABASE LAB_DB;
USE SCHEMA RAW;   -- or STAGING / ANALYTICS / ML / GEN_AI
```

## Teardown (end of study session)

```sql
USE ROLE ACCOUNTADMIN;
ALTER WAREHOUSE LAB_WH      SUSPEND;
ALTER WAREHOUSE LAB_LOAD_WH SUSPEND;
ALTER WAREHOUSE LAB_ML_WH   SUSPEND;
```

Credit-friendly default: everything auto-suspends after 60 seconds anyway, but explicit `SUSPEND` closes the loop immediately.

## Full Reset (after you're done with the certs)

```sql
USE ROLE ACCOUNTADMIN;
DROP DATABASE IF EXISTS LAB_DB;
DROP WAREHOUSE IF EXISTS LAB_WH;
DROP WAREHOUSE IF EXISTS LAB_LOAD_WH;
DROP WAREHOUSE IF EXISTS LAB_ML_WH;
DROP RESOURCE MONITOR IF EXISTS LAB_RM;
DROP INTEGRATION IF EXISTS LAB_S3_INT;
DROP ROLE IF EXISTS LAB_ADMIN;
DROP ROLE IF EXISTS LAB_ENGINEER;
DROP ROLE IF EXISTS LAB_ANALYST;
DROP ROLE IF EXISTS LAB_ML_ENGINEER;
DROP ROLE IF EXISTS LAB_AI_DEVELOPER;
DROP ROLE IF EXISTS LAB_GOVERNANCE;
```
