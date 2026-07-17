# On-Prem to ADLS Gen2 Migration

**Metadata-driven incremental data pipeline** — Azure Data Factory orchestrates the migration of a 20-table SQL Server star schema to Azure Data Lake Storage Gen2, using a watermark-based incremental loading strategy.

📊 [**View the full project presentation (PDF)**](Migration_Project_EndToEnd.pdf)

---

## Architecture

Three orchestrated pipelines:

| Pipeline | Role |
|---|---|
| `PL_Parent_Orchestrator` | Reads the watermark control table, loops through every source table |
| `PL_Incremental_Load` | Checks for new data, copies changed rows to ADLS as Parquet |
| `PL_Update_Watermark` | Records the load checkpoint after each successful copy |

## Tech Stack

- **Orchestration**: Azure Data Factory (metadata-driven, parameterized pipelines)
- **Source**: On-premise SQL Server, accessed via a Self-hosted Integration Runtime
- **Destination**: Azure Data Lake Storage Gen2 (Parquet, partitioned by table and date)
- **Security**: Azure Key Vault + managed identity — no credentials stored in code
- **Version control**: Git-integrated ADF (this repo is the live source of truth)

## Key Design Decisions

- **Incremental, not full, loads** — a watermark table tracks the last-loaded value per table, so only new or changed rows move on each run.
- **One pipeline, any number of tables** — table-specific logic is driven entirely by parameters and a control table, not hardcoded per table.
- **Secrets never touch source control** — the SQL Server password lives only in Key Vault; ADF's managed identity retrieves it at runtime.

## Results

Successfully migrated 20 tables (~1M+ rows total) from on-premise SQL Server to ADLS Gen2 in a single orchestrated run, validated against source row counts and watermark checkpoints.

---

*Built by [Alan James](https://github.com/alanjamesde)*
