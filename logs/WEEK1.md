# Week 1 — Foundations: Understanding CI/CD & Azure Environment Setup

![Week](https://img.shields.io/badge/Week-1%2F5-blue?style=flat-square)
![Focus](https://img.shields.io/badge/Focus-Architecture%20%26%20Setup-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Completed-success?style=flat-square)

## 🎯 Sprint Goal
Understand the *why* behind CI/CD for data engineering before touching any tooling, then stand up the raw Azure infrastructure (three environments) that the rest of the project builds on.

## 📋 Scope for the Week
- Understanding CI/CD conceptually (DevOps vs. DataOps)
- Studying the target end-to-end architecture
- Requirement analysis for a 3-environment setup
- Azure account setup
- Resource Group creation (Dev / QA / Prod)
- Azure Data Factory instance creation (all 3 environments)
- Storage Account (ADLS Gen2) provisioning
- Key Vault provisioning
- Initial Git conceptual groundwork
- Project planning and folder structure design

---

## 🗓️ Daily Breakdown

### Day 1 — CI/CD Fundamentals & Why Data Engineers Need It
- Studied the distinction between **DevOps** (general software delivery) and **DataOps** (the data-engineering-specific subset most senior data engineers own directly, without a dedicated DevOps team in the loop).
- Key realization: CI/CD for ADF isn't just "deploying code" — it's deploying **pipelines, linked services, datasets, storage accounts, and access policies together**, because ADF pipelines are only meaningful in the context of the infrastructure they point at.
- Documented why this skill shows up in nearly every data engineering job description: engineers who understand deployment write more modular, environment-agnostic pipelines from day one.

