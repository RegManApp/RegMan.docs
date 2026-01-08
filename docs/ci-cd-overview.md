# RegMan CI/CD Overview (Multi‑Repo)

Date: 2026-01-08

This document is a **read-only, factual** overview of how CI/CD is structured across RegMan’s **four separate Git repositories**, and which GitHub Actions workflows exist and run in practice.

## 1) High-level architecture

RegMan is split into four repos:

1. **Frontend repo** (React / Vite)
2. **Backend repo** (ASP.NET Core)
3. **Docs repo** (documentation)
4. **.github repo** (org-level/shared documentation and optionally reusable workflows)

### Where CI/CD runs in practice

- **CI/CD is triggered and executed from the Frontend repo and Backend repo only.**
- The **Docs repo** does not run CI/CD.
- The **.github repo does not orchestrate** CI/CD for the Frontend/Backend repos.

This is a common multi-repo setup: each deployable system owns its own pipeline, so changes to one repo do not require workflows to exist or run in another repo.

## 2) Pipeline flows (plain English)

### Frontend CI/CD flow

- Developer pushes code to the **Frontend repo** (or opens a PR).
- GitHub Actions runs the frontend workflow.
- The workflow builds the Vite app.
- The workflow deploys the built `dist/` output to **Cloudflare Pages**.

### Backend CI/CD flow

- Developer pushes code to the **Backend repo** (or opens a PR).
- GitHub Actions runs the backend CI workflow (build + publish artifact).
- Separately, the backend deploy workflow runs on pushes and:
  - Restores dependencies
  - Applies EF Core migrations
  - Builds, tests, publishes
  - Deploys to the hosting environment via WebDeploy (MonsterASP.NET)

## 3) Workflow inventory (authoritative)

The table below lists **every GitHub Actions workflow file currently present in the workspace**, by repo.

| Repo     | Workflow file                   | Trigger                                                                    | Purpose                                                                                 | CI/CD               | Runs in practice?                       |
| -------- | ------------------------------- | -------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------------- | --------------------------------------- |
| Backend  | `.github/workflows/dotnet.yml`  | `push` to `main` (path-filtered), `pull_request` to `main` (path-filtered) | Build backend, publish artifact (and includes static analysis steps if present in file) | CI                  | Yes (runs in Backend repo)              |
| Backend  | `.github/workflows/publish.yml` | `push` (path-filtered)                                                     | Restore, migrate DB, build/test/publish, deploy via WebDeploy                           | CD (and build/test) | Yes (runs in Backend repo)              |
| Frontend | `.github/workflows/deploy.yml`  | `push` to `main` (path-filtered), `pull_request` to `main` (path-filtered) | Build Vite app and deploy to Cloudflare Pages                                           | CD (and build)      | Yes (runs in Frontend repo)             |
| Docs     | _(none found)_                  | N/A                                                                        | N/A                                                                                     | N/A                 | No (no workflows)                       |
| .github  | _(no workflow files found)_     | N/A                                                                        | N/A                                                                                     | N/A                 | No (does not run CI/CD for other repos) |

### Notes on triggers (what “runs in practice” means)

“Runs in practice” here means:

- The workflow file exists **inside the same repository** where the events (`push`, `pull_request`) occur.
- Therefore GitHub Actions will evaluate it when those events happen in that repo.

## 4) Why the `.github` repo does not run other repos’ CI

### What a `.github` repo is normally used for

A repository named `.github` (typically under an organization account) is commonly used for:

- Shared documentation and organization-level files (issue templates, PR templates, CODEOWNERS)
- Optionally, **reusable workflows** that other repos can call

### Push-triggered workflows vs reusable workflows

- **Push/PR-triggered workflows** (`on: push`, `on: pull_request`) run when events happen **in that same repo**.
- **Reusable workflows** are designed to be called from another repo using `workflow_call`:
  - A “caller” workflow in Repo A runs on push/PR.
  - That workflow calls a reusable workflow defined in Repo B.

### Why RegMan does not duplicate CI in `.github`

In this architecture:

- The Frontend and Backend repos are the **sources of truth** for their own CI/CD.
- Duplicating the same CI/CD logic in `.github` would be misleading and create parallel pipelines.
- If shared CI logic is ever desired, the safe approach is to:
  - Keep push/PR triggers in Frontend/Backend
  - Move only common steps into reusable workflows (`workflow_call`) in `.github`

## 5) Best-practice justification

- **Frontend CI stays in the Frontend repo** because it is the deployable unit, has its own dependencies and deploy target, and changes there should not require coordination with other repos.
- **Backend CI stays in the Backend repo** for the same reason: it is independently deployable with its own build/test/migration/deploy process.
- **Shared knowledge belongs in Docs** (this repo) and shared pipeline code—if ever required—belongs in **reusable workflows** rather than duplicating push-triggered pipelines across repos.
