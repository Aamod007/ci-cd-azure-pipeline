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