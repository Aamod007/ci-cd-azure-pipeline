# Week 4 — Triggers, Manual Publish, ARM Templates & CI Automation

![Week](https://img.shields.io/badge/Week-4%2F5-blue?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-CI%20Pipeline-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

## 🎯 Sprint Goal
Understand ARM template generation from both angles — the manual "Publish" button first, then the fully automated CI pipeline equivalent — before relying on automation blindly.

## 📋 Scope for the Week
- Schedule trigger creation
- Manual Publish workflow (deep dive)
- ARM template structure inspection
- Introduction to `@microsoft/azure-data-factory-utilities`
- Azure DevOps Parallel Jobs request
- YAML pipeline fundamentals (stages → jobs → steps)
- Building and debugging the CI (`ci-build.yaml`) pipeline

---

## 🗓️ Daily Breakdown

### Day 1 — Schedule Trigger
- Added a daily **Schedule Trigger** to the pipeline built in Week 3.
- Deliberately left the trigger **stopped** at creation time — trigger state management is handled explicitly later by the pre/post deployment script, so leaving it in a known "off" state avoided confusion during this week's manual-publish testing.