# Testing Documentation Overview

*Standardized testing guidance for GitHub API projects (Mailgun focus)*

**Repos:** git@github.com:mailgun/gozer.git, git@github.com:mailgun/jefe.git  
**Last Updated:** March 16, 2026

---

## Document Purpose

### GITHUB-API-TESTING-STANDARDS.md (not yet created)
**Scope:** High-level testing framework for all GitHub/Mailgun API projects

**Use When:**
- Starting a new assignment
- Defining testing approach
- Understanding deployment workflows
- Learning Mailgun staging environment
- Reference for Kubernetes/GitLab alternative platform

**Key Topics:**
- Slack-based deployment (@Director-Develop for staging, @Director for production)
- Production deployment process (consistent multi-region rollout)
- Service discovery (gozer, jefe, and other Mailgun services)
- VPN setup (Pritunl)
- DuckDNS test domains (API + UI workflows)
- REST and gRPC testing approaches
- Monitoring (Kibana, Nomad, Grafana/Loki)
- Security practices (credentials, .env files)
- Test automation patterns
- Test data coordination (parallel testing, naming conventions)
- Team contacts and escalation paths
- Rollback criteria
- Auth0 Organizations API patterns
- **NEW:** Kubernetes/GitLab platform guide (Sinch Identity services)

### HOW-TO-Identity-testing-staging-github.md (not yet created)
**Scope:** Staging testing template for GitHub/Nomad projects (gozer, jefe)

**Use When:**
- Testing a gozer or jefe assignment in Mailgun staging
- Need a step-by-step post-deployment testing guide
- First time testing gozer/SAML or Mailgun domain features
- Troubleshooting issues on the Nomad platform

**Key Topics:**
- Scope and resource prerequisite checklist
- Environment configuration
- Test data setup (DuckDNS domains)
- Deployment process (Slack commands)
- Prioritized step-by-step process
- Exit conditions and rollback procedures

### HOW-TO-Identity-testing-staging-gitlab.md (not yet created)
**Scope:** Staging testing template for GitLab/Kubernetes projects (user-management-bff, internal-user-management-bff)

**Use When:**
- Testing a user-management-bff or internal-user-management-bff assignment in Sinch staging
- Need a step-by-step post-deployment testing guide for GitLab-managed services
- Troubleshooting issues on the Kubernetes/EKS platform

**Key Topics:**
- Scope and resource prerequisite checklist
- Environment configuration
- Test data setup
- GitLab CI deployment process
- Prioritized step-by-step process
- Exit conditions and rollback procedures

---

## Platform Overview

The testing standards cover **two infrastructure platforms** used by the Identity team:

### Mailgun Platform (GitHub/Nomad) - Primary Focus
- **Repos:** gozer, jefe
- **Deployment:** 
  - Staging: @Director-Develop in #eng-director-dev
  - Production: @Director in #chatops
- **Orchestration:** Nomad
- **Logs:** Kibana
- **APIs:** REST (SAML, domain management)
- **Regions:** us-east4, us-west1, europe-west1

### Sinch Identity Platform (GitLab/Kubernetes) - Alternative
- **Repos:** user-management-bff, internal-user-management-bff
- **Deployment:** GitLab CI (manual jobs)
- **Orchestration:** Kubernetes/EKS
- **Logs:** Grafana/Loki
- **APIs:** gRPC (identity services)
- **Documentation:** See "Alternative Platform" section in GITHUB-API-TESTING-STANDARDS.md

**Most testing guidance in these documents focuses on the Mailgun/Nomad platform.** The standards document includes a comprehensive section on Kubernetes/GitLab for teams working on Sinch Identity services.

---

## Quick Navigation

**I want to...**

- **Understand general testing approach** → GITHUB-API-TESTING-STANDARDS.md (not yet created)
- **Test a GitHub/Nomad assignment (gozer, jefe)** → HOW-TO-Identity-testing-staging-github.md (not yet created)
- **Test a GitLab/Kubernetes assignment (user-management-bff)** → HOW-TO-Identity-testing-staging-gitlab.md (not yet created)

---

## Testing Workflow Summary

### 1. Pre-Deployment
- [ ] PR merged and CI/CD passed
- [ ] Test plan prepared
- [ ] Test data identified
- [ ] VPN access confirmed

