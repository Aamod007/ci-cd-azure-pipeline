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

