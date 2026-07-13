# Week 3 — Linked Services, Dynamic Pipeline Design & Parameterization

![Week](https://img.shields.io/badge/Week-3%2F5-blue?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Pipeline%20Engineering-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

## 🎯 Sprint Goal
Build a single, reusable, parameter-driven ADF pipeline (instead of one static pipeline per source file) — the pattern that makes environment promotion in later weeks actually meaningful, since a hardcoded pipeline can't safely move between environments.

## 📋 Scope for the Week
- Linked Service creation: Azure Blob Storage, HTTP, Key Vault
- Dataset design (parameterized, not hardcoded)
- Lookup Activity — reading a runtime configuration file
- ForEach Activity — dynamic iteration
- Copy Activity — HTTP → Azure Blob Storage
- Pipeline-level parameters for on-demand runtime overrides

---

## 🗓️ Daily Breakdown

### Day 1 — Linked Service: Azure Blob Storage
- Created `LS_ADLS`, authenticated via **System-Assigned Managed Identity** (per the access grant completed in Week 2) — no account key entered anywhere.
- Ran **Test Connection** immediately after creation to confirm the RBAC role assignment from Week 2 had actually propagated (Azure role assignments can take a few minutes to become effective — this was factored into the testing sequence).

### Day 2 — Linked Service: HTTP Source
- Created `LS_HTTP`, pointing at a public raw-file endpoint used as the demo data source for this project (two CSV files: `bookings.csv`, `passengers.csv`).
- Used **Anonymous** authentication since the demo source requires none — but deliberately still built out the **Key Vault linked service** in parallel this same day, specifically to simulate and document the production-realistic pattern (most real HTTP/API sources are *not* anonymous and would pull credentials from Key Vault via this exact linked service).

### Day 3 — Linked Service: Key Vault + First Secret Retrieval Test
- Created `LS_KeyVault`, authenticated via Managed Identity.
- Verified with **Test Connection** — confirmed the Week 2 access-policy fix was correctly in place; no secrets are strictly required for the demo source, but the linked service is fully wired for the case where they are.

### Day 4 — Runtime Parameter File Design
- Designed the `params.json` structure to drive the pipeline dynamically:
  ```json
  {
    "params": [
      { "sourceUrl": "<endpoint>/bookings.csv", "syncFile": "bookings.csv" },
      { "sourceUrl": "<endpoint>/passengers.csv", "syncFile": "passengers.csv" }
    ]
  }
  ```
- **Design rationale:** wrapping the array in a `params` key (rather than a bare top-level array) makes it far easier to reference cleanly inside a Lookup Activity's dynamic-content expressions later.
- Uploaded `params.json` to a new `parameters` container in Azure Blob Storage (Dev).

### Day 5 — Lookup Activity + ForEach Loop
- Built the `Get Parameters` **Lookup Activity**, sourced from a new parameterized JSON dataset (`DS_Parameters`) pointed at the uploaded `params.json`.
- Debugged the Lookup output shape directly (`value[0].params`) before wiring it into the loop — confirmed via a manual debug run rather than assuming the JSON structure would map cleanly.
- Built the **ForEach Activity**, with `Items` set to the dynamic expression referencing the Lookup output's `params` array — this is what allows the loop to process an arbitrary number of source files without any pipeline changes.

### Day 6 — Copy Activity (Parameterized Source + Sink)
- Inside the ForEach loop, built the **Copy Activity**:
  - **Source dataset** (`DS_Source`, HTTP/CSV) — parameterized with a `URL` dataset parameter, populated at runtime via `@item().sourceUrl`
  - **Sink dataset** (`DS_Sink`, Azure Blob Storage/CSV) — parameterized with a `file` dataset parameter, populated via `@item().syncFile`, writing into the `raw` container
- Ran a full debug pass — both files (`bookings.csv`, `passengers.csv`) landed correctly in the `raw` container on the first successful run after resolving a minor dynamic-content reference typo.

### Day 7 — Pipeline-Level Parameters (Runtime Overrides)
- Added two pipeline-scoped parameters, deliberately **not wired into the deployed environment-config path**:
  - `backfillFlag` — allows a developer/operator to trigger historical reprocessing on demand
  - `incrementalFlag` — toggles incremental vs. full load
- **Design decision documented:** these are override-only parameters set manually at debug/trigger time for special-case runs (e.g., backfills) — they are never meant to differ between Dev/QA/Prod as part of a standard deployment, which is why they live as pipeline parameters rather than in the ARM parameter-override files used later.

---

## 🏗️ Objects Built This Week

| Object | Type | Notes |
|---|---|---|
| `LS_ADLS` | Linked Service | Managed Identity auth |
| `LS_HTTP` | Linked Service | Anonymous (demo source) |
| `LS_KeyVault` | Linked Service | Managed Identity auth, built for future secrets |
| `DS_Parameters` | Dataset | JSON, points at `params.json` |
| `DS_Source` | Dataset | HTTP/CSV, parameterized `URL` |
| `DS_Sink` | Dataset | Azure Blob Storage/CSV, parameterized `file` |
| `Get Parameters` | Lookup Activity | Reads `params.json` |
| `ForEach Loop` | ForEach Activity | Iterates `params` array |
| `Copy Activity` | Inside ForEach | HTTP → Azure Blob Storage, fully dynamic |
| Pipeline parameters | `backfillFlag`, `incrementalFlag` | Runtime-only overrides |

---

## 🧠 Technical Decisions

| Decision | Reasoning |
|---|---|
| One dynamic pipeline instead of N static pipelines | Adding a new source becomes a data change (edit JSON), not a code change (edit + redeploy pipeline) |
| Config-driven via ADLS-stored JSON, not hardcoded pipeline parameters | In production, source lists change frequently (new URL requests are common) — storing config in the data lake avoids a redeploy for every list change |
| Wrapping the params array in a `params` key | Simplifies dynamic-content expressions when referencing nested Lookup output |
| Keeping `backfillFlag`/`incrementalFlag` as pipeline parameters, not ARM-deployed variables | These are operational, on-demand toggles — not environment configuration, so they don't belong in the Dev/QA/Prod parameter override files |

---

## 🚧 Problems & Solutions

| Problem | Solution |
|---|---|
| Lookup Activity output shape unclear before wiring into ForEach | Ran an isolated debug run first, inspected raw JSON output, then wrote the dynamic expression against the confirmed shape |
| `params.json` upload initially malformed (missing wrapping braces) | Restructured into a valid JSON object with a `params` key rather than a bare array |
| Copy Activity failed on first debug run due to a dynamic-content typo in the sink filename expression | Corrected `@item().syncFile` reference; re-ran debug to confirm |

---

## 📚 Learnings

- A pipeline that "works" for one file is not the same as a pipeline that's actually production-ready — dynamic, config-driven design is what makes environment promotion (Week 5) safe and meaningful.
- Debugging Lookup Activity output *before* wiring it into a ForEach loop saves significant time versus debugging both together.
- Separating **operational runtime parameters** (backfill/incremental flags) from **environment configuration parameters** (which storage account, which Key Vault) is a design boundary that has to be decided deliberately — conflating the two makes both the pipeline and the CI/CD parameter files harder to reason about.

## ✅ Week 3 Deliverables
- [x] All three linked services created and connection-tested
- [x] Dynamic, config-driven pipeline built and successfully debugged end-to-end
- [x] Parameterized datasets (source + sink) confirmed working
- [x] Pipeline-level runtime parameters implemented and documented

**Next week:** Add a schedule trigger, walk the manual Publish process end-to-end to understand ARM template generation, then automate that same export via the CI pipeline.