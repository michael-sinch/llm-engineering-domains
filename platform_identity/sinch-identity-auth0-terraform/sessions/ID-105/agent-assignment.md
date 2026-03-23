# Agent Assignment

**Last Updated:** 2026-03-13 10:25

---

## Session Recovery Header

> **Purpose:** Enables context recovery and cross-model handoff. Update after each major workflow step.

| Field | Value |
|-------|-------|
| Framework | ~/src/GitHub/tasks/ID-105_chatlayer_sinch_updates/llm-engineering-agent |
| Target Repo | /Users/mikbig/src/GitHub/tasks/ID-105_chatlayer_sinch_updates |
| Assignment ID | ID-105 |
| WIP Session Dir | ~/src/WIP/sinch-identity-auth0-terraform/session-ID-105-20260313-1025/ |
| Session Started | 2026-03-13 10:25 |
| Current Phase | 3 — Intake |
| Last Updated | 2026-03-13 10:25 |
| Model at Last Update | claude-sonnet-4-6 |

---

**Purpose:** Assignment template — populated per assignment by the intake action

> **Intake action:** [actions/assignment-intake.md](actions/assignment-intake.md)

---

## Table of Contents
1. [Classification of Assignment](#classification-of-assignment)
2. [Tracking ID or Label](#tracking-id-or-label)
3. [Assignment](#assignment)
4. [Scope Boundary](#scope-boundary)
5. [Open Questions & Answers](#open-questions--answers)
6. [Implementation Plan](#implementation-plan)
7. [Architecture Discovery & Validation](#architecture-discovery--validation)
8. [Data Source Analysis](#data-source-analysis)
9. [Stakeholder Validation](#stakeholder-validation)
10. [JIRA Summary](#jira-summary)

---

## Classification of Assignment
- Repository: https://github.com/mailgun/sinch-identity-auth0-terraform
- Classification: Infrastructure / Terraform (Auth0 configuration management)

## Tracking ID or Label
ID-105

## Assignment
Chatlayer Sinch Updates — changes to the sinch-identity-auth0-terraform repo:

1. **Application "Chatlayer - DEV"**: Change allowed logout URLs to include `https://app.dev.chatlayer.ai` and `http://localhost:3000`
2. **API "Chatlayer API"**: Rename to include "-DEV" suffix
3. **API "Chatlayer API"**: Set `allow_offline_access = true`

## Scope Boundary
**In scope:**
- Modify `env/staging.tfvars` to update logout URLs for Chatlayer - DEV application
- Modify `custom_apis.tf` to rename the API and enable offline access

**Out of scope:**
- Changes to production (`env/prod.tfvars`)
- Any other applications or APIs not mentioned
- Auth0 tenant-level settings

**Scope Creep Policy**: If work required appears to extend beyond the scope defined above:
1. Stop and record the potential expansion in the table below
2. Assess whether it is **critical** (blocks completion) or **significant** (improves quality but not required)
3. Notify the user and obtain explicit approval before proceeding
4. Record the decision here before continuing

| # | Potential Scope Expansion | Critical / Significant | User Decision | Approved? |
|---|--------------------------|----------------------|---------------|-----------|
| 1 | <FILL IN if applicable>  | | | |

## Open Questions & Answers

| # | Question | Answer | Status |
|---|----------|--------|--------|
| 1 | Which environment should the logout URL change apply to? | staging.tfvars only | ✅ Resolved |
| 2 | Should the API be renamed to "Chatlayer API - DEV"? | Yes — "Chatlayer API - DEV" | ✅ Resolved |
| 3 | Should logout URLs be appended or replaced? | Append to existing list | ✅ Resolved |

## Implementation Plan

- [ ] 1. Read current state of `env/staging.tfvars` around line 206 (logout URLs for Chatlayer - DEV)
- [ ] 2. Read current state of `custom_apis.tf` around line 24 and 49 (offline access flag and API name)
- [ ] 3. Update `env/staging.tfvars`: add `https://app.dev.chatlayer.ai` and `http://localhost:3000` to logout URLs
- [ ] 4. Update `custom_apis.tf` line 49: rename API to include "-DEV"
- [ ] 5. Update `custom_apis.tf` line 24: set `allow_offline_access = true`
- [ ] 6. Validate Terraform syntax (terraform fmt / validate)

## Architecture Discovery & Validation

The repository is a Terraform codebase managing Auth0 configuration:
- `custom_apis.tf` — defines Auth0 Resource Server (API) resources
- `env/staging.tfvars` — environment-specific variable values for staging
- Changes are environment-variable-driven where possible

## Data Source Analysis

- `env/staging.tfvars` contains per-environment overrides; this is where application callback/logout URLs are defined per environment
- `custom_apis.tf` defines the API resource directly; the name and offline_access setting live directly in the resource block or may use variables

## Stakeholder Validation

- Assignment sourced from ID-105 ticket
- References to specific GitHub file lines confirm exact locations

---

## JIRA Summary

**ID-105 — Chatlayer Sinch Updates**

Update the sinch-identity-auth0-terraform Terraform configuration for the Chatlayer application and API:

1. Add `https://app.dev.chatlayer.ai` and `http://localhost:3000` to allowed logout URLs for the "Chatlayer - DEV" application in staging.tfvars
2. Rename the "Chatlayer API" resource server to "Chatlayer API - DEV" in custom_apis.tf
3. Enable `allow_offline_access = true` on the Chatlayer API resource server in custom_apis.tf
