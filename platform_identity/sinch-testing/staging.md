# Sinch Testing — Staging Environment

**Last Updated:** March 12, 2026

> **Usage for LLM Agents:** Relevant when writing staging environment tests or curl examples targeting Sinch/Mailgun services (gozer, jefe). The Host header is required for all protected staging endpoints.

---

## Access to Staging Deployment Resources

How to get access to staging deployment resources for Sinch/mailgun services such as Gozer, Jefe.

### Security and Authentication

#### VPN (required)

- Launch and authenticate Printul VPN - Staging

**Result:** VPN tunnel for staging resources

#### Application Access (required)

- Browse to: https://mailgun.okta.com/app/UserHome
- Select access for Vault

**Result:** Launches vault

#### Auth0 Credentials for Local Development (Jefe / Gozer)

Retrieve the Auth0 M2M `client_id` and `client_secret` using one of two methods:

**Method 1 — Vault (preferred when available):**

```bash
export VAULT_ADDR=https://vault-001.us-east4.staging.mailforce.tech:8200
vault login -method=oidc
vault kv get secrets/auth0/staging-client
```

**Method 2 — Auth0 Dashboard (use when Vault is unavailable):**

1. Navigate and authenticate: [https://manage.auth0.com/dashboard/eu/staging-sinch/applications/](https://manage.auth0.com/dashboard/eu/staging-sinch/applications/)
2. Locate the application by name (e.g. **"Gozer Service Client"** — also used for Jefe)
3. Copy the **Client ID** and **Client Secret** from the application settings

**Place the retrieved values in each repo's `config/dev.yaml`** (copy from `config/dev_template.yaml` if it doesn't exist yet):

- **Jefe** → `sinch_management.client_id` and `sinch_management.client_secret`
- **Gozer** → `_SECRETS` section: SMA and OMA portal `client_id` and `client_secret`

> **Security:** `config/dev.yaml` is gitignored in both repos — never commit it. Never commit credentials to git under any circumstances.

### Logging Access

- Browse to: https://grafana.us-east4.staging.mailforce.tech/?orgId=1&from=now-6h&to=now&timezone=browser
- Select Default space
- Select Logs | Streams
- Search for term
- Highlight for term
- Set "show dates" date range

**Result:** Locating jefe logs for staging deployment

### Domain API Access

All the staging endpoints that are "protected" live under the same internal dns:
`http://vulcand-protected.service.staging.mailforce:9001`

Example: `http://vulcand-protected.service.staging.mailforce:9001/v3/portal/domains`

### Logging Endpoints

1. Deployment to Staging: http://nomad-servers.service.staging.mailforce:4646/ui/jobs
2. "Internal Logs"/unknown use case: http://kibana-int-7x-001.us-east4.staging.mailforce.tech:5601/
3. Staging Runtime logs: https://grafana.us-east4.staging.mailforce.tech/?orgId=1&from=now-6h&to=now&timezone=browser

---

## Host Header Requirement

**Date Added:** March 6, 2026
**Context:** Critical operational requirement for staging protected endpoints
**Impact:** All GitHub services (gozer, jefe) staging tests affected
**Status:** Documented and integrated into testing templates

### Table of Contents

