# NorthPeak Lakehouse

A learning project: migrating a fictional bank, **NorthPeak Bank**, from AWS (S3, RDS Postgres, Kinesis) into a **Databricks Lakehouse** with a bronze → silver → gold medallion architecture, governed by Unity Catalog and deployed from Git.

All data is synthetic. Built on Databricks Free Edition ($0).

## Architecture

```
SIMULATED AWS SOURCES                        DATABRICKS LAKEHOUSE (catalog: northpeak)

S3 bucket (txn CSV)      --Auto Loader-->    bronze.transactions_raw
RDS Postgres (customers) --CDC JSON----->    bronze.customers_cdc
Kinesis (card events)    --JSON files--->    bronze.card_events_raw
                                                    |
                                    Lakeflow declarative pipeline
                                  (clean, cast, dedupe, expectations)
                                                    v
                  silver.transactions, silver.customers (SCD2), silver.card_events
                                                    v
         gold.daily_account_balance, gold.customer_360, gold.fraud_signals
```

Orchestration: Lakeflow Jobs · Deploy: GitHub + Asset Bundles · Governance: Unity Catalog grants, column masks, row filters

## Roadmap

| Phase | What I build | Status |
|---|---|---|
| 0 | Setup: Free Edition workspace, this repo, Git folder | ⬜ |
| 1 | Unity Catalog: catalog, schemas, volume as fake S3 | ⬜ |
| 2 | Simulate AWS sources with synthetic, slightly dirty data | ⬜ |
| 3 | Auto Loader into bronze (incremental, exactly once) | ⬜ |
| 4 | Declarative pipeline: bronze → silver → gold | ⬜ |
| 5 | CDC from the customer database (SCD Type 2) | ⬜ |
| 6 | Delta performance: OPTIMIZE, Z-ORDER, VACUUM | ⬜ |
| 7 | Governance: grants, PII masking, row filters | ⬜ |
| 8 | Orchestration with Lakeflow Jobs + cost controls | ⬜ |
| 9 | CI/CD with Asset Bundles and GitHub Actions | ⬜ |
| 10 | Data engineering with AI | ⬜ |
| 11 | Real AWS integration (optional) | ⬜ |

## Repo layout

```
src/00_setup/                 catalog, schemas, volumes
src/01_generate_source_data/  synthetic data generator
src/02_bronze_autoloader/     Auto Loader ingestion
src/03_pipeline/              declarative pipeline code
src/04_maintenance/           optimize, vacuum
src/05_governance/            grants, masks, row filters
src/06_ai/                    AI layer
tests/                        unit tests
docs/                         notes and diagrams
LEARNINGS.md                  what broke, how I fixed it, what I'd say in an interview
```

## Schema conversion and type mapping

| Source column | Source type (as received) | Target Delta type | Rule |
|---|---|---|---|
| txn_ts | string, 2 formats | TIMESTAMP | try `yyyy-MM-dd HH:mm:ss`, then `dd/MM/yyyy HH:mm` |
| amount | string, may be blank | DECIMAL(18,2) | never use FLOAT for money |
| currency | string, mixed case | STRING | `upper()` |
| customer_id | int (Postgres `integer`) | BIGINT | |
| ts_ms | epoch milliseconds | TIMESTAMP | `timestamp_millis()` |
| event_ts | ISO-8601 string | TIMESTAMP | |