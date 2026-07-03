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

### Day 7 — Key Vault Access Resolution
- Hit the expected RBAC blocker from Week 1: attempting to create a secret returned *"operation not allowed by RBAC... please wait several minutes for role assignments to become effective."*
- Root cause identified: the Key Vault's **Access Configuration** was set to **Azure role-based access control**, which requires an explicit `Key Vault Administrator` IAM role before *any* account — including the owner — can manage secrets.
- Resolved in two steps:
  1. Assigned own account the **Key Vault Administrator** role via IAM.
  2. Switched **Access Configuration** to **Vault access policy** — required separately, because ADF's Web Activity / linked-service secret retrieval calls the Key Vault's data-plane API, which only respects Access Policies, not the management-plane RBAC role alone.
- Granted the Data Factory's Managed Identity **Get/List** secret permissions via the resulting Access Policy screen.

---

## 🏗️ Configuration Completed This Week

| Item | Status |
|---|---|
| Azure DevOps org + project | ✅ |
| Git repository created | ✅ |
| ADF (Dev) connected to Git | ✅ Collaboration: `main`, Publish: `adf_publish` |
| Branch policy on `main` | ✅ Min. 1 reviewer |
| First feature branch → PR → merge cycle | ✅ Verified end-to-end |
| Managed Identity → Storage Blob Data Contributor | ✅ Dev environment |
| Key Vault Access Configuration | ✅ Switched to Vault Access Policy |
| Managed Identity → Key Vault Get/List | ✅ Dev environment |

---

## 🧠 Technical Decisions

| Decision | Reasoning |
|---|---|
| System-Assigned over User-Assigned Managed Identity | Recommended Azure pattern — lifecycle is tied to the resource itself, no separate identity to manage or rotate |
| Vault Access Policy over pure RBAC for the Key Vault's data plane | Required specifically because ADF's runtime secret retrieval doesn't yet respect RBAC-only vaults in this configuration |
| Fast-forward merge strategy | Keeps repository history linear and easy to reason about for a small-team / solo project |
| Allow self-approval on PRs (for now) | Practical for a single-developer sprint; explicitly flagged as something to remove in a real team setting |

---

## 🚧 Problems & Solutions

| Problem | Solution |
|---|---|
| Azure DevOps org invisible after first login | Reconnected the organization under "Default Directory" via Organization Settings → Microsoft Entra |
| `main` branch editable directly, defeating the purpose of Git integration | Applied branch policy requiring PR + reviewer before merge |
| "Operation not allowed by RBAC" when creating Key Vault secrets | Assigned `Key Vault Administrator` IAM role to own account first |
| ADF Web Activity couldn't retrieve secrets even after RBAC role assignment | Switched Key Vault Access Configuration to "Vault access policy" and explicitly granted Get/List to the Managed Identity |

---

## 📚 Learnings

- Git branch protection in ADF works identically in spirit to code-repo branch protection — but is enforced entirely at the Azure DevOps repo level, since ADF Studio has no native policy engine of its own.
- RBAC and Access Policy are **not interchangeable** for Key Vault — data-plane secret operations (what ADF actually needs at runtime) depend on which access model is active, and the two can silently conflict if not deliberately chosen.
- "Managed Identity" in the IAM role-assignment picker is its own member category, separate from "User" and "Group" — an easy source of a failed role assignment on a first attempt.

## ✅ Week 2 Deliverables
- [x] Azure DevOps organization and project fully configured
- [x] ADF (Dev) Git-integrated with protected `main` branch
- [x] Feature branch → PR → merge workflow verified end-to-end
- [x] Managed Identity access to Storage and Key Vault resolved for Dev

**Next week:** Build the actual dynamic ADF pipeline — linked services, HTTP source, ADLS sink, Lookup + ForEach parameterization pattern.