# ci-cd-azure-pipeline

> **Enterprise-grade CI/CD pipeline for automated multi-supplier data ingestion and transformation on Azure.**

[![Azure DevOps](https://img.shields.io/badge/Azure%20DevOps-CI%2FCD-0078D7?style=flat&logo=azuredevops&logoColor=white)](https://dev.azure.com)
[![Azure Data Factory](https://img.shields.io/badge/ADF-Orchestration-0072C6?style=flat&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/en-us/products/data-factory)
[![Delta Lake](https://img.shields.io/badge/Delta%20Lake-Storage-003366?style=flat&logo=apachespark&logoColor=white)](https://delta.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## Demo

> **Loom Walkthrough:** *(link added after Week 4 deployment)*
> **Live Dashboard:** *(URL added after deployment)*

---

## Problem Statement

Large-scale enterprise environments frequently receive daily data payloads from hundreds of external suppliers, often in heterogeneous formats (CSV, JSON, XLSX) with inconsistent schemas and encodings. This fragmentation typically requires significant manual intervention for data cleaning and validation, leading to high operational overhead and poor auditability.

This project implements a fully automated, end-to-end Azure data pipeline to address this challenge. It ingests, validates, standardizes, and processes five heterogeneous supplier feeds into a Bronze → Silver → Gold Medallion Lakehouse architecture. The solution features automated bad-row quarantine, comprehensive schema evolution tracking, and is deployed entirely via Azure DevOps CI/CD using ARM templates.

A core feature is the **schema-drift detector**: it automatically identifies when a supplier modifies upstream column definitions, logs the schema evolution, and enables the pipeline to fail gracefully or self-heal without manual engineering intervention.

---

## Architecture

![Azure CI/CD Architecture](architecture.svg)

### CI/CD Pipeline Breakdown

| Stage | What Happens |
|-------|-------------|
| **Feature Branch** | Developer builds ADF pipeline in Live Mode |
| **Publish** | ADF auto-generates ARM templates to `adf_publish` branch |
| **CI Pipeline (YAML)** | Validates ARM templates → packages as build artifact |
| **CD → DEV** | Deploys ARM templates to ADF DEV environment automatically |
| **Approval Gate → QA** | Manual approval required before QA deployment |
| **Pre/Post Scripts** | Run environment-specific scripts (linked service swap, trigger toggle) |
| **Approval Gate → PROD** | Second manual approval before PROD deployment |

---

## Tech Stack

| Layer | Tool | Purpose |
|-------|------|---------|
| **Orchestration** | Azure Data Factory (ADF) | Ingestion + transformation pipelines |
| **Storage** | Azure Data Lake Storage Gen2 | Bronze / Silver / Gold medallion layers |
| **Secrets** | Azure Key Vault | Connection strings, storage keys |
| **Identity** | Managed Identity | Passwordless auth between Azure services |
| **CI/CD** | Azure DevOps (YAML Pipelines) | Automated build + multi-env deployment |
| **IaC** | ARM Templates | Infrastructure-as-code for ADF |
| **Language** | Python (ADF Data Flows / scripts) | Pre/Post deployment scripts |
| **Dashboard** | Databricks SQL / Azure Synapse | Observability + data quality metrics |

---

## Quickstart

### Prerequisites

- Azure Subscription (Azure for Students — $100 free credits)
- Azure DevOps organisation (free tier)
- Azure CLI installed locally
- Python 3.11+
- Git

### Step 1 — Clone the repo

```bash
git clone https://github.com/<your-username>/ci-cd-azure-pipeline.git
cd ci-cd-azure-pipeline
```

### Step 2 — Set up Azure resources

```bash
# Login to Azure
az login

# Run the setup script (creates Resource Group, ADLS, Key Vault, ADF)
python scripts/setup_azure_resources.py --env dev
```

### Step 3 — Configure Azure DevOps

1. Create a new Azure DevOps project
2. Connect your GitHub repo to Azure DevOps Pipelines
3. Import `pipelines/ci-pipeline.yml` → CI pipeline
4. Import `pipelines/cd-pipeline.yml` → CD pipeline
5. Add service connection: **Azure Resource Manager** → your subscription

### Step 4 — Configure Key Vault secrets

```bash
# Add your ADLS connection string to Key Vault
az keyvault secret set \
  --vault-name <your-kv-name> \
  --name "adls-connection-string" \
  --value "<your-adls-connection-string>"
```

### Step 5 — Drop test supplier files

```bash
# Copy the 5 simulated supplier files to ADLS raw zone
python scripts/upload_supplier_files.py --env dev
```

### Step 6 — Trigger the pipeline

```bash
# Trigger ADF pipeline manually (or wait for the daily 6 AM schedule)
az datafactory pipeline create-run \
  --factory-name <your-adf-name> \
  --resource-group <your-rg-name> \
  --name "pl_ingest_all_suppliers"
```

### Step 7 — Run tests

```bash
pip install -r requirements.txt
pytest tests/ -v
```

---

## Data Sources

| Supplier | Format | Deliberate Quirk | Maps To |
|----------|--------|-----------------|---------|
| `supplier_walmart_us` | CSV UTF-8 | Standard baseline | Walmart US supply chain |
| `supplier_samsung_kr` | XLSX | Merged header cells | Samsung Korea |
| `supplier_unilever_nl` | CSV ISO-8859-1, `;` delimiter | Encoding + delimiter mismatch | Unilever Netherlands |
| `supplier_tata_in` | JSON | Nested `items[]` array | Tata India |
| `supplier_ikea_se` | CSV | Swedish column names (`produkt_id`) | IKEA Sweden |

All files are **synthetic data** modelled on real supplier data patterns. No real data is used.

---

## Mini-Extension: Schema-Drift Detector

One of the biggest causes of production data pipeline failures at MNCs is **schema drift** — when a supplier silently changes their column names or adds new fields, breaking downstream pipelines.

**What it does:**
1. Every ADF pipeline run compares the incoming file schema against the last known schema stored in `audit.schema_history`
2. If a column is **added**, **removed**, or **renamed** → logs a `schema_change` event
3. Pipeline does **NOT fail** — it self-heals and continues (fail-soft architecture)
4. Dashboard shows schema change history per supplier over time

**Why it matters:**
> *"Schema drift was the #1 cause of pipeline downtime at Walmart and Amazon's supply chain teams. I built a detector that handles it without human intervention."*

**Tables created:**
```sql
audit.schema_history (
    supplier_id     TEXT,
    detected_at     TIMESTAMP,
    event_type      TEXT,   -- 'column_added' | 'column_removed' | 'type_changed'
    column_name     TEXT,
    old_value       TEXT,
    new_value       TEXT
)

audit.quarantine_log (
    supplier_id     TEXT,
    file_name       TEXT,
    row_number      INT,
    raw_data        TEXT,
    failure_reason  TEXT,
    ingested_at     TIMESTAMP
)
```

---

## Project Structure

```
ci-cd-azure-pipeline/
│
├── adf/                          # ADF pipeline JSON definitions
│   ├── pipelines/
│   │   ├── pl_ingest_all_suppliers.json
│   │   ├── pl_validate_and_quarantine.json
│   │   └── pl_transform_gold.json
│   ├── datasets/
│   ├── linkedServices/
│   └── triggers/
│
├── pipelines/                    # Azure DevOps YAML pipelines
│   ├── ci-pipeline.yml           # CI: validate + package ARM templates
│   └── cd-pipeline.yml           # CD: deploy DEV → QA → PROD
│
├── arm-templates/                # Auto-generated + custom ARM templates
│   ├── ARMTemplateForFactory.json
│   └── ARMTemplateParametersForFactory.json
│
├── scripts/                      # Pre/Post deployment scripts
│   ├── setup_azure_resources.py
│   ├── upload_supplier_files.py
│   ├── pre_deployment.py         # Stop triggers before deploy
│   └── post_deployment.py        # Restart triggers after deploy
│
├── data/
│   └── sample/                   # 5 simulated supplier files
│       ├── supplier_walmart_us.csv
│       ├── supplier_samsung_kr.xlsx
│       ├── supplier_unilever_nl.csv
│       ├── supplier_tata_in.json
│       └── supplier_ikea_se.csv
│
├── tests/
│   ├── test_schema_drift.py
│   ├── test_quarantine_logic.py
│   └── test_arm_template_valid.py
│
├── docs/
│   ├── adr/
│   │   ├── ADR-001-adf-vs-databricks-workflows.md
│   │   ├── ADR-002-arm-templates-vs-bicep.md
│   │   └── ADR-003-medallion-vs-flat-storage.md
│   ├── design_doc.md
│   ├── reflection.md
│   ├── roadmap_3rd_year.md
│   └── resume_bullets.md
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Architecture Decision Records (ADRs)

| ADR | Decision | Why |
|-----|----------|-----|
| [ADR-001](docs/adr/ADR-001-adf-vs-databricks-workflows.md) | ADF over Databricks Workflows | Simpler for batch ingestion at 2nd year level; ADF is free within Azure credits |
| [ADR-002](docs/adr/ADR-002-arm-templates-vs-bicep.md) | ARM Templates over Bicep | ADF natively generates ARM — no additional tooling needed |
| [ADR-003](docs/adr/ADR-003-medallion-vs-flat-storage.md) | Bronze/Silver/Gold over flat storage | Industry standard; enables replay without re-ingestion |

---

## Known Limitations

- **Azure credits:** Running ADF pipelines on a schedule will consume Azure for Students credits. Set auto-pause on all compute resources.
- **5 suppliers only:** The pipeline is scoped to 5 simulated suppliers — not production scale.
- **No real-time:** This is a batch pipeline (daily schedule). Streaming is out of scope for 2nd year.
- **Dashboard is read-only:** The observability dashboard shows metrics but does not support write-back or alerting.
- **ARM template conflicts:** If ADF is modified in Live Mode without publishing, CI/CD will deploy stale templates. Always publish before triggering the pipeline.

---



## License

MIT License — see [LICENSE](LICENSE)

---

## Acknowledgements

- Architecture inspired by the [Azure CI/CD for Data Engineers](https://www.youtube.com/watch?v=...) course
- ARM template patterns from [Azure Data Factory documentation](https://learn.microsoft.com/en-us/azure/data-factory/)
- Medallion architecture pattern from [Databricks documentation](https://www.databricks.com/glossary/medallion-architecture)
- Problem scenario modelled on real supplier data challenges at Walmart, Amazon, and Unilever (publicly documented)

---

*Built during the 2nd Year Internship (June–July 2026) | Segment 2 — Foundations of Data Engineering | Problem H2 — CSV Hell Pipeline*