| Section | Topic |
|---------|-------|
| [The Requirement](#the-requirement) | What must be included in every request (`Host: localhost`) |
| [Why This Is Required](#why-this-is-required) | Vulcand proxy routing architecture explanation |
| [Services Affected](#services-affected) | Gozer and Jefe endpoint lists |
| [Common Issues & Solutions](#common-issues--solutions) | 404, 502, and inconsistent routing fixes |
| [Testing Examples](#testing-examples) | Ready-to-use curl commands with correct headers |
| [Verification Checklist](#verification-checklist) | Pre-test checklist (VPN, headers, base URL) |

### Overview

All staging protected endpoint requests must include `Host: localhost` header for proper routing through the Vulcand proxy.

### The Requirement

All curl requests to staging protected endpoints MUST include:

```bash
--header 'Host: localhost'
```

**Correct Usage:**

```bash
curl --location \
  'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/organizations/user/<email>' \
  --header 'Host: localhost' \
  --header 'Content-Type: application/json'
```

**Incorrect Usage (Will Fail):**

```bash
# Missing Host header - will fail or route incorrectly
curl 'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/organizations/user/<email>'
```

### Why This Is Required

**Staging Protected Services:**
- **Base URL:** `http://vulcand-protected.service.staging.mailforce:9001`
- **Shared Entry Point:** All protected services (gozer, jefe, etc.) use the same URL
- **Routing Mechanism:** Vulcand proxy uses the Host header to route to the correct backend service
- **Access:** Requires Pritunl VPN connection

**Routing Logic:**

1. Request arrives at: `vulcand-protected.service.staging.mailforce:9001`
2. Vulcand proxy examines the `Host` header
3. Based on Host header, request is routed to appropriate backend service
4. Without Host header, Vulcand cannot determine routing → request fails

### Services Affected

**All protected endpoints require Host header:**

#### Gozer Endpoints

- Organizations: `/v1/auth/management/organizations/*`
- SAML: `/v2/portal/sma/*`
- Domains: `/v3/portal/domains`

#### Jefe Endpoints

- Manager: `/v1/auth/management/manager/*`
- MFA: `/v1/auth/management/mfa/*`
- Connections: `/v1/auth/management/connections/*`

#### Health Checks

- `/_ping` endpoints also require Host header

### Common Issues & Solutions

#### Issue: 404 Not Found

**Symptom:** Endpoint returns 404 even though deployment is confirmed
**Cause:** Missing Host header
**Solution:** Add `--header 'Host: localhost'` to request

#### Issue: 502 Bad Gateway

**Symptom:** Request returns 502 or connection error
**Cause:** Missing Host header or VPN not connected
**Solution:**

1. Verify VPN connection: `ifconfig | grep tun`
2. Add Host header to request

#### Issue: Inconsistent Routing

**Symptom:** Request sometimes works, sometimes fails
**Cause:** Missing Host header causing random routing
**Solution:** Always include Host header in all requests

### Testing Examples

#### Health Check

```bash
curl --header 'Host: localhost' \
  http://vulcand-protected.service.staging.mailforce:9001/_ping
```

#### GET Request (Organizations)

```bash
curl --location \
  'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/organizations/user/test@example.com' \
  --header 'Host: localhost' \
  --header 'Content-Type: application/json'
```

#### POST Request (MFA Enrollment)

```bash
curl --location --request POST \
  'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/mfa/enrollment-ticket/auth0|123456' \
  --header 'Host: localhost' \
  --header 'Content-Type: application/json'
```

#### POST with Data (Domain Creation)

```bash
curl --location \
  'http://vulcand-protected.service.staging.mailforce:9001/v3/portal/domains' \
  --header 'Host: localhost' \
  --header 'Content-Type: application/json' \
  --data '{
    "domain": "test.duckdns.org",
    "organization_id": "org_xxxxx"
  }'
```

#### For Test Scripts

```bash
# Define as constant
STAGING_BASE_URL="http://vulcand-protected.service.staging.mailforce:9001"
HOST_HEADER="Host: localhost"

# Use in requests
curl --location "${STAGING_BASE_URL}/v1/auth/management/..." \
  --header "${HOST_HEADER}" \
  --header 'Content-Type: application/json'
```

### Verification Checklist

When testing staging endpoints:

- [ ] VPN connected (Pritunl)
- [ ] Host header included in request
- [ ] Content-Type header included
- [ ] Base URL correct: `http://vulcand-protected.service.staging.mailforce:9001`
- [ ] Endpoint path correct
- [ ] Test with health check first

### Historical Context

This requirement was identified during ID-94 (MFA Enrollment Ticket endpoint) implementation when staging testing documentation was being created. The constraint was provided by operations team but had not been captured in formal documentation.

---

**Last Verified:** March 6, 2026
**Next Review:** June 6, 2026
**Maintained By:** Engineering Team
