# Azure Data Factory CI/CD Pipeline
### Enterprise-Grade DevOps Implementation for Data Engineering Workflows

![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Azure DevOps](https://img.shields.io/badge/Azure%20DevOps-0078D7?style=for-the-badge&logo=azuredevops&logoColor=white)
![ADF](https://img.shields.io/badge/Azure%20Data%20Factory-0072C6?style=for-the-badge&logo=microsoftazure&logoColor=white)
![YAML](https://img.shields.io/badge/YAML-CB171E?style=for-the-badge&logo=yaml&logoColor=white)
![Status](https://img.shields.io/badge/status-active-success?style=for-the-badge)
![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)

**Author:** Aamod Kumar · [GitHub](https://github.com/Aamod007) · [LinkedIn](https://linkedin.com/in/aamod-kumar)
**Duration:** 5 Weeks (self-directed, structured learning sprint)
**Domain:** Cloud Engineering · Data Engineering · DevOps

---

## 📑 Table of Contents

1. [Project Overview](#project-overview)
2. [Problem Statement](#problem-statement)
3. [Objectives](#objectives)
4. [Complete Architecture](#complete-architecture)
5. [Azure Services Used](#azure-services-used)
6. [CI/CD Workflow](#cicd-workflow)
7. [Git Branching Strategy](#git-branching-strategy)
8. [Azure DevOps Pipeline](#azure-devops-pipeline)
9. [ARM Template Deployment](#arm-template-deployment)
10. [Environment Strategy](#environment-strategy)
11. [Key Vault Integration](#key-vault-integration)
12. [Managed Identity Authentication](#managed-identity-authentication)
13. [Storage Account Configuration](#storage-account-configuration)
14. [Azure Data Factory Pipelines](#azure-data-factory-pipelines)
15. [Dynamic Parameterization](#dynamic-parameterization)
16. [Deployment Process](#deployment-process)
17. [Security Implementation](#security-implementation)
18. [Folder Structure](#folder-structure)
19. [Technologies](#technologies)
20. [Prerequisites](#prerequisites)
21. [Installation Guide](#installation-guide)
22. [Configuration Guide](#configuration-guide)
23. [Screenshots](#screenshots)
24. [Testing](#testing)
25. [Challenges & Resolutions](#challenges--resolutions)
26. [Future Enhancements](#future-enhancements)
27. [Learning Outcomes](#learning-outcomes)
28. [Weekly Logs](#weekly-logs)

---

## Project Overview

This repository documents the design and implementation of a **production-style CI/CD pipeline for Azure Data Factory (ADF)**, built end-to-end across three isolated environments — **Dev → QA → Production** — using **Azure DevOps**, **Git**, **ARM templates**, and **YAML pipelines**.

The project simulates how a data engineering team would actually ship pipeline changes in a regulated, multi-developer environment: isolated feature branches, pull-request-gated merges into a protected `main` branch, automated ARM template generation, environment-scoped parameter overrides, Key Vault-backed secrets, managed-identity authentication (no hardcoded credentials anywhere), and a manual approval gate before anything reaches production.

Rather than treating ADF as a UI-only drag-and-drop tool, this project treats it as **infrastructure-as-code**: every linked service, dataset, pipeline, and trigger is version-controlled, exported as an ARM template, and promoted through environments via automated pipelines — the same pattern used by data engineering teams in real organizations.

| Field | Detail |
|---|---|
| Project Type | Self-directed Cloud/Data Engineering training project |
| Institution | `Futurence` |
| Mentor / Guide | `Aalekh Rai` |
| Duration | 5 Weeks |
| Cloud Provider | Microsoft Azure |
| Primary Service | Azure Data Factory (ADF) |
| CI/CD Platform | Azure DevOps (YAML Pipelines) |
| Version Control | Git (Azure Repos) |

---

## Problem Statement

In any real-world data engineering team, more than one developer works against the same Azure Data Factory instance. Without a disciplined deployment process, this creates three concrete problems:

1. **Concurrent-edit conflicts** — multiple developers modifying the same live ADF instance simultaneously corrupt each other's in-progress pipelines, since ADF's default "Live Mode" has no built-in isolation between developers.
2. **Unsafe promotion to production** — without a controlled pipeline, there is no reliable, repeatable way to move tested pipeline logic from a development factory to QA and then to production without manual re-creation (which is error-prone and not auditable).
3. **Hardcoded secrets and environment leakage** — without parameterization, a pipeline built in Dev will keep pointing at Dev's storage account and Key Vault even after being deployed to QA or Production, silently corrupting environment boundaries.

This project solves all three by treating ADF as versioned infrastructure and using industry-standard CI/CD tooling to move code safely between environments.

---

## Objectives

- [x] Set up isolated **Dev, QA, and Production** environments in Azure, each with its own Data Factory, Storage Account (Azure Blob Storage), and Key Vault.
- [x] Connect ADF to **Git (Azure Repos)** and implement a **feature-branch workflow** protected by branch policies and pull requests.
- [x] Build **dynamic, parameter-driven ADF pipelines** (HTTP → Azure Blob Storage) instead of static, hardcoded pipelines.
- [x] Authenticate all inter-service communication using **System-Assigned Managed Identity** — zero hardcoded keys or secrets.
- [x] Automate **ARM template generation** (Continuous Integration) using the official `@microsoft/azure-data-factory-utilities` npm package instead of manual "Publish" clicks.
- [x] Build a **YAML-based Continuous Deployment pipeline** that promotes the ARM template through Dev → QA → Production with environment-specific parameter files.
- [x] Implement **pre/post-deployment PowerShell scripts** to safely stop and restart ADF triggers during deployment (handles object deletion, which ARM alone cannot do safely).
- [x] Add a **manual approval gate** before any deployment reaches Production, using Azure DevOps Environments.
- [x] Document the entire pipeline as reusable, professional, GitHub-grade documentation.

---

## Complete Architecture

### High-Level Flow

![Azure CI/CD Architecture](architecture.svg)

### ADF "Live Mode" vs. Git Mode

- **Live Mode** — the "source of truth" state of an ADF instance, editable only via publish. Never edited directly.
- **Collaboration Branch (`main`)** — mirrors Live Mode; developers never commit here directly.
- **Feature Branches** — isolated developer sandboxes branched off `main`; all pipeline/dataset/linked-service development happens here.
- **`adf_publish` branch** — auto-managed by Azure; stores the exported ARM templates and pre/post deployment scripts once "Publish" (manual) or the CI pipeline (automated) runs.

---

## Azure Services Used

| Service | Purpose |
|---|---|
| **Azure Data Factory** | Core ETL/orchestration engine; source of the ARM templates being deployed |
| **Azure Data Lake Storage Gen 2** | Sink for pipeline output data; hierarchical namespace enabled |
| **Azure Key Vault** | Centralized secrets management (RBAC-based access policy) |
| **Azure DevOps – Repos** | Git-based source control, branch policies, pull requests |
| **Azure DevOps – Pipelines** | YAML-based CI (build/export) and CD (deploy) automation |
| **Azure DevOps – Library** | Environment-scoped Variable Groups (Dev/QA/Prod) |
| **Azure DevOps – Environments** | Approval gates for production deployment |
| **Azure Active Directory (Entra ID)** | System-Assigned Managed Identities + Service Principal (Service Connection) |
| **Azure Resource Manager (ARM)** | Infrastructure-as-code deployment format for the entire ADF instance |

---

## CI/CD Workflow

### Continuous Integration (CI)
1. Developer pushes to a feature branch → opens a Pull Request into `main`.
2. PR requires **minimum 1 reviewer approval** (branch policy) before merge.
3. On merge, the CI pipeline (`ci-build.yaml`) triggers:
   - Installs Node.js on the build agent
   - Installs the `@microsoft/azure-data-factory-utilities` npm package
   - Runs `npm run build validate` against the ADF root folder
   - Runs `npm run build export` to generate the ARM template + parameters file
   - Publishes the ARM template folder (including the Microsoft-provided pre/post deployment PowerShell script) as a pipeline artifact

### Continuous Deployment (CD)
1. `cd-deploy.yaml` (a reusable template) downloads the CI artifact.
2. Runs the **pre-deployment script** (stops active ADF triggers so in-flight objects aren't corrupted mid-deploy).
3. Deploys the ARM template using the `AzureResourceManagerTemplateDeployment@3` task in **Incremental mode**, overriding parameters with an environment-specific `arm-params/{env}.json` file.
4. Runs the **post-deployment script** (deletes orphaned objects removed from source, restarts previously running triggers).
5. Repeats automatically for **Dev** and **QA**.
6. **Production** deployment runs as a `deployment` job tied to a DevOps **Environment** with an approval check — a human must explicitly approve before the job proceeds.

---

## Git Branching Strategy

| Branch | Purpose | Who can write |
|---|---|---|
| `main` | Collaboration branch; mirrors ADF Live Mode | No direct commits — PR only, branch-policy protected |
| `feature/*` | Isolated per-developer / per-change branches | Any contributor |
| `adf_publish` | Auto-generated ARM template + scripts store | Managed by Azure Data Factory / pipeline only |

**Policy applied on `main`:**
- Minimum 1 reviewer required
- Requester can approve their own PR (small-team setting; disable for larger teams)
- Fast-forward merge strategy used to keep history linear

This mirrors the isolation model used by real engineering teams — no two developers can silently overwrite each other's in-progress pipeline logic, because each works in a fully isolated ADF "branch factory" view until merge.

---

## Azure DevOps Pipeline

### Pipeline Hierarchy (YAML)
```
Pipeline
 └── Stage        (one per logical goal — Build, Deploy-Dev, Deploy-QA, Deploy-Prod)
      └── Job       (unit of work, executed by an Agent)
           └── Step/Task   (individual command — install Node, run validate, deploy ARM, etc.)
```

### Orchestrator Pattern
Instead of one giant YAML file, the pipeline is split for maintainability:

| File | Role |
|---|---|
| `ci-cd-pipeline.yaml` | Parent/orchestrator — defines trigger, agent pool, stages, and imports variable groups |
| `ci-build.yaml` | Reusable CI template — installs deps, validates, exports ARM template |
| `cd-deploy.yaml` | Reusable CD template — downloads artifact, runs pre/post scripts, deploys ARM |

Stages call the reusable templates with parameters (`dataFactoryName`, `resourceGroup`, `subscriptionId`, `environment`, `azureResourceManagerConnection`), so the same deployment logic runs identically against Dev, QA, and Prod — only the parameter values change.

### Agent Pool
- Microsoft-hosted **Ubuntu-latest** agent (1 free parallel job, 1,800 build minutes/month — requested via Azure DevOps parallelism request form; approval typically takes 24–48 hours).

---

## ARM Template Deployment

Every object inside ADF — linked services, datasets, pipelines, triggers — is backed by an ARM (Azure Resource Manager) template under the hood. Two ways to generate it were implemented and tested:

1. **Manual Publish** (traditional) — clicking "Publish" inside ADF Studio exports the ARM template to the `adf_publish` branch. Still used in ~50% of real-world setups; useful for understanding the mechanics.
2. **Automated Export** (modern/CI) — the `@microsoft/azure-data-factory-utilities` npm package runs `npm run build export` inside the pipeline agent, producing the same ARM template without any manual click. This is the approach the CI pipeline in this repo uses.

**Deployment task:** `AzureResourceManagerTemplateDeployment@3`
**Deployment mode:** `Incremental` (never `Complete` — complete mode deletes any resource not present in the template, which is destructive and unnecessary here)

---

## Environment Strategy

| Environment | Resource Group | Data Factory | Storage Account | Key Vault | Deployment Trigger | Approval Required |
|---|---|---|---|---|---|---|
| Dev | `ADF-CICD-Dev` | `adf-<name>-dev` | `storage-dev` | `key-dev` | Automatic on merge to `main` | No |
| QA | `ADF-CICD-QA` | `adf-<name>-qa` | `storage-qa` | `key-qa` | Automatic, after Dev | No |
| Production | `ADF-CICD-Prod` | `adf-<name>-prod` | `storage-prod` | `key-prod` | Automatic, after QA | **Yes — manual gate** |

Each environment is provisioned identically (same resource types, same RBAC pattern), with only names and parameter values differing — enforced entirely through the `arm-params/{environment}.json` override files, never through code changes.

---

## Key Vault Integration

- Key Vault is used to centralize secrets (connection strings, API keys) so nothing is hardcoded inside pipeline JSON.
- **Access model:** Vault Access Policy (not pure Azure RBAC) is used for the vault itself, because ADF's Web Activity / linked service integration calls the Key Vault API directly and needs policy-based access to retrieve secrets at runtime.
- The deploying identity (developer account initially, then the Service Principal / Managed Identity) is granted the **Key Vault Administrator** role via IAM to manage secrets, separate from the **Vault Access Policy** that grants the Data Factory's managed identity **Get/List** permissions on secrets.
- Every environment (Dev/QA/Prod) has its own Key Vault — secrets never cross environment boundaries.

---

## Managed Identity Authentication

**System-Assigned Managed Identity** is used everywhere instead of access keys or connection-string secrets:

- Every Azure Data Factory instance automatically gets a system-assigned identity (visible under **Settings → Identity** in the portal).
- This identity is granted the **Storage Blob Data Contributor** role (via Azure IAM → Add Role Assignment → select "Managed Identity" → filter by "Data Factory") on the Azure Blob Storage storage account.
- The same identity pattern is used for Key Vault access.
- **Why it matters:** no access keys are ever stored in linked service JSON, pipeline parameters, or Git — eliminating an entire class of credential-leak risk.

---

## Storage Account Configuration

- **Type:** Azure Data Lake Storage Gen 2 (Blob storage with **Hierarchical Namespace enabled** — this checkbox is mandatory; without it, the account behaves as flat blob storage, not a data lake).
- **Redundancy:** LRS (Locally Redundant Storage) — sufficient for a non-production-critical training/demo workload.
- **Containers created:**
  - `raw` — landing zone for ingested CSV data (bookings, passengers)
  - `parameters` — holds the runtime `params.json` file consumed by the Lookup activity

---

## Azure Data Factory Pipelines

### Pipeline Design
A single dynamic, parameter-driven Copy pipeline replaces what would otherwise be N separate static pipelines:

```
Lookup Activity (reads params.json from ADLS)
        │
        ▼
ForEach Activity (iterates array of {sourceUrl, syncFile} pairs)
        │
        ▼
   Copy Activity (HTTP source → Azure Blob Storage sink), fully parameterized
```

### Linked Services
| Name | Type | Auth |
|---|---|---|
| `LS_ADLS` | Azure Blob Storage | System-Assigned Managed Identity |
| `LS_HTTP` | HTTP (public REST/raw file endpoint) | Anonymous (demo source) |
| `LS_KeyVault` | Azure Key Vault | System-Assigned Managed Identity |

### Pipeline-Scoped Parameters
- `backfillFlag` (string/boolean) — enables re-processing historical data on demand without redeploying code
- `incrementalFlag` (string/boolean) — toggles incremental vs. full load at runtime

These are intentionally **not** hardcoded into the pipeline logic — they're overridden manually only during on-demand runs (e.g., backfills), never during standard scheduled/deployed execution.

### Trigger
- Schedule trigger, daily recurrence, started/stopped safely across environments via the pre/post deployment PowerShell scripts (so deployments never silently disable or duplicate a running trigger).

---

## Dynamic Parameterization

Rather than hardcoding source URLs and sink filenames into the pipeline, a `params.json` file stored in Azure Blob Storage (`parameters` container) drives the ForEach loop:

```json
{
  "params": [
    { "sourceUrl": "<raw-file-endpoint>/bookings.csv", "syncFile": "bookings.csv" },
    { "sourceUrl": "<raw-file-endpoint>/passengers.csv", "syncFile": "passengers.csv" }
  ]
}
```

**Why this matters:** adding a new source file becomes a **data change** (edit the JSON in the data lake), not a **code change** (edit and redeploy the pipeline) — exactly how production pipelines are designed to minimize deployment risk.

### ARM Parameter Overrides (per environment)
Each environment has its own override file:
```
arm-params/
 ├── dev.json
 ├── qa.json
 └── prod.json
```
Each file overrides the linked-service connection properties (storage account name, Key Vault name, API base URL) so the same ARM template deploys correctly into each environment without pointing back at Dev resources by mistake — the single most important correctness check in the entire pipeline.

---

## Deployment Process

### Pre/Post Deployment Script
Microsoft ships a PowerShell script (`ADFDeploy.ps1`, generated automatically alongside the ARM template artifact) that solves a specific limitation: **ARM templates cannot safely delete objects removed from source** (ADF is a "living" infrastructure that doesn't natively support incremental object deletion via ARM alone).

The same script runs twice with different flags:

| Run | Flag | Purpose |
|---|---|---|
| Pre-deployment | `-predeployment $true` | Stops currently running ADF triggers so they aren't left in a broken state mid-deploy |
| Post-deployment | `-predeployment $false -deleteDeployment $true` | Deletes any pipeline/dataset/trigger objects removed from source, then restarts triggers that were previously running |

### Deployment Task Reference
```yaml
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    azureResourceManagerConnection: '${{ parameters.azureResourceManagerConnection }}'
    subscriptionId: '$(subscriptionId)'
    resourceGroupName: '$(resourceGroup)'
    csmFile: '$(Pipeline.Workspace)/arm-template/ARMTemplateForFactory.json'
    csmParametersFile: '$(Build.Repository.LocalPath)/cicd/arm-params/${{ parameters.environment }}.json'
    deploymentMode: 'Incremental'
```

---

## Security Implementation

- ✅ **Zero hardcoded secrets** — all auth via System-Assigned Managed Identity
- ✅ **Least-privilege service connection** — Azure DevOps service connection (auto-generated Service Principal / App Registration) scoped to only the required resource groups
- ✅ **Branch protection** — `main` cannot be committed to directly; PR + review required
- ✅ **Environment isolation** — separate Key Vault, storage, and RBAC per environment; no cross-environment secret sharing
- ✅ **Manual approval gate** — Production deployment cannot proceed without explicit human sign-off via Azure DevOps Environments
- ✅ **Parameters file never auto-uploaded** — production parameter overrides are uploaded manually by a senior reviewer rather than through an automated CLI task, specifically to prevent a bad parameter change from silently breaking production (a deliberate "automation vs. safety" trade-off)

---

## Folder Structure

```
adf-cicd-project/
├── README.md                          # This file
├── logs/
│   ├── WEEK1.md
│   ├── WEEK2.md
│   ├── WEEK3.md
│   ├── WEEK4.md
│   └── WEEK5.md
├── linkedTemplates/                   # (generated) large-ARM-template linked files
├── globalParameters/                  # (generated) ADF global parameters
├── cicd/
│   ├── ci-cd-pipeline.yaml            # Parent/orchestrator pipeline
│   ├── ci-build.yaml                  # Reusable CI (validate + export) template
│   ├── cd-deploy.yaml                 # Reusable CD (deploy) template
│   └── arm-params/
│       ├── dev.json
│       ├── qa.json
│       └── prod.json
├── package.json                       # Node dependency manifest (ADF utilities)
└── ARMTemplateForFactory.json         # (generated at build time — CI artifact)
```

---

## Technologies

| Category | Tools |
|---|---|
| Cloud Platform | Microsoft Azure |
| Orchestration | Azure Data Factory (ADF) |
| Storage | Azure Data Lake Storage Gen 2 |
| Secrets | Azure Key Vault |
| CI/CD | Azure DevOps Pipelines (YAML) |
| Source Control | Git / Azure Repos |
| IaC | ARM Templates (Azure Resource Manager) |
| Scripting | PowerShell (pre/post deployment), Node.js/npm (build tooling) |
| Identity | Azure AD / Entra ID, System-Assigned Managed Identity, Service Principal |

---

## Prerequisites

- An active Azure subscription (Free Tier is sufficient for Dev/QA/Prod-scale demo resources)
- An Azure DevOps organization connected to the same Azure AD tenant (**important:** select the correct "Default Directory," not "Microsoft Account," when creating the org, or the organization will disappear on next login)
- Basic familiarity with Azure Data Factory UI (linked services, datasets, pipelines)
- Node.js-capable build agent (handled automatically by the Microsoft-hosted Ubuntu agent — no local install needed)
- Requested and approved **Parallel Jobs** allocation in Azure DevOps (free tier: 1 parallel job / 1,800 minutes per month — must be explicitly requested via Microsoft's parallelism request form)

---

## Installation Guide

1. **Create Resource Groups:** `ADF-CICD-Dev`, `ADF-CICD-QA`, `ADF-CICD-Prod`
2. **Provision per environment:** Data Factory (V2) → Storage Account (Azure Blob Storage, hierarchical namespace ON) → Key Vault
3. **Grant Managed Identity access:**
   - Storage: IAM → Add role assignment → `Storage Blob Data Contributor` → select Data Factory's managed identity
   - Key Vault: Access Configuration → `Vault access policy` → grant Get/List secret permissions to the same identity
4. **Create Azure DevOps project** → enable **Repos** and **Pipelines**
5. **Create Git repository**, connect ADF (Dev) to it via **Manage → Git Configuration** (Collaboration branch: `main`)
6. **Apply branch policy** on `main` (min. 1 reviewer)
7. **Create Service Connection** (Project Settings → Service Connections → Azure Resource Manager → scoped to relevant resource groups)
8. **Create Variable Groups** in Library: `dev-group`, `qa-group`, `prod-group` (resourceGroup, dataFactoryName, subscriptionId)
9. **Create Environment** `PROD` under Pipelines → Environments, add an **Approval** check
10. **Commit the `cicd/` YAML files** and `package.json`, then create the pipeline pointing at `ci-cd-pipeline.yaml`

---

## Configuration Guide

**`package.json`** (minimal — pulls in the official ADF build utilities):
```json
{
  "scripts": { "build": "node node_modules/@microsoft/azure-data-factory-utilities/lib/index" },
  "dependencies": { "@microsoft/azure-data-factory-utilities": "^1.0.0" }
}
```

**Variable Group fields (per environment):**
| Variable | Example |
|---|---|
| `resourceGroup` | `ADF-CICD-QA` |
| `dataFactoryName` | `adf-project-qa` |
| `subscriptionId` | `<subscription-guid>` |

**Service Connection naming:** avoid hyphens/special characters in the name referenced inside YAML — this caused a real deployment failure during this project (see [Challenges](#challenges--resolutions)).

---

## Screenshots

> _Add screenshots here before publishing:_
- [ ] Architecture diagram (exported from draw.io / Excalidraw)
- [ ] ADF Studio — Git configuration screen
- [ ] Azure DevOps — Branch policy configuration
- [ ] Pull Request approval screen
- [ ] CI pipeline run — green checkmarks
- [ ] CD pipeline — Dev/QA parallel deploy + Prod approval gate
- [ ] Key Vault — Access Configuration screen
- [ ] Managed Identity — Object ID confirmation
- [ ] ARM template artifact contents

---

## Testing

| Test | Method | Result |
|---|---|---|
| Feature isolation | Created 2 parallel feature branches, confirmed zero pipeline visibility across branches | ✅ Pass |
| Branch protection | Attempted direct push to `main` | ✅ Blocked as expected |
| CI export | Ran `npm run build validate` / `export` against a syntactically invalid pipeline | ✅ Correctly failed validation |
| Managed Identity auth | Ran `Test Connection` on ADLS + Key Vault linked services post-deploy | ✅ Pass, no credentials used |
| Environment isolation (critical test) | After QA deployment, inspected linked service properties in ADF QA Studio | ✅ Confirmed pointing to QA storage/Key Vault, not Dev |
| Approval gate | Attempted Prod deployment without approving the environment check | ✅ Pipeline correctly paused, resumed only after manual approval |
| Trigger continuity | Verified previously-running triggers auto-restarted post-deployment via pre/post script | ✅ Pass |
| Incremental deployment mode | Confirmed unrelated resources in the resource group were not deleted after deploy | ✅ Pass |

---

## Challenges & Resolutions

| Challenge | Root Cause | Resolution |
|---|---|---|
| Azure DevOps organization "disappearing" after logout | Organization created under "Microsoft Account" context instead of "Default Directory" | Reconnected via Organization Settings → Microsoft Entra → switched directory |
| `variable group could not be found` | Variable group name in YAML didn't match the actual name created in Library | Standardized naming convention (`dev-group`, `qa-group`, `prod-group`) across all YAML references |
| YAML `unexpected value 'variables'` errors | Incorrect indentation — stage-level list item (`-`) placement broke the YAML parser | Corrected indentation so `variables:` sat correctly nested under the stage, not as a second list item |
| Deployed ADF pointing at wrong environment's storage/Key Vault | ARM parameters file was hardcoded to `qa.json` instead of being passed dynamically | Introduced an `environment` pipeline parameter and referenced it dynamically: `arm-params/${{ parameters.environment }}.json` |
| `Resource 'ADF-QA' under resource group 'Dev' not found` | Variable group import mistake — Dev stage was importing the QA group | Fixed variable group reference per stage |
| Service connection reference failure | Used the connection's internal ID instead of its display name; also had a stray hyphen in the name | Referenced the service connection by its exact name; removed special characters |
| Prod deployment failed — parameters file not found | Uploaded a `.txt` file instead of `.json` to the `parameters` container | Re-uploaded correct `params.json`; added this as a documented manual QA step before every Prod run |
| `deploy to QA` stage listed twice / duplicate job name error | Copy-paste error while duplicating the deploy job for QA/Prod | Renamed jobs uniquely (`DeployToQA`, `DeployToProd`) |
| Prod deployed before QA (wrong `dependsOn`) | Incorrect stage dependency chain | Set `dependsOn: DeployToQA` explicitly on the Production stage |

---

## Future Enhancements

- [ ] Migrate resource provisioning itself (Resource Groups, Storage, Key Vault) into a "bootstrap" IaC pipeline (Bicep/Terraform) instead of manual portal setup
- [ ] Add automated data-quality tests post-deployment (e.g., row-count validation) before promoting Dev → QA
- [ ] Integrate Azure Monitor + Log Analytics alerts for trigger failures
- [ ] Add a rollback stage (redeploy previous ARM template artifact) triggered on failed post-deployment validation
- [ ] Move Key Vault access fully to RBAC (from Vault Access Policy) once compatible with all ADF Web Activity scenarios
- [ ] Parameterize the Node/npm build step versions to remove the current version-pinning warning
- [ ] Add unit tests for pipeline JSON via a custom validation script beyond the built-in `build validate`

---

## Learning Outcomes

By the end of this project, the following were demonstrated with working, tested implementations (not just conceptual understanding):

- Designing and reasoning about a full CI/CD architecture **before writing any YAML** — stages, jobs, and environment boundaries mapped out first
- Git branching discipline in a UI-driven tool (ADF) that most engineers only associate with code editors
- The distinction between **compile-time parameters** (`${{ }}`) and **runtime variables** (`$( )`) in Azure Pipelines YAML — and why it matters
- Why **Incremental** deployment mode is the safe default for ARM template deployment
- Why **object deletion** is a special case in ADF CI/CD, and how Microsoft's pre/post deployment script solves it
- Practical debugging of YAML indentation errors, variable group mismatches, and service-connection reference errors — all real failures encountered and fixed during this build
- The trade-off between **full automation** and **deliberate manual checkpoints** (e.g., choosing not to automate the Prod parameters file upload, to keep a human in the loop for the highest-risk file in the pipeline)

---

## Weekly Logs

Detailed day-by-day and technical-decision logs for each week of the project are maintained separately:

| Week | Focus | Log |
|---|---|---|
| Week 1 | Foundations — Azure setup, resource provisioning, architecture design | [`logs/WEEK1.md`](logs/WEEK1.md) |
| Week 2 | Azure DevOps, Git branching, RBAC, Managed Identity | [`logs/WEEK2.md`](logs/WEEK2.md) |
| Week 3 | Linked services, dynamic pipeline design, parameterization | [`logs/WEEK3.md`](logs/WEEK3.md) |
| Week 4 | Triggers, manual publish, ARM templates, CI automation | [`logs/WEEK4.md`](logs/WEEK4.md) |
| Week 5 | CD pipeline, QA/Prod deployment, approval gates, wrap-up | [`logs/WEEK5.md`](logs/WEEK5.md) |

---

## License

This project is documented under the MIT License. Feel free to fork, adapt, and reuse the pipeline structure for your own ADF CI/CD implementation.

---

## Conclusion

This project moves Azure Data Factory from a "click-and-publish" UI tool to a fully version-controlled, environment-isolated, approval-gated deployment pipeline — the same operating model used by data engineering teams running production workloads. Every design decision documented here (Incremental deployment mode, Managed Identity over keys, manual Prod parameter uploads, pre/post trigger scripts) reflects a real trade-off encountered and resolved during the build, not a theoretical best practice copied from documentation.