### 2. Deployment
```
# In Slack #eng-director-dev
@Director-Develop current version svc=gozer
@Director-Develop deploy svc=gozer tag=PR<number> env=mfstaging region=us-east4
```

### 3. Testing
```bash
# Wait 2-3 minutes for stabilization
source .env
./test_id88_staging_enhanced.py
```

### 4. Verification
- [ ] All tests passed
- [ ] Performance acceptable (< 500ms)
- [ ] Manual Auth0 check completed
- [ ] Logs reviewed (Kibana)

### 5. Sign-off or Rollback
- **Pass:** Document results, approve for production
- **Fail:** Rollback, document issues, create incident report

---

## Key Tools and Access

| Tool | Purpose | Access | Platform | Requires VPN |
|------|---------|--------|----------|--------------|
| **Pritunl VPN** | Access staging infrastructure | [Okta Portal](https://mailgun.okta.com/app/UserHome) | Both | N/A |
| **Slack** | Deployment commands | #eng-director-dev | Mailgun/Nomad | No |
| **GitLab CI** | Deployment pipelines | Project pipelines | Kubernetes | No |
| **Nomad UI** | Service health monitoring | http://nomad-servers.service.staging.mailforce:4646 | Mailgun/Nomad | Yes |
| **Kibana** | Log analysis | http://kibana-int-7x-001.us-east4.staging.mailforce.tech:5601 | Mailgun/Nomad | Yes |
| **Grafana** | Metrics and logs | https://grafana.int.prod.sinch.com | Kubernetes | No |
| **Auth0** | SAML configuration verification | https://manage.auth0.com/dashboard/eu/staging-mailgun | Both | No |
| **Sinch ID** | Domain verification UI | https://my.ninowire.com | Both | No |
| **DuckDNS** | Test domain creation | https://www.duckdns.org | Both | No |

---

## Test Scripts

### Original Script
**File:** `../../test_id88_staging.py`
- Basic smoke testing
- Interactive prompts
- Auth0 verification

### Enhanced Script (Recommended)
**File:** `../../test_id88_staging_enhanced.py`
- Response time tracking
- Performance metrics
- Environment variable support
- Edge case testing
- JSON results export

**Usage:**
```bash
# Interactive
./test_id88_staging_enhanced.py

# With .env file
source .env && ./test_id88_staging_enhanced.py

# Non-interactive (CI/CD)
./test_id88_staging_enhanced.py --non-interactive
```

---

## Document Maintenance

**Update these documents when:**
- Deployment process changes
- New testing tools introduced
- Environment access procedures update
- Security practices evolve
- New patterns emerge from testing

**Version Control:**
- Standards doc: v1.0 (2026-02-24)
- ID-88 guide: v1.0 (2026-02-24)
- Template source: Identity testing template v3.0

---

## Related Documentation

**In Project Root:**
- TEST_INSTRUCTIONS.md (not yet created) - Quick reference
- assignment.md (WIP session file, stored in $DOMAIN_ROOT/<repo>/sessions/<id>/) - Implementation details
- [verify.md](/verify.md) - Verification checklist
- [README.md](/README.md) - Gozer architecture

**External Resources:**
- [Setting up test SSO config](https://sinchenterprise.atlassian.net/wiki/spaces/MS/pages/1177387186/Setting+up+a+test+SSO+config)
- [Jira ID-88](https://sinchenterprise.atlassian.net/browse/ID-88)

---

---

## go.mod replace Directive for Cross-Service Local Testing

When developing against a local, unpublished version of a dependency (e.g. testing Gozer against a Jefe branch that has not yet been merged and versioned), use a `go.mod` replace directive to point to the local path.

**Lifecycle:**

| Phase | Directive state |
| --- | --- |
| Local development | Uncommented (active) |
| Before committing | Commented out |
| After upstream merges and version is cut | Remove the line entirely |

**Why this matters for test runs:**

- With the directive commented (committed state): `make test` and `go build` resolve against the published module version. If the local dependency added new methods, the build will fail — this is expected and correct.
- With the directive uncommented (local dev): builds and tests use the local path. Never commit in this state.

See also: `policies/git-operations.md` — go.mod Change Rules.

---

**Last Updated:** March 13, 2026

*Created: February 24, 2026*
