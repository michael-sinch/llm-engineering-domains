# GitHub API Testing Standards

*Standardized post-deployment testing framework for Mailgun GitHub projects*

**Repos:** git@github.com:mailgun/gozer.git, git@github.com:mailgun/jefe.git  
**Last Updated:** February 24, 2026 03:20 PM EST

---

## Purpose

This document establishes standardized testing practices for GitHub-based API projects at Mailgun, with emphasis on:
- Post-deployment staging verification
- Slack-based deployment workflow integration
- REST and gRPC API testing approaches
- Manual and automated testing balance
- Observability and monitoring integration

**Primary Repositories:**
- `git@github.com:mailgun/gozer.git` - Identity/SAML management
- `git@github.com:mailgun/jefe.git` - Domain management
- Other Mailgun API services following similar patterns

---

## Table of Contents

1. [Testing Philosophy](#testing-philosophy)
2. [Deployment Workflow Integration](#deployment-workflow-integration)
3. [Service Discovery](#service-discovery)
4. [Environment Access](#environment-access)
5. [Test Data Management](#test-data-management)
6. [Testing Approaches](#testing-approaches)
7. [Verification Standards](#verification-standards)
8. [Monitoring and Observability](#monitoring-and-observability)
9. [Rollback Criteria](#rollback-criteria)
10. [Team Contacts & Escalation](#team-contacts--escalation)
11. [Security Practices](#security-practices)
12. [Test Automation](#test-automation)

---

## Testing Philosophy

### Core Principles

**1. API-First Testing**
- Prioritize backend API testing over UI testing
- Automated scripts preferred for repeatability
- UI testing as validation supplement, not primary method

**2. Post-Deployment Focus**
- Testing occurs after successful deployment to staging
- Verify actual deployed behavior, not pre-deployment code
- Integration with real staging dependencies

**3. Fast Feedback**
- Smoke tests complete within 5 minutes
- Critical path validation before comprehensive testing
- Clear pass/fail signals for quick decision-making

**4. Production-Readiness**
- Staging tests mirror production scenarios
- Performance baselines established
- Rollback procedures documented and tested

### Testing Scope Hierarchy

**Level 1: Smoke Tests (Required)**
- Critical functionality works
- No breaking changes to existing APIs
- Performance within acceptable range
- Typically 3-5 test cases, < 5 min execution

**Level 2: Functional Tests (Required)**
- Main feature scenarios (3+ cases)
- Edge cases and error handling (3+ cases)
- Backward compatibility verification
- Typically 10-15 test cases, < 15 min execution

**Level 3: Regression Tests (Optional)**
- Adjacent feature verification
- Cross-service integration checks
- Performance benchmarking
- Typically 10-20 test cases, < 30 min execution

**Level 4: Load/Performance Tests (As Needed)**
- Concurrent user scenarios
- Rate limiting verification
- Resource usage patterns
- Requires separate test environment or off-hours scheduling

---

## Deployment Workflow Integration

### Slack-Based Deployment (Mailgun Standard)

#### Deployment Channels

**Staging Deployments:**
- **Channel:** `#eng-director-dev`
- **Bot:** `@Director-Develop` (staging bot)
- **Access:** Engineering team members
- **Applicable Services:** gozer, jefe, other Mailgun GitHub services

**Production Deployments:**
- **Channel:** `#chatops`
- **Bot:** `@Director` (production bot)
- **Access:** Limited to release managers
- **Applicable Services:** gozer, jefe, other Mailgun GitHub services

#### Deployment Commands

**Check Current Version:**
```
@Director-Develop current version svc=<service-name>
```

**Example:**
```
@Director-Develop current version svc=gozer
```

**Response:**
```
Currently Deployed Version: mfstaging
us-east4: PR124@95ad33
```

**Deploy to Staging:**
```
@Director-Develop deploy svc=<service-name> tag=<PR-number> env=<environment> region=<region>
```

**Parameters:**
- `svc`: Service name (e.g., `gozer`, `jefe`)
- `tag`: PR number (e.g., `PR126`) or branch name
- `env`: Environment - always `mfstaging` for staging
- `region`: One of `us-east4`, `us-west1`, `europe-west1`

**Example:**
```
@Director-Develop deploy svc=gozer tag=PR126 env=mfstaging region=us-east4
```

**Multi-Region Staging Deployment:**
- Staging deployments can be fragmented/iterative
- Deploy to one region, test, then deploy to others
- All regions should eventually have consistent versions

**Example Iterative Deployment:**
```
# Step 1: Deploy to US first
@Director-Develop deploy svc=gozer tag=PR126 env=mfstaging region=us-east4

# Step 2: Test in us-east4, then deploy to other regions
@Director-Develop deploy svc=gozer tag=PR126 env=mfstaging region=us-west1
@Director-Develop deploy svc=gozer tag=PR126 env=mfstaging region=europe-west1
```

**Monitor Deployment:**
- Bot provides real-time updates in thread
- Nomad UI link provided for detailed monitoring
- Health checks reported (e.g., "5 healthy / 5 desired")
- Completion message confirms successful deployment

#### Pre-Deployment Checklist

Before deploying to staging:
- [ ] PR approved and merged
- [ ] PR number identified (e.g., PR126)
- [ ] CI/CD pipeline passed
- [ ] Test plan prepared
- [ ] Test data identified or prepared
- [ ] Monitoring/observability access confirmed

#### Post-Deployment Testing Window

**Standard Timeline:**
1. **T+0 min:** Deployment completes
2. **T+2 min:** Service health stabilizes
3. **T+2-10 min:** Execute smoke tests
4. **T+10-30 min:** Execute functional tests
5. **T+30-60 min:** Monitor for issues
6. **T+1 hour:** Sign-off or rollback decision

**Best Practice:** Begin testing 2-3 minutes after deployment completion to allow service stabilization.

#### Production Deployment Process

**Channel & Bot:**
- **Channel:** `#chatops`
- **Bot:** `@Director` (Note: Production uses `@Director`, not `@Director-Develop`)
- **Access:** Release managers only

**Production Deployment Command:**
```
@Director deploy svc=<service-name> tag=<PR-number> env=mfproduction region=<region>
```

**Parameters:**
- `env`: Always `mfproduction` for production
- `region`: One of `us-east4`, `us-west1`, `europe-west1`

**Critical:** Production deployments must be consistent across all regions as quickly as possible

**Production Deployment Strategy:**
```
# Deploy to all regions in rapid succession
@Director deploy svc=gozer tag=PR126 env=mfproduction region=us-east4
@Director deploy svc=gozer tag=PR126 env=mfproduction region=us-west1
@Director deploy svc=gozer tag=PR126 env=mfproduction region=europe-west1
```

**Pre-Production Deployment Checklist:**
- [ ] Staging testing fully completed in all regions
- [ ] All acceptance criteria met
- [ ] Performance validated (meets SLA requirements)
- [ ] Rollback plan documented and ready
- [ ] Release manager approval obtained
- [ ] Change notification sent (if required)
- [ ] Monitoring alerts configured
- [ ] On-call engineer notified

**Production Deployment Timeline:**
1. **T+0 min:** Deploy to first region (typically us-east4)
2. **T+2 min:** Monitor health, deploy to second region
3. **T+4 min:** Monitor health, deploy to third region
4. **T+10 min:** All regions deployed, monitor for issues
5. **T+30 min:** Initial stability confirmation
6. **T+2 hours:** Production sign-off or rollback decision

**Post-Production Monitoring:**
- Watch error rates in all regions
- Monitor response times against baseline
- Check for increased support tickets
- Review logs for unexpected errors
- Confirm no regression in adjacent features

**Production Sign-off Criteria:**
- [ ] All regions deployed successfully
- [ ] Health checks passing in all regions
- [ ] Error rates within normal range
- [ ] Response times within SLA
- [ ] No critical issues reported
- [ ] Monitoring shows expected behavior

**Rollback Authority:**
- **TODO:** Document who has authority to call rollback
- **TODO:** Define severity levels requiring immediate rollback
- **TODO:** Escalation path for production issues

---

## Service Discovery

### GitHub/Mailgun Services Registry

**Known Services:**
- **gozer** - Identity/SAML management service
- **jefe** - Domain management service
- **TODO:** Additional services to be documented

**Service Information Sources:**
- **TODO:** Central service registry location
- **TODO:** Service documentation index
- **TODO:** Architecture diagrams

### Finding Service Endpoints

**Staging URL Patterns:**
- **TODO:** Document URL patterns for services
- **TODO:** Service discovery mechanism
- **TODO:** DNS naming conventions

**Example Patterns:**
```
# Internal (via VPN)
http://<service-name>.service.staging.mailforce:PORT

# External (if applicable)
https://<service-name>-staging.mailgun.com
```

### Service-Specific Documentation

**Where to Find:**
- Repository README.md files
- **TODO:** Confluence space for service documentation
- **TODO:** API documentation location (OpenAPI/Swagger)
- **TODO:** Architectural Decision Records (ADRs)

---

## Environment Access

### VPN Requirements

#### Mailgun Staging VPN (Pritunl)

**Purpose:** Access to internal staging infrastructure
- Nomad UI monitoring
- Internal service endpoints
- Kibana log access
- Staging databases (read-only recommended)

**Setup:**
1. Navigate to [Okta Portal](https://mailgun.okta.com/app/UserHome)
2. Locate "Mailforce Staging - Pritunl" card
3. Download Pritunl client configuration
4. Import configuration to Pritunl client
5. Connect to VPN

**Verification:**
```bash
# Check VPN tunnel interface exists
ifconfig | grep tun

# Test access to Nomad UI
curl -I http://nomad-servers.service.staging.mailforce:4646/ui/
```

#### Sinch Identity VPN (If Applicable)

For projects spanning Sinch Identity infrastructure:
- Setup via Sinch IT
- Required for `dashboard.staging.sinch.com` access
- Auth0 management portal access

### Service Endpoints

#### Protected Endpoints (Gozer, Jefe)

**Staging Base URL:**
```
http://vulcand-protected.service.staging.mailforce:9001
```

**⚠️ CRITICAL REQUIREMENT: Host Header Override**

All staging protected endpoints require the `Host: localhost` header for proper routing:

```bash
curl --location 'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/<endpoint>' \
  --header 'Host: localhost' \
  --header 'Content-Type: application/json'
```

**Why Required:**
- Staging protected endpoints use a shared internal DNS
- The Host header ensures proper service routing in the Vulcand proxy
- Without it, requests may fail or route incorrectly

**Access Requirements:**
- Pritunl VPN connection required
- All services (gozer, jefe) share the same base URL

#### Gozer (Identity/SAML Service)

**Common Endpoints:**
- Organizations: `/v1/auth/management/organizations/user/<email>`
- SAML configuration: `/v2/portal/sma/<domain>/saml`
- Domains: `/v3/portal/domains`

**Example Request:**
```bash
curl --location 'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/organizations/user/test@example.com' \
  --header 'Host: localhost' \
  --header 'Content-Type: application/json'
```

#### Jefe (Domain Management Service)

**Common Endpoints:**
- Manager: `/v1/auth/management/manager/*`
- MFA: `/v1/auth/management/mfa/*`
- Organizations: `/v1/auth/management/organizations/*`
- Connections: `/v1/auth/management/connections/*`

**Example Request:**
```bash
curl --location --request POST \
  'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/mfa/enrollment-ticket/<USER_ID>' \
  --header 'Host: localhost' \
  --header 'Content-Type: application/json'
```

### Monitoring Dashboards

**Nomad UI:**
- URL: `http://nomad-servers.service.staging.mailforce:4646/ui/`
- Purpose: Service health, deployment status, resource usage
- Accessible via Pritunl VPN only

**Kibana (Logs):**
- URL: `http://kibana-int-7x-001.us-east4.staging.mailforce.tech:5601`
- Purpose: Application logs, error tracking, request tracing
- Credentials: Via Vault ([1Password Link](https://share.1password.com/s#Ofc6g-40quZgI16THzOmK8ciwalQQkGvUomvZhrBh30))
- Requires Pritunl VPN

**Grafana (Metrics):**
- Purpose: Performance metrics, response times, throughput
- Usage: Track individual API calls, identify bottlenecks
- Access details: Project-specific

---

## Test Data Management

### Test Domain Creation (DuckDNS)

**Purpose:** Free, temporary domains for testing domain verification, SAML SSO, and DNS-based features.

**When to Use:**
- SAML configuration testing
- Domain verification workflows
- Email routing tests
- Any feature requiring domain ownership proof

**Official Documentation:** [DuckDNS TXT Record Guide](https://pacbard.duckdns.org/articles/202310-txt-record-on-duckdns/)

#### Setup Process

**1. Create Account:**
- Navigate to [https://www.duckdns.org](https://www.duckdns.org)
- Login via GitHub SSO (Mailgun org) or Google
- Capture account token (displayed after login, next to duck logo)

**2. Create Test Domain:**
- Enter subdomain name (e.g., `mg-id88-test`)
- Click "Add domain"
- **IMPORTANT:** Do NOT click "Update Domain" with default IP
  - Default uses your current IP, which you likely don't want
  - Security: Avoid exposing internal processes in domain name

**3. Configure DNS Records:**

For domain verification (TXT record):
```bash
# Template
curl "https://www.duckdns.org/update?domains={SUBDOMAIN}&token={TOKEN}&txt={TXT_VALUE}"

# Example (use quotes for zsh/bash)
curl "https://www.duckdns.org/update?domains=mg-id88-test&token=2355e2ef-xxxx-xxxx-xxxx-xxxxxxxxxxxx&txt=sinch-domain-verification=0a6675cf-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

**Response:** `OK` (success) or error message

**4. Verify DNS Propagation:**
```bash
# Using dig (built-in)
dig mg-id88-test.duckdns.org TXT

# Look for ANSWER SECTION with your TXT record
# Propagation typically takes 1-5 minutes

# Alternative: Web-based checker
# https://dnschecker.org/
```

#### Alternative: UI-Based Workflow (Sinch ID Portal)

**When to Use:** Testing full SSO configuration with domain verification UI.

**Process:**

**1. Create DuckDNS Domain:**
- Same steps as above (create subdomain at duckdns.org)
- **Do NOT** click "update ips" with default values

**2. Register Domain in Sinch ID:**
- Navigate to [Sinch ID Portal](https://my.ninowire.com)
- Login with staging credentials
- Go to "SSO Configuration" → "Domains" → "Add new Domain"
- Enter your DuckDNS domain (e.g., `mg-id88-test.duckdns.org`)
- Copy the TXT record value shown (format: `sinch-domain-verification=<guid>`)

**3. Register TXT Record:**
```bash
# Use the TXT value from Sinch ID UI
curl "https://www.duckdns.org/update?domains=mg-id88-test&token=<your-token>&txt=sinch-domain-verification=<guid-from-sinch-id>"
```

**4. Verify in Sinch ID Portal:**
- Return to domain screen in Sinch ID
- Click "Check DNS" button (may need to refresh page)
- Wait for verification success (1-5 minutes)
- Once verified, click "Manage" → "Configure Enterprise SSO"
- Copy Callback URL and Audience for SAML configuration

**Note:** For ID-88 testing (SAML attribute mappings), full SSO configuration not required. Direct API testing is sufficient.

**5. Cleanup After Testing:**
- Login to [https://www.duckdns.org](https://www.duckdns.org)
- Locate domain in table at bottom
- Click "delete domain"

#### Test Domain Best Practices

- **Naming:** Use assignment ID in name (e.g., `mg-id88-<descriptor>`)
- **Documentation:** Record domain name in test data file
- **Cleanup:** Delete domains after testing (prevent accumulation)
- **Security:** Never use production-sounding names
- **Lifetime:** DuckDNS domains free for 30 days; renew if needed

### Test Data Coordination

**Avoiding Conflicts:**
- Use unique identifiers for parallel testing
- Timestamp-based naming recommended
- Developer-specific prefixes optional

**Naming Conventions:**
```
# Pattern: <project>-<assignment>-<developer>-<timestamp>
mg-id88-mikbig-20260224
mg-id88-test-142530

# Or use random suffixes
mg-id88-test-xa83k
```

**Shared vs. Isolated Test Data:**
- **Shared:** Use for integration testing across services
- **Isolated:** Default for individual developer testing
- **TODO:** Document shared test account credentials location
- **TODO:** Test data cleanup schedule/automation

**Resource Reservation:**
- **TODO:** Process for reserving shared test resources
- **TODO:** Communication channel for test coordination
- **TODO:** Maximum lifetime for test data

**Best Practices:**
- Always clean up test data after testing
- Never depend on existing test data (create fresh)
- Tag test data with creation date and owner
- Document any permanent test fixtures

### Test Data Documentation

For each assignment, create test data file:

**File:** `test-data-<assignment-id>.md` (in `.gitignore`)

**Template:**
```markdown
# Test Data for <Assignment ID>

## Created: YYYY-MM-DD
## Assignment: <Brief description>

### Test Domains
- Domain 1: `<subdomain>.duckdns.org`
  - Purpose: <e.g., SAML testing>
  - TXT record: <verification value>
  - Created: YYYY-MM-DD
  - Status: Active / Deleted

### Test Users (if applicable)
- User 1: `<email or ID>`
  - Purpose: <test scenario>
  - Associated data: <accounts, roles, etc.>

### Test Configurations
- Config 1: <Name/ID>
  - Purpose: <test case>
  - Details: <relevant info>

### Cleanup Status
- [ ] Domains deleted
- [ ] Test users removed (or marked as test)
- [ ] Test configs deleted
- Cleanup date: YYYY-MM-DD

### Notes
<Any additional context>
```

---

## Testing Approaches

### REST API Testing

**Primary Tools:**
- Python 3.9+ with `requests` library
- `curl` for quick manual tests
- Postman/Insomnia for exploration (not automated)

**Request Authentication:**
- Session cookies (from browser)
- Bearer tokens (Auth0 ID tokens)
- Service tokens (from Auth0 Management API)

**Standard Headers:**
```python
headers = {
    "Authorization": f"Bearer {id_token}",
    "Content-Type": "application/json",
    "Origin": "https://sma-portal-staging.mailgun.com"  # If required
}
```

**Example Test Function:**
```python
def test_endpoint(base_url: str, headers: dict) -> Tuple[bool, float]:
    """
    Test REST endpoint with timing
    Returns: (success, response_time_ms)
    """
    import time
    start = time.time()
    
    try:
        response = requests.get(f"{base_url}/v1/resource", 
                               headers=headers, 
                               timeout=10)
        elapsed_ms = (time.time() - start) * 1000
        
        if response.status_code == 200:
            print(f"✓ PASS ({elapsed_ms:.0f}ms)")
            return (True, elapsed_ms)
        else:
            print(f"✗ FAIL: {response.status_code}")
            return (False, elapsed_ms)
    except Exception as e:
        elapsed_ms = (time.time() - start) * 1000
        print(f"✗ FAIL: {e}")
        return (False, elapsed_ms)
```

### gRPC API Testing

**Primary Tools:**
- `grpcurl` - Command-line tool
- `grpcui` - Web UI for gRPC services
- Python with `grpcio` library

**Installation:**
```bash
# macOS
brew install grpcurl

# Or with Go
go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
```

**Basic Usage:**
```bash
# List services
grpcurl -plaintext <endpoint>:443 list

# List methods for service
grpcurl -plaintext <endpoint>:443 list package.ServiceName

# Call method
grpcurl -plaintext -d '{"field": "value"}' \
  <endpoint>:443 package.ServiceName/MethodName
```

**grpcui for Interactive Testing:**
```bash
# Launch web UI
grpcui <endpoint>:443

# Opens browser with interactive form for gRPC calls
```

### Manual UI Testing

**When to Use:**
- Visual regression verification
- User flow validation
- Post-API testing sanity check
- Features with complex UI interactions

**Staging Portals:**
- **SMA Portal:** `https://sma-portal-staging.mailgun.com`
- **Mailgun Dashboard:** Staging URL varies
- **Sinch Dashboard:** `https://dashboard.staging.sinch.com`
- **Sinch ID (Ninowire):** `https://my.ninowire.com`

**Confluence References:**
- [Setting up test SSO config](https://sinchenterprise.atlassian.net/wiki/spaces/MS/pages/1177387186/Setting+up+a+test+SSO+config)

**Best Practices:**
- Always start with API testing first
- UI testing supplements, doesn't replace automation
- Document UI flows with screenshots for complex cases
- Use test accounts, not personal accounts

---

## Verification Standards

### Test Case Requirements

**Minimum Test Coverage:**
- **Main functionality:** 3 test cases minimum
  - Success with typical data
  - Success with minimal data
  - Success with edge-valid data
- **Error handling:** 3 test cases minimum
  - Invalid input
  - Missing required parameters
  - Malformed requests
- **Pagination** (if applicable): 2 test cases
  - First page retrieval
  - Subsequent page retrieval
- **Performance:** Track all response times
- **Total Minimum:** 8-10 test cases for basic coverage

### Verification Matrix Template

| Test Case | Description | Expected Result | Actual Result | Response Time | Status | Notes |
|-----------|-------------|-----------------|---------------|---------------|--------|-------|
| TC-1 | Create with valid data | 200 OK, resource created | | | ⬜ / ✓ / ✗ | |
| TC-2 | Create with minimal data | 200 OK, defaults applied | | | ⬜ / ✓ / ✗ | |
| TC-3 | Create with maximum data | 200 OK, all fields set | | | ⬜ / ✓ / ✗ | |
| TC-4 | Create with invalid data | 400 Bad Request | | | ⬜ / ✓ / ✗ | |
| TC-5 | Create with missing required | 400 Bad Request | | | ⬜ / ✓ / ✗ | |
| TC-6 | Update existing resource | 200 OK, resource updated | | | ⬜ / ✓ / ✗ | |
| TC-7 | Get non-existent resource | 404 Not Found | | | ⬜ / ✓ / ✗ | |
| TC-8 | Performance baseline | < 500ms p95 | | | ⬜ / ✓ / ✗ | |

### Acceptance Criteria Checklist

Every assignment test should verify:
- [ ] Main feature functionality works as specified
- [ ] Error cases handled gracefully (no 500 errors)
- [ ] Backward compatibility maintained (no breaking changes)
- [ ] Performance within acceptable limits (< 500ms p95 for CRUD, < 2s for complex operations)
- [ ] Security requirements met (authentication, authorization)
- [ ] No regressions in adjacent features
- [ ] Logs contain appropriate information (no sensitive data)
- [ ] Documentation accurate

---

## Monitoring and Observability

### Log Analysis (Kibana)

**Access:**
- URL: `http://kibana-int-7x-001.us-east4.staging.mailforce.tech:5601`
- Requires: Pritunl VPN connection
- Credentials: Via Vault

**Use Cases:**
- Verify API call received and processed
- Check for errors or warnings during test execution
- Trace request flow through services
- Identify performance bottlenecks

**Basic Searches:**
```
# Find requests for specific domain
domain:"test-domain.com"

# Find errors in time range
level:error AND @timestamp:[now-1h TO now]

# Find specific API call
path:"/v2/portal/sma/*/saml" AND method:POST
```

### Service Health (Nomad)

**Access:**
- URL: `http://nomad-servers.service.staging.mailforce:4646/ui/`
- Requires: Pritunl VPN connection

**Use Cases:**
- Verify deployment completed successfully
- Check service instance health
- Identify crashing pods
- Monitor resource usage (CPU, memory)

**Key Indicators:**
- **Health Status:** All allocations healthy
- **Restarts:** Should be 0 after deployment stabilizes
---

## Team Contacts & Escalation

### Primary Contacts

**Platform & Infrastructure:**
- **Slack Channel:** **TODO**
- **On-Call:** **TODO**
- **Scope:** Nomad, Kibana, VPN, infrastructure issues

**Identity Team:**
- **Slack Channel:** **TODO**
- **Lead:** **TODO**
- **Scope:** Auth0, SAML, user management, gozer service

**Domain Management Team:**
- **Slack Channel:** **TODO**
- **Lead:** **TODO**
- **Scope:** Domain features, jefe service

**Release Management:**
- **Slack Channel:** `#chatops`
- **Contacts:** **TODO**
- **Scope:** Production deployment approvals, rollback decisions

### Support Channels

**General Questions:**
- **TODO:** Primary support channel
- **TODO:** Documentation/wiki location

**VPN/Access Issues:**
- **IT Support:** **TODO**
- **Response Time:** **TODO**

**Monitoring Tools:**
- **Nomad/Kibana Access:** **TODO** (Platform team)
- **Grafana Access:** **TODO**

**Auth0 Issues:**
- **Auth0 Admin:** **TODO** (Identity team)
- **Tenant Access:** **TODO**

### Escalation Path

**Level 1: Standard Issues** (Response: TODO)
- Use team-specific Slack channels
- Provide context, logs, reproduction steps
- Tag relevant team members if urgent

**Level 2: Blocking Issues** (Response: TODO)
- Escalate in team channel with @here or @channel
- Include impact assessment
- Request immediate assistance

**Level 3: Production Issues** (Response: TODO)
- Post in `#chatops` or **TODO** incident channel
- Page on-call engineer if critical
- Follow incident response process

**Rollback Decision Authority:**
- **Staging:** Engineering team consensus
- **Production:** **TODO** (Release manager? Engineering lead?)
- **Emergency:** On-call engineer has authority

### Response Time Expectations

- **Staging Issues:** **TODO**
- **Production Issues:** **TODO**
- **Security Issues:** **TODO**
- **Infrastructure Outage:** **TODO**

### Knowledge Resources

**Documentation:**
- **Confluence:** **TODO** (Space URL)
- **Wiki:** **TODO**
- **Runbooks:** **TODO**

**Training:**
- **Onboarding:** **TODO**
- **Service-Specific:** Check repository README files

---

- **Resource Usage:** Within normal ranges
- **Events:** No error events in recent history

### Performance Monitoring (Grafana)

**Use Cases:**
- Track response time trends
- Identify performance degradation
- Compare before/after deployment
- Establish performance baselines

**Best Practices:**
- Capture baseline metrics before testing
- Monitor during test execution
- Compare results to historical data
- Flag anomalies for investigation

---

## Rollback Criteria

### Decision Matrix

| Severity | Issue Type | Action | Timeline |
|----------|-----------|--------|----------|
| **Critical** | Service completely down | Rollback immediately | < 5 min |
| **Critical** | Data corruption | Rollback immediately | < 5 min |
| **Critical** | Security vulnerability | Rollback immediately | < 5 min |
| **High** | Main feature broken | Rollback | < 15 min |
| **High** | Performance > 5x baseline | Rollback | < 30 min |
| **Medium** | Edge case fails | Fix forward or rollback | < 1 hour |
| **Low** | Logging issues | Fix forward | Next release |
| **Low** | Minor UI glitch | Fix forward | Next release |

### Rollback Process

**1. Stop Testing:**
- Immediately halt ongoing tests
- Notify team in Slack (#eng-director-dev)
- Tag on-call engineer if critical

**2. Document Issue:**
- Screenshot error messages
- Copy full error logs from Kibana
- Record request/response that failed
- Note exact timestamp of failure
- Document scope (one feature or broader?)

**3. Execute Rollback:**

Via Slack:
```
@Director-Develop deploy svc=<service> tag=<previous-PR> env=mfstaging region=us-east4
```

**4. Verify Rollback:**
- Check deployment completes successfully
- Test basic functionality works
- Confirm issue no longer present
- Review logs for anomalies

**5. Post-Rollback Actions:**
- [ ] Create incident report
- [ ] Update GitLab/GitHub issue
- [ ] Root cause analysis
- [ ] Plan remediation
- [ ] Schedule re-test

---

## Security Practices

### Credential Management

**Rules:**
- ✅ Store tokens in `.env` files (must be in `.gitignore`)
- ✅ Never commit credentials to git
- ✅ Never display token values in logs, documentation, or chat
- ✅ Reference by variable name only ("`AUTH0_TOKEN` - Already configured")
- ✅ Use placeholders in examples (`<obtain-from-auth0-dashboard>`)
- ✅ Rotate tokens regularly (90 days minimum)
- ✅ Use read-only tokens when possible
- ✅ Store in password manager (1Password)

**Environment Variable Pattern:**

`.env` file (in `.gitignore`):
```bash
# Auth0 Configuration
AUTH0_CLIENT_ID=<from-auth0-dashboard>
AUTH0_CLIENT_SECRET=<from-auth0-dashboard>
AUTH0_DOMAIN=staging-mailgun.auth0.com

# Service Tokens
SERVICE_API_TOKEN=<from-service-dashboard>
```

Load in shell:
```bash
source .env

# Verify loaded (shows names only, not values)
env | grep AUTH0_ | cut -d= -f1
```

Use in Python:
```python
import os

auth0_client_id = os.getenv("AUTH0_CLIENT_ID")
if not auth0_client_id:
    raise ValueError("AUTH0_CLIENT_ID not set. Run: source .env")
```

### Test Data Privacy

**Rules:**
- Use obviously fake data (e.g., `test@example.com`, not real emails)
- Mark test accounts clearly (prefix with `test-` or similar)
- Don't test with production data
- Clean up test data after testing
- Document all test data created

---

## Test Automation

### Script Organization

**Recommended Structure:**
```
project-root/
├── .env                           # Credentials (gitignored)
├── test_<assignment>_<id>.py     # Main test script
├── TEST_INSTRUCTIONS.md          # Human-readable instructions
├── test-data-<id>.md             # Test data docs (gitignored)
└── AGENT-TEMPLATE/
    └── TESTING/
        ├── <ASSIGNMENT>-STAGING-TESTING.md  # Detailed guide
        └── GITHUB-API-TESTING-STANDARDS.md  # This file
```

### Python Script Best Practices

**Template Structure:**
```python
#!/usr/bin/env python3
"""
<Assignment ID> Staging Test
<Brief description>

Requirements:
    pip install requests  # or grpcio grpcio-tools
"""

import sys
import time
from typing import Tuple

# --- Configuration ---
ENDPOINT_BASE_URL = "https://staging.example.com"
EXPECTED_RESPONSE_TIME_MS = 500

# --- Test Functions ---
def test_main_scenario() -> Tuple[bool, float]:
    """Test main functionality"""
    start = time.time()
    # Test implementation
    elapsed_ms = (time.time() - start) * 1000
    return (success, elapsed_ms)

def test_edge_case() -> Tuple[bool, float]:
    """Test edge case"""
    # Similar structure
    pass

# --- Main Execution ---
def main():
    print("=" * 60)
    print(f"  <Assignment ID> Staging Test")
    print("=" * 60)
    
    results = []
    response_times = []
    
    # Run tests
    success, elapsed = test_main_scenario()
    results.append(success)
    response_times.append(elapsed)
    
    # Summary
    passed = sum(results)
    total = len(results)
    avg_time = sum(response_times) / len(response_times)
    
    print(f"\nResults: {passed}/{total} passed")
    print(f"Avg response time: {avg_time:.0f}ms")
    
    return 0 if all(results) else 1

if __name__ == "__main__":
    try:
        sys.exit(main())
    except KeyboardInterrupt:
        print("\n\nTest interrupted by user")
        sys.exit(130)
```

### Color-Coded Output

Use ANSI escape codes for better readability:
```python
class Colors:
    GREEN = '\033[92m'
    RED = '\033[91m'
    YELLOW = '\033[93m'
    BLUE = '\033[94m'
    RESET = '\033[0m'

def print_status(status: str, message: str):
    color = Colors.GREEN if status == "PASS" else Colors.RED
    print(f"{color}[{status}]{Colors.RESET} {message}")
```

### Response Time Tracking

Always measure and report:
```python
import time

start = time.time()
# API call
elapsed_ms = (time.time() - start) * 1000

# Report
if elapsed_ms < EXPECTED_RESPONSE_TIME_MS:
    print(f"✓ Performance: {elapsed_ms:.0f}ms (within {EXPECTED_RESPONSE_TIME_MS}ms target)")
else:
    print(f"⚠ Performance: {elapsed_ms:.0f}ms (exceeds {EXPECTED_RESPONSE_TIME_MS}ms target)")
```

---

## Appendix: Common Patterns

### Testing SAML Configurations

**Pattern:** Create → Verify → Update → Verify → Cleanup

**Key Checks:**
- Configuration created with correct defaults
- Attribute mappings applied correctly
- Updates preserve or modify as expected
- Auth0 connection reflects changes

### Testing Domain Features

**Pattern:** Create test domain (DuckDNS) → Configure → Verify → Test feature → Cleanup

**Key Checks:**
- DNS records propagate correctly
- Domain verification works
- Feature behavior correct for verified domains
- Cleanup removes test artifacts

### Testing with Auth0

**Pattern:** Get Management API token → Query connection/user → Verify data → Compare with API results

**Key Checks:**
- API response matches Auth0 data
- Auth0 configuration applied correctly
- No discrepancies between systems

**Auth0 Organizations (Advanced):**

For domain control features that use Auth0 Organizations:

**Common APIs:**
```bash
# Get user by email
GET /api/v2/users-by-email?email={email}

# Get user's organizations
GET /api/v2/users/{user_id}/organizations

# Add user to organization
POST /api/v2/organizations/{org_id}/members
Body: { "members": ["auth0|..."] }

# Remove user from organization
DELETE /api/v2/organizations/{org_id}/members
Body: { "members": ["auth0|..."] }
```

**Use Cases:**
- Domain grouping under organizations
- User access control for domain management
- Multi-domain administration
- Gozer interfaces with these APIs for domain control

**Reference:** [Working with Auth0 Organizations](https://sinchenterprise.atlassian.net/wiki/spaces/MS/pages/...) (Confluence)

---

## Alternative Platform: Kubernetes/GitLab (Sinch Identity)

**Note:** This section covers the alternative infrastructure used by Sinch Identity team projects. Most Mailgun projects (like gozer, jefe) use the Nomad/GitHub platform described earlier in this document.

### Platform Comparison

| Aspect | Mailgun (GitHub) | Sinch Identity (GitLab) |
|--------|------------------|-------------------------|
| **Primary Use** | Current document focus | Alternative platform |
| **Repos** | gozer, jefe | user-management-bff, internal-user-management-bff, sinch-identity-auth0-terraform |
| **Source Control** | GitHub | GitLab |
| **Orchestration** | Nomad | Kubernetes/EKS |
| **Deployment** | Slack (@Director-Develop) | GitLab CI (manual jobs) |
| **Logs** | Kibana | Grafana/Loki |
| **Secrets** | Vault, .env files | Sealed secrets (sinch CLI) |
| **APIs** | REST (SAML, domain mgmt) | gRPC (identity services) |
| **VPN** | Pritunl (Mailforce Staging) | AWS VPN for EKS access |

### Kubernetes/EKS Access Setup

**Prerequisites:**
- AWS CLI installed
- kubectl installed (`brew install kubectl` on macOS)
- Sinch VPN access

**Configuration:**

```bash
# 1. Configure AWS SSO
aws configure sso

# 2. Login via browser
# - Allow botocore-client-main
# - Select environment: ent-test
# - Select role: em-i-and-a-edit-staging
# - Set profile name (or use default)

# 3. Update kubectl contexts
aws eks update-kubeconfig --name eu1tst-eks001 --region eu-west-1 --profile <profile-name>
aws eks update-kubeconfig --name us1tst-eks001 --region us-east-1 --profile <profile-name>

# 4. Verify access (requires VPN)
kubectl get pods -n identity
kubectl get secrets -n identity
```

**EKS Clusters:**
- **Staging:** eu1tst-eks001 (eu-west-1), us1tst-eks001 (us-east-1)
- **Production:** eu1 (eu-west-1), us1 (us-east-1)

**Namespace:** `identity`

### GitLab CI Deployment

**Deployment Method:** Manual job trigger in GitLab CI pipeline

**Process:**
1. Navigate to project's GitLab CI pipeline page
2. Locate desired pipeline (commit/branch)
3. Trigger deployment job manually
4. Monitor pipeline progress

**Rollback:** Available via GitLab CI historical pipelines

**Deployment Window:** None - can deploy anytime

### Grafana/Loki Logging

**Access:**
- **URL:** https://grafana.int.prod.sinch.com/
- Single instance for staging + production
- Team folder: [Sinch Identity](https://grafana.int.prod.sinch.com/dashboards/f/aeoqsrnitaolca/sinch-identity)

**Log Source:** Loki - "Logs - beehive"

**Log Filtering:**
```
# Basic filter for service
{app="user-management-bff",container="user-management-bff",namespace="identity"}

# With error filter
{app="user-management-bff",container="user-management-bff",namespace="identity"} |~ "Error"

# Time-based
{app="...",namespace="identity"} |~ "pattern" [5m]
```

**Log Format:** JSON to stdout, ingested to Loki

### Sealed Secrets Management

**Tool:** Sinch CLI with sealed plugin

**Installation:**
```bash
# Install Sinch CLI (homebrew method)
brew install sinch-cli  # (or follow GitLab readme)

# Install sealed plugin
sinch plugin install sealed
```

**Sealing Secrets:**
```bash
# Navigate to secrets folder
cd path/to/secrets/

# Seal secret for environment
sinch sealed <secret-name> \
  --set password=<value> \
  --set username=<value> \
  --namespace identity \
  --env us1tst > sealed-secret.yaml
```

**Environments:** us1, eu1 (prod), us1tst, eu1tst (staging)

### gRPC Service Testing

**Tools:**
- `grpcurl` - Command-line testing
- `grpcui` - Web UI for interactive testing

**Common Services (staging):**
- Service Control: `servicecontrol.beehive-msi:9090` (us-east-1)
- Account Service: `accountsadmin.beehive-msi:9090` (us-east-1)
- IAM Builder: `key-service.identity-access:9090` (us-east-1)
- User Management BFF: `user-management-bff.us1tst.identity.staging.sinch.com:443`
- Internal User Management BFF: `internal-user-management-bff.us1tst.identity.staging.sinch.com:443`

**Example Testing:**
```bash
# Launch interactive UI
grpcui user-management-bff.us1tst.identity.staging.sinch.com:443

# Command-line testing
grpcurl -plaintext -d '{"field": "value"}' \
  endpoint:443 package.ServiceName/MethodName
```

### Observability

**Metrics:** Prometheus (scraped automatically)

**Tracing:** Istio + OpenTelemetry Collector
- Sampled per-request (subject to change based on load)
- Dashboards in Grafana (TODO - being developed)

**Alerting:** 
- Log level: Warning and above
- SLOs: Not yet defined
- Slack: #sinch-identity

### Key Differences Summary

**When to use Nomad/GitHub approach (gozer, jefe):**
- Mailgun-branded services
- REST APIs for SAML, domain management
- Slack-based deployment workflow
- Kibana for logs

**When to use Kubernetes/GitLab approach (user-management-bff):**
- Sinch Identity services
- gRPC APIs for user/account management
- GitLab CI deployment
- Grafana/Loki for logs
- Sealed secrets

---

## Document Maintenance

**Update Triggers:**
- New deployment workflows introduced
- Testing tools change
- Environment access procedures update
- Security practices evolve

**Version History:**
- v1.0 (2026-02-24): Initial version based on Identity template and Mailgun practices

---

---

## Testify Mock Defensive Patterns

### Nil-Safe Type Assertion (Mandatory)

Always use the comma-ok form for pointer return values in testify mock methods. The hard assertion panics when the mock returns `nil`, which is a standard test case for error paths.

```go
// Wrong — panics when mock returns nil
func (m *Client) GetUser(ctx context.Context, id string) (*User, error) {
    ret := m.Called(ctx, id)
    r0 := ret.Get(0).(*User)   // panics if nil
    return r0, ret.Error(1)
}

// Correct — nil-safe
func (m *Client) GetUser(ctx context.Context, id string) (*User, error) {
    ret := m.Called(ctx, id)
    r0, _ := ret.Get(0).(*User)
    return r0, ret.Error(1)
}
```

Apply this to every mock method that returns a pointer. No exceptions.

### Nil Return Coverage (Required)

Every mock method that returns a pointer must have at least one test case that configures the mock to return `nil` for that pointer. This validates both the nil-safe assertion and the calling code's nil handling.

```go
mockClient.On("GetUser", mock.Anything, "missing-id").
    Return(nil, errors.New("not found")).Once()
```

---

## Local Service Config Validation

Before running a service locally with `make run` or equivalent:

1. **Read the `Makefile` `run` target** — identify the exact `-config` path argument
2. **Create the config file at that path** — a file in the wrong location is silently ignored; the service starts but loads placeholder values
3. **Verify credentials are populated** — placeholder values cause all authenticated requests to fail with `500`, which is indistinguishable from a real server error without checking logs

**Common failure pattern:**

```makefile
# Makefile run target uses:  -config config/dev.yaml
# Developer created:          dev.yaml  ← silently ignored
# Result:                     500 on all authenticated requests
```

**Diagnostic:** If a locally running service returns `500` on every request, check the service logs for auth/credential errors before assuming a code bug.

---

**Last Updated:** March 13, 2026

*Last Updated: February 24, 2026*
