# Week 5 â€” Continuous Deployment: QA, Production & Approval Gates

![Week](https://img.shields.io/badge/Week-5%2F5-blue?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-CD%20Pipeline%20%26%20Wrap--up-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

## ðŸŽ¯ Sprint Goal
Close the loop: take the ARM template artifact produced in Week 4 and promote it safely through QA into an approval-gated Production deployment â€” the payoff for every architectural decision made in Weeks 1â€“4.

## ðŸ“‹ Scope for the Week
- QA and Production resource provisioning
- Variable Groups (per-environment)
- Service Connection (Azure Resource Manager)
- ARM parameter override files per environment
- `cd-deploy.yaml` reusable deployment template
- Pre/post deployment PowerShell script integration
- Manual approval gate (Azure DevOps Environments)
- Full pipeline dry run: Dev â†’ QA â†’ Prod
- Final documentation and internship summary

## ðŸ“¸ Visuals
![CD Pipeline](../images/cd-pipeline.png)

---

## ðŸ—“ï¸ Daily Breakdown

### Day 1 â€” QA & Production Resource Provisioning
- Provisioned QA and Production environments identically to Dev's Week 1 setup: Data Factory, Azure Blob Storage (hierarchical namespace enabled), Key Vault â€” each in its own resource group (`ADF-CICD-QA`, `ADF-CICD-Prod`).
- Repeated the Week 2 RBAC pattern for both new environments: Managed Identity â†’ `Storage Blob Data Contributor`, Key Vault â†’ Access Policy â†’ Get/List permissions.
- **Consistency check:** deliberately re-used the exact same naming convention across environments (`adf-<project>-{env}`) so that ARM parameter overrides later would only need to change a predictable, minimal set of values.

### Day 2 â€” Variable Groups & Service Connection
- Created three Library Variable Groups: `dev-group`, `qa-group`, `prod-group`, each holding `resourceGroup`, `dataFactoryName`, and `subscriptionId` for its environment.
- Created the Azure DevOps **Service Connection** (Azure Resource Manager type), which auto-provisions a Service Principal / App Registration in Azure AD.
- **Scoping decision:** initially scoped the connection to a single resource group, then widened it to cover all three project resource groups â€” documented trade-off: one service connection with broad scope is operationally simpler to maintain than three narrowly-scoped connections, appropriate for a project of this size; a stricter, single-resource-group-per-connection model is noted as more appropriate for larger, multi-team, higher-sensitivity projects.

### Day 3 â€” ARM Parameter Override Files
- Created `cicd/arm-params/dev.json`, `qa.json`, and `prod.json`, each cloned from the Week 4 published `ARMTemplateParametersForFactory.json` and edited to point at the correct environment's Key Vault, storage account, and API endpoint values.
- This is the file set responsible for preventing the most damaging class of deployment bug in this entire project: **a Data Factory in one environment silently pointing at another environment's storage or secrets.**

### Day 4 â€” Building `cd-deploy.yaml` (Reusable Deployment Template)
- Authored the reusable deploy template, parameterized by `dataFactoryName`, `resourceGroup`, `subscriptionId`, `environment`, and `azureResourceManagerConnection`.
- Step sequence:
  1. `DownloadPipelineArtifact@2` â€” pulls the `ARMTemplate` artifact published by the CI stage
  2. `AzurePowerShell@5` (pre-deployment) â€” runs the Microsoft-provided script with `-predeployment $true`, stopping any currently running triggers before deployment touches the factory
  3. `AzureResourceManagerTemplateDeployment@3` â€” deploys in **Incremental** mode, using `arm-params/${{ parameters.environment }}.json` as the override file
  4. `AzurePowerShell@5` (post-deployment) â€” runs the same script with `-predeployment $false -deleteDeployment $true`, removing orphaned objects and restarting previously-active triggers

