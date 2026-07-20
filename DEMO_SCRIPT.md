# CI/CD Demo Script & Checklist

This document is designed to guide you through presenting the Azure Data Factory CI/CD pipeline to your team or reviewers.

## 🎯 The Scenario
"We are going to add a critical Error Alerting feature to our pipeline. We will do this safely in an isolated branch, and watch our automated pipeline promote it all the way to Production."

## 📝 Demo Checklist

### 1. Setup & Proof of Isolation
- [ ] Open the Azure Portal.
- [ ] Show the 3 distinct Resource Groups (Dev, QA, Prod).
- [ ] Open the **Dev** and **Prod** Data Factory Studios side-by-side. 
- [ ] Point out that they currently look identical but are physically separated.

### 2. Development (The Code Change)
- [ ] In the **Dev** ADF Studio, create a new branch: `feature/add-error-alerts`.
- [ ] Open your main pipeline.
- [ ] Add a **Web Activity** to the canvas named `Send_Alert`.
- [ ] Connect your main Copy/Data flow activity to this Web Activity using the **Red "On Failure" arrow**.
- [ ] Explain: *"If our pipeline fails, it will now trigger an alert. This is a standard enterprise pattern."*
- [ ] Save the changes in your branch.

### 3. Continuous Integration (CI)
- [ ] Go to Azure Repos and create a Pull Request from `feature/add-error-alerts` to `main`.
- [ ] Show the PR description to the audience.
- [ ] Approve and Merge the PR.
- [ ] Jump to **Azure Pipelines**. Show the CI pipeline spinning up automatically to validate and generate the ARM template.

### 4. Continuous Deployment (CD)
- [ ] Once CI finishes, show the CD pipeline kicking off automatically.
- [ ] Watch it deploy to the **Dev** environment.
- [ ] Watch it seamlessly promote to the **QA** environment using `qa.json` parameters.

### 5. Production Approval Gate
- [ ] Show the pipeline **pausing** at the Production stage.
- [ ] Explain: *"We never push to Production blindly. Our pipeline requires a manual sign-off."*
- [ ] Go to **Environments** in Azure DevOps and manually **Approve** the release.
- [ ] Watch the final deployment to Production complete.

### 6. The Big Reveal
- [ ] Go back to the **Production** ADF Studio.
- [ ] Refresh the page.
- [ ] Show the audience the new `Send_Alert` activity sitting on the canvas, proving the code successfully and securely traveled through the entire pipeline.
