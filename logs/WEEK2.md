# Week 2 — Azure DevOps, Git Branching Strategy & Access Control

![Week](https://img.shields.io/badge/Week-2%2F5-blue?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Git%20%26%20RBAC-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

## 🎯 Sprint Goal
Move Azure Data Factory from an ungoverned "Live Mode only" state into a fully Git-backed, branch-protected workflow, and resolve every access-control dependency (Managed Identity → Storage, Managed Identity → Key Vault) needed before any real pipeline development can start.

## 📋 Scope for the Week
- Azure DevOps organization and project setup
- Git repository creation (Azure Repos)
- Branching strategy design (`main` + `feature/*`)
- Branch policies (PR + reviewer enforcement)
- Pull request workflow
- System-Assigned Managed Identity configuration
- Role-Based Access Control (RBAC) — Storage Blob Data Contributor
- Key Vault access resolution (RBAC vs. Access Policy)

---

## 🗓️ Daily Breakdown

### Day 1 — Azure DevOps Organization Setup
- Created the Azure DevOps organization, with a specific gotcha resolved immediately: the org must be created (or reconnected) under the **"Default Directory"** in the Microsoft account selector, not left on "Microsoft Account" — otherwise the organization silently disappears from view on the next login.
- Verified this via **Organization Settings → Microsoft Entra**, confirming the directory binding before proceeding.
- Created the Azure DevOps project (`ADF-CICD`) using the **Agile** process template, with **Git** as the version control backend.

### Day 2 — Repository Creation & Initial Structure
- Created a dedicated Git repository (`ADF-CICD-Repo`) inside the project, separate from the default auto-created repo, with a starter README commit.
- Confirmed the repository started with a single `main` branch and zero pipeline history — the clean baseline the ADF Git integration will sync into.
- Disabled unused Azure DevOps surface area (Releases, Task Groups, Deployment Groups) in Project Settings, since this project standardizes on **YAML pipelines**, which Microsoft has explicitly been recommending over classic Release Pipelines since well before this project started.

### Day 3 — Connecting ADF to Git
- From ADF Studio → **Manage → Git Configuration**, connected the Dev Data Factory to the new Azure Repos repository.
- Configured the two critical properties:
  - **Collaboration branch:** `main`
  - **Publish branch:** `adf_publish` (auto-managed, not human-edited)
- Confirmed via the branch dropdown in ADF Studio that `main` and `adf_publish` now both existed, with `main` reflecting Live Mode's current (empty) state.

### Day 4 — Branch Policy Enforcement
- Applied a branch policy on `main`:
  - **Minimum reviewer count: 1**
  - **Allow requester to approve their own changes: enabled** (single-developer setting for this project; documented as something to disable in a real multi-developer team)
- Verified the policy was live by confirming `main` could no longer be edited directly from ADF Studio without going through a feature branch + PR.

### Day 5 — First Feature Branch & Pull Request Workflow
- Created `feature/branch` directly from ADF Studio's branch dropdown, confirming it branched cleanly off `main` with zero pipeline conflicts against any other in-progress branch.
- Made a trivial change (empty pipeline) in the feature branch, then walked the full PR lifecycle end-to-end for the first time: **Create PR → Reviewer approval → Complete (fast-forward merge)** — confirming the merged change appeared in `main` immediately and the feature branch was auto-deleted post-merge, as expected.

### Day 6 — Managed Identity → Storage RBAC
- Located the Dev Data Factory's **System-Assigned Managed Identity** under Settings → Identity (Object ID confirmed — critical to verify this is the *system-assigned* identity, not accidentally a *user-assigned* one, which is not the recommended pattern).
- Created the `raw` container in the Dev ADLS Gen2 account.
- Granted the identity the **Storage Blob Data Contributor** role via **IAM → Add role assignment**, selecting "Managed Identity" as the member type and filtering by the Data Factory's name specifically (a step that's easy to fumble if you search under the wrong member category).

