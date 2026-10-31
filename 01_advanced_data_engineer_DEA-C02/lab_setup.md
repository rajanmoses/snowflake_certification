# DEA-C02 Lab Setup (VS Code + Git + Snowflake)

Professional workflow: **source of truth in Git**, edit/run via **VS Code**, use **Snowsight** for Query Profile + visual UI. Repo survives trial expiry and doubles as a portfolio artifact.

---

## Step 1 — Create a free Snowflake trial account

1. Go to **https://signup.snowflake.com**.
2. Fill in name, company (any), email, country. **No credit card required.**
3. Choose:
   - **Edition**: **Enterprise** (needed for ~60% of DEA-C02 labs — 90-day TT, MVs, SOS, Dynamic Tables, Access History). Business Critical only if you want Tri-Secret / cross-cloud replication labs.
   - **Cloud provider**: AWS (most common; `us-east-1` or your nearest region).
4. Check email for the activation link → click → set username/password.
5. First login lands you in **Snowsight** at `https://app.snowflake.com/<org>/<account>`.
6. Capture your account identifier for VS Code later. Run this inside Snowsight:
   - Left sidebar → **Projects → Workspaces** → click **+** (top-right) → **SQL Worksheet** (or **SQL File**). Snowflake renamed "Worksheets" to live under "Workspaces" in recent UI updates.
   - Fastest alternative: press `Cmd+K` (Mac) / `Ctrl+K` (Windows) → type `worksheet` → Enter.
   - Top of the worksheet: set **Role** = `ACCOUNTADMIN`, **Warehouse** = `COMPUTE_WH` (any WH is fine; this query is cloud-services-only and consumes no credits).
   - Paste and run (`Cmd+Return` on Mac / `Ctrl+Enter` on Windows, or click the blue **Run** button):
   ```sql
   SELECT CURRENT_ACCOUNT(), CURRENT_REGION(), CURRENT_ORGANIZATION_NAME();
   ```
   - Example output: `AB12345` | `AWS_US_EAST_1` | `MYORG`.
   - Your **VS Code account identifier** = `<org>-<account>` → e.g. `MYORG-AB12345`. Save this value for Step 4.
7. **Set a budget safety net immediately** (prevents runaway credit burn). Run in the same worksheet:
   ```sql
   USE ROLE ACCOUNTADMIN;
   CREATE OR REPLACE RESOURCE MONITOR trial_budget
     CREDIT_QUOTA = 100
     FREQUENCY = MONTHLY
     TRIGGERS
       ON 50 PERCENT DO NOTIFY
       ON 90 PERCENT DO NOTIFY
       ON 100 PERCENT DO SUSPEND_IMMEDIATE;
   ALTER ACCOUNT SET RESOURCE_MONITOR = trial_budget;
   ```

**Trial facts:** $400 USD credits OR 30 days — whichever first. Convert to on-demand (add card) to keep account past expiry, else data is deleted after a short grace period. Back up DDL + Git repo *before* expiry.

---

## Step 2 — Install VS Code + extensions

1. Install **VS Code**: https://code.visualstudio.com
2. Open Extensions panel (`Cmd+Shift+X`) → install:
   - **Snowflake** (publisher: Snowflake Inc.)
   - **GitLens** (Git supercharger)
   - **Python** (for Snowpark labs)
3. Reload VS Code.

---

## Step 3 — Scaffold the Git repo

```bash
mkdir dea-c02-labs && cd dea-c02-labs
git init
mkdir -p sql/{01_data_movement,02_performance,03_storage,04_security,05_transformation}
mkdir -p python notes screenshots data
```

Create `.gitignore`:
```gitignore
.env
.env.local
*.pem
*.p8
*.key
.venv/
__pycache__/
*.pyc
.vscode/settings.json
~/.snowsql/
.DS_Store
```

Create `.env.example` (template, no secrets):
```bash
SF_ACCOUNT=<org>-<account>
SF_USER=<your_login_name>
SF_PASSWORD=<set_locally_only>
SF_ROLE=SYSADMIN
SF_WAREHOUSE=COMPUTE_WH
SF_DATABASE=LAB_DB
SF_SCHEMA=PUBLIC
```

---

## Step 4 — Connect VS Code to Snowflake

1. Command Palette (`Cmd+Shift+P`) → **"Snowflake: Add Connection"**.
2. Fill in:
   - Account identifier: `<org>-<account>` (from Step 1.6)
   - Username, authenticator (`snowflake` for password or `externalbrowser` for MFA)
   - Default role / warehouse / database / schema
3. Status bar (bottom-right) should show the active connection.
4. Open a `.sql` file → write a statement → `Cmd+Enter` → runs on Snowflake.

---

## Step 5 — Create your lab database

Run once from VS Code:
```sql
USE ROLE SYSADMIN;
CREATE DATABASE IF NOT EXISTS LAB_DB;
CREATE SCHEMA IF NOT EXISTS LAB_DB.PUBLIC;
CREATE WAREHOUSE IF NOT EXISTS LAB_WH
  WAREHOUSE_SIZE = XSMALL
  AUTO_SUSPEND = 60
  AUTO_RESUME = TRUE;
USE WAREHOUSE LAB_WH; USE DATABASE LAB_DB; USE SCHEMA PUBLIC;
```

---

## Step 6 — Daily lab loop

1. Open Q&A file in Obsidian (e.g. `01_data_movement_QA.md` Q5).
2. Create `sql/01_data_movement/q05_external_vs_iceberg.sql` in VS Code.
3. Paste the lab block, run statement-by-statement with `Cmd+Enter`.
4. When a question involves Query Profile: grab `SELECT LAST_QUERY_ID();` → open Snowsight → History → paste ID → screenshot into `screenshots/`.
5. `git add . && git commit -m "lab: data movement Q5 external/iceberg/directory"`.

---

## Cost-saving habits during trial

| Habit | Why |
|-------|-----|
| `WAREHOUSE_SIZE = XSMALL` default | 1 credit/hr baseline |
| `AUTO_SUSPEND = 60` on every WH | No idle burn |
| `CREATE TRANSIENT TABLE` for scratch | No Fail-safe cost |
| `DATA_RETENTION_TIME_IN_DAYS = 0` on throwaway | No TT bytes |
| Daily `SHOW WAREHOUSES` → suspend active ones | Catches forgotten resumes |
| Resource monitor at 100 credits (Step 1.7) | Hard cap |

---

## Backup before trial expires

```sql
-- Export recursive DDL for your lab DB
SELECT GET_DDL('DATABASE', 'LAB_DB', TRUE);

-- Unload any data to stage, download via SnowSQL GET
COPY INTO @~/export/sales/ FROM sales FILE_FORMAT=(TYPE=PARQUET) HEADER=TRUE;
```
Store DDL output and `screenshots/` in the repo. Repo is portable — point it at any future Snowflake account and rerun.
