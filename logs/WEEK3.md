# Week 3 — Linked Services, Dynamic Pipeline Design & Parameterization

![Week](https://img.shields.io/badge/Week-3%2F5-blue?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Pipeline%20Engineering-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=flat-square)

## 🎯 Sprint Goal
Build a single, reusable, parameter-driven ADF pipeline (instead of one static pipeline per source file) — the pattern that makes environment promotion in later weeks actually meaningful, since a hardcoded pipeline can't safely move between environments.

## 📋 Scope for the Week
- Linked Service creation: ADLS Gen2, HTTP, Key Vault
- Dataset design (parameterized, not hardcoded)
- Lookup Activity — reading a runtime configuration file
- ForEach Activity — dynamic iteration
- Copy Activity — HTTP → ADLS Gen2
- Pipeline-level parameters for on-demand runtime overrides

---

## 🗓️ Daily Breakdown

### Day 1 — Linked Service: ADLS Gen2
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
- Uploaded `params.json` to a new `parameters` container in ADLS Gen2 (Dev).

### Day 5 — Lookup Activity + ForEach Loop
- Built the `Get Parameters` **Lookup Activity**, sourced from a new parameterized JSON dataset (`DS_Parameters`) pointed at the uploaded `params.json`.
- Debugged the Lookup output shape directly (`value[0].params`) before wiring it into the loop — confirmed via a manual debug run rather than assuming the JSON structure would map cleanly.
- Built the **ForEach Activity**, with `Items` set to the dynamic expression referencing the Lookup output's `params` array — this is what allows the loop to process an arbitrary number of source files without any pipeline changes.

### Day 6 — Copy Activity (Parameterized Source + Sink)
- Inside the ForEach loop, built the **Copy Activity**:
  - **Source dataset** (`DS_Source`, HTTP/CSV) — parameterized with a `URL` dataset parameter, populated at runtime via `@item().sourceUrl`
  - **Sink dataset** (`DS_Sink`, ADLS Gen2/CSV) — parameterized with a `file` dataset parameter, populated via `@item().syncFile`, writing into the `raw` container
- Ran a full debug pass — both files (`bookings.csv`, `passengers.csv`) landed correctly in the `raw` container on the first successful run after resolving a minor dynamic-content reference typo.

---

## 🏗️ Objects Built This Week

| Object | Type | Notes |
|---|---|---|
| `LS_ADLS` | Linked Service | Managed Identity auth |
| `LS_HTTP` | Linked Service | Anonymous (demo source) |
| `LS_KeyVault` | Linked Service | Managed Identity auth, built for future secrets |
| `DS_Parameters` | Dataset | JSON, points at `params.json` |
| `DS_Source` | Dataset | HTTP/CSV, parameterized `URL` |
| `DS_Sink` | Dataset | ADLS Gen2/CSV, parameterized `file` |
| `Get Parameters` | Lookup Activity | Reads `params.json` |
| `ForEach Loop` | ForEach Activity | Iterates `params` array |
| `Copy Activity` | Inside ForEach | HTTP → ADLS Gen2, fully dynamic |

---

## 🧠 Technical Decisions

| Decision | Reasoning |
|---|---|
| One dynamic pipeline instead of N static pipelines | Adding a new source becomes a data change (edit JSON), not a code change (edit + redeploy pipeline) |
| Config-driven via ADLS-stored JSON, not hardcoded pipeline parameters | In production, source lists change frequently (new URL requests are common) — storing config in the data lake avoids a redeploy for every list change |
| Wrapping the params array in a `params` key | Simplifies dynamic-content expressions when referencing nested Lookup output |