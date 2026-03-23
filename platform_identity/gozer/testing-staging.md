# HOW-TO: Identity Testing in Staging (Template)

---

## Document Metadata

**Version:** 3.0  
**Created:** February 2026  
**Last Updated:** March 16, 2026
**Status:** Template (Ready for Use)
**Location:** AGENT-TEMPLATE/ (protected by .gitignore)  
**Purpose:** Standardized post-deployment testing guide for Identity assignments

---

## Description of Purpose
This template provides a standardized, step-by-step guide for testing Identity-related features and assignments in the Sinch staging environment. It is designed to be copied and filled in for each specific assignment, ensuring consistency and completeness in the testing process.

**Testing Scope:** This template is for post-deployment external testing (automated and/or manual), not for unit or integration testing during development.

---

## Table of Contents
1. Scope
2. Resource Prerequisite Checklist
3. Environment Configuration
4. Test Data Setup
5. Prioritized Step-by-Step Process
    - Entry Prerequisite
    - Steps
    - Exit Conditions
6. Verification Matrix
7. Performance Expectations
8. Test Automation
9. Troubleshooting
10. Rollback Procedures

---

## Scope
    - `sinch-identity-auth0-terraform` (GitHub)
    - `user-management-bff` (GitLab)
    - <FILL IN: Additional repos for assignment>
**Teams:** Identity
**Services:** Sinch Identity, Mailgun Identity, Auth0
**Primary Repos Using These Instructions:**
        - `git@github.com:mailgun/gozer.git` (Gozer)
        - `git@github.com:mailgun/jefe.git` (Jefe)
        - `git@github.com:mailgun/sinch-identity-auth0-terraform.git` (Sinch Identity Auth0 Terraform)
        - `gitlab.com/sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/user-management-bff` (User Management BFF)
        - <FILL IN: Additional repos for assignment>
**Environments:** Staging (`my.ninowire.com`, `dashboard.staging.sinch.com`)
**Use Case:** Testing new features, bug fixes, or assignments related to identity management, authentication, and user flows.

---

## Resource Prerequisite Checklist
- [ ] Staging VPN access (Mailgun and/or Sinch VPN)
- [ ] Staging user credentials (or ability to create new user)
- [ ] Access to Auth0 management portal (staging)
- [ ] Access to required GitHub/GitLab repos
- [ ] Docker or Podman installed (for local testing)
- [ ] grpcui installed (for API testing)
- [ ] Authentication tokens configured (see Environment Configuration below)
- [ ] <FILL IN: Any assignment-specific resources>

---

## Environment Configuration

### Proto Field Naming Verification

⚠️ **CRITICAL:** Before filling in request/response examples, verify exact field names from proto files.

**Steps:**
1. Locate the proto definition:
   ```bash
   # Search for the endpoint definition
   grep -r "rpc <EndpointName>" proto/
   ```
2. Open the proto file and note exact field names (case-sensitive)
3. Use proto field names exactly as defined (e.g., `userId` not `user_id`, `pageSize` not `page_size`)
4. Document proto file location and line number for reference

**Example:**
```markdown
**Proto Reference:** `proto/api/user_service.proto` line 290
- Field names: `userId` (camelCase), `pageSize`, `pageToken`
- Service: `internaluserpb.InternalUserService`
- Method: `SearchUserRolesAcrossAccounts`
```

⚠️ **Security Note:** See [guidance.md](/guidance.md) for credential handling rules.

### Authentication Setup

**Required Environment Variables:**

⚠️ **Security Note:** Reference the `.env` file location only. Never display actual token values in documentation.

Check if `.env` file exists in project root (DO NOT COMMIT - must be in .gitignore).

**If `.env` exists:**
- Document which variables are already configured (list names only)
- List which variables need to be added
- Never display actual token values

**If `.env` does not exist:**
Create `.env` file with the following structure:

```bash
# Auth0 Configuration
AUTH0_CLIENT_ID=<from-auth0-dashboard>
AUTH0_CLIENT_SECRET=<from-auth0-dashboard>
AUTH0_AUDIENCE=staging-sinch.eu.auth0.com
AUTH0_DOMAIN=staging-sinch.eu.auth0.com

# MSI/IAM Configuration
MSI_BEARER_TOKEN_OVERRIDE=<token-for-roles-api>

# <FILL IN: Assignment-specific tokens>
```

**Token Acquisition:**
1. **Auth0 Credentials:** Available from [Auth0 Dashboard](https://manage.auth0.com/dashboard/eu/staging-sinch)
2. **MSI Bearer Token:** Perform token exchange with Auth0 to get roles API token
3. **<FILL IN: Assignment-specific token instructions>**

**Load Environment Variables:**
```bash
# From project root directory
source .env

# Verify variables are loaded (shows variable names, not values)
env | grep -E "AUTH0_|MSI_|GITLAB_" | cut -d= -f1
```

**Expected Output:**
```
AUTH0_CLIENT_ID
AUTH0_CLIENT_SECRET
AUTH0_AUDIENCE
AUTH0_DOMAIN
MSI_BEARER_TOKEN_OVERRIDE
<FILL IN: Additional expected variables>
```

**Security Notes:**
- ✅ Never commit `.env` files (verify in .gitignore)
- ✅ Never display actual token values in documentation or chat
- ✅ Reference variables by name only: "`GITLAB_TOKEN` - Already configured"
- ✅ Use placeholders in examples: `<obtain-from-dashboard>`
- Rotate tokens regularly (90 days recommended)
- Use read-only tokens when possible
- Keep tokens in password manager
- See [guidance.md](/guidance.md) for security best practices

---

## Test Data Setup

### Required Test Data

**Document the test data needed for this assignment:**

- [ ] **Test Users:**
  - User ID(s): <FILL IN>
  - Email(s): <FILL IN>
  - Required state: <FILL IN: e.g., "user with multiple account associations">

- [ ] **Test Accounts:**
  - Account ID(s): <FILL IN>
  - Account type: <FILL IN>
  - Required configuration: <FILL IN>

- [ ] **Test Roles:**
  - Role ID(s): <FILL IN>
  - Role names: <FILL IN>
  - Permissions required: <FILL IN>

- [ ] **Other Resources:**
  - <FILL IN: Projects, domains, organizations, etc.>

### Test Data Creation Steps

1. **<FILL IN: Step 1>**
   ```bash
   # Example command or grpcui instruction
   <FILL IN>
   ```

2. **<FILL IN: Step 2>**
   ```bash
   <FILL IN>
   ```

3. **Verification:**
   - <FILL IN: How to verify test data is created correctly>

4. **Document Test Data:**
   Create `test-data-<assignment-id>.md` in AGENT-TEMPLATE/ directory:
   ```markdown
   # Test Data for <FILL IN: Assignment ID>
   
   ## Test Users
   - User 1: `auth0|______________` (email: ______________)
     - <FILL IN: Associated data>
   
   ## Test Accounts
   - Account 1: `______________`
   
   ## Test Roles
   - Role 1: `______________` (name: ______________)
   
   ## Created: YYYY-MM-DD
   ## Last Verified: YYYY-MM-DD
   ```
   
   **Note:** This file won't be committed (directory is in .gitignore)

**Test Data Cleanup:**
- <FILL IN: Steps to clean up test data after testing>
- Update `test-data-<assignment-id>.md` with cleanup status

---

## Prioritized Step-by-Step Process

### Entry Prerequisite
- Ensure all items in the Resource Prerequisite Checklist are available and working.
- Confirm assignment details and required test cases.

### Steps
1. **Connect to Staging VPN**
    - Use VPN client to connect to `my.ninowire.com` or other required endpoints.
    - Mailgun VPN for internal endpoints/logs; Sinch VPN for infrastructure tools.
2. **Prepare Test User**
    - Use existing staging credentials or sign up at [dashboard.staging.sinch.com](https://dashboard.staging.sinch.com/).
    - If user does not appear, check Auth0: [Auth0 Staging Users](https://manage.auth0.com/dashboard/eu/staging-sinch/users).
    - Create user if needed.
3. **Login Test**
    - Attempt login at `my.ninowire.com` with staging user.
    - Confirm access and expected user flows.
4. **Assignment-Specific Testing**
    - <FILL IN: Steps for assignment>
    - Example: Verify domain functionality, API endpoint, etc.
    - Use grpcui for API testing (GitLab services):
      ```bash
      grpcui user-management-bff.us1tst.identity.staging.sinch.com:443
      grpcui internal-user-management-bff.us1tst.identity.staging.sinch.com:443
      ```
    - Use curl for endpoint testing (GitHub services - gozer/jefe):
      
      **⚠️ CRITICAL:** Always include `--header 'Host: localhost'` for staging protected endpoints
      
      ```bash
      # Example: Organizations endpoint
      curl --location \
        'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/organizations/user/<email>' \
        --header 'Host: localhost' \
        --header 'Content-Type: application/json'
      
      # Example: Domain creation (Gozer)
      curl --location \
        'http://vulcand-protected.service.staging.mailforce:9001/v3/portal/domains' \
        --header 'Host: localhost' \
        --header 'Content-Type: application/json' \
        --data '{
          "domain":"test.duckdns.org",
          "organization_id":"org_xxxxx"
        }'
      
      # Example: MFA endpoint (Jefe)
      curl --location --request POST \
        'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/mfa/enrollment-ticket/<USER_ID>' \
        --header 'Host: localhost' \
        --header 'Content-Type: application/json'
      ```
      
      **Why Host header is required:**
      - All staging protected services (gozer, jefe) share: `http://vulcand-protected.service.staging.mailforce:9001`
      - The Host header ensures proper routing through the Vulcand proxy
      - Without it, requests will fail or route incorrectly
      
5. **Confirm Results**
    - Check Auth0 logs for user activity and success/failure exchanges.
    - Confirm described defect or verify assignment functionality.
    - Document confirmation details (to be iteratively added).

### Exit Conditions
- All required test cases executed and results documented.
- Assignment-specific acceptance criteria met.
- No critical issues blocking deployment or release.

---

## Verification Matrix

### Test Case Guidance

**Minimum Test Cases Required:**
- **Main functionality:** 3 test cases (typical scenarios: success with data, minimal data, empty/no data)
- **Edge cases:** 3 test cases (invalid input, missing parameters, empty strings)
- **Pagination:** 2 test cases (first page, subsequent pages) - if applicable
- **Cross-verification:** 1+ test cases per related endpoint - if applicable
- **Performance:** Track response time in all test cases
- **Total minimum:** 10-13 test cases for comprehensive coverage

### Test Case Results

| Test Case | Description | Expected Result | Actual Result | Response Time | Status | Notes |
|-----------|-------------|-----------------|---------------|---------------|--------|-------|
| TC-1 | <FILL IN: Test case name> | <FILL IN> | <FILL IN> | <FILL IN: e.g., 345ms> | ⬜ Pass / ❌ Fail | |
| TC-2 | <FILL IN: Test case name> | <FILL IN> | <FILL IN> | <FILL IN> | ⬜ Pass / ❌ Fail | |
| TC-3 | <FILL IN: Edge case> | <FILL IN> | <FILL IN> | <FILL IN> | ⬜ Pass / ❌ Fail | |
| TC-4 | <FILL IN: Error handling> | <FILL IN> | <FILL IN> | <FILL IN> | ⬜ Pass / ❌ Fail | |

### Acceptance Criteria Checklist

- [ ] <FILL IN: Acceptance criterion 1>
- [ ] <FILL IN: Acceptance criterion 2>
- [ ] <FILL IN: Acceptance criterion 3>
- [ ] <FILL IN: Performance requirements met>
- [ ] <FILL IN: Security requirements met>
- [ ] No regressions in existing functionality

---

## Performance Expectations

**Response Time Targets:**
- Endpoint: `<FILL IN: endpoint name>`
- Expected latency: <FILL IN: e.g., "< 500ms p95">
- Timeout threshold: <FILL IN: e.g., "5s">

**Load Expectations:**
- Concurrent requests: <FILL IN>
- Throughput: <FILL IN: e.g., "100 req/sec">

**Resource Usage:**
- Memory: <FILL IN>
- CPU: <FILL IN>

**During Testing:**
- Monitor response times for all test cases
- Document any performance anomalies
- Compare with baseline (existing similar endpoints)

---

## Test Automation

### Automation Strategy

**Primary Language:** Python (for consistency and advanced features)  
**Alternative:** Bash (for simple, single-operation tasks only)

**Automation Goals:**
- Enable quick regression testing
- Automate repetitive test cases
- Track performance metrics
- Provide clear pass/fail reporting

### Python Testing Script (Recommended)

Create `test_<assignment>_<assignment_id>.py`:

```python
#!/usr/bin/env python3
"""
Automated testing for <FILL IN: assignment description>
<FILL IN: Assignment ID>

Requirements:
    pip install grpcio grpcio-tools
    OR use grpcurl via subprocess
"""

import sys
import json
import subprocess
from datetime import datetime
from typing import List, Dict, Tuple

# Configuration
ENDPOINT = "<FILL IN: grpc-endpoint:443>"
SERVICE = "<FILL IN: package.ServiceName>"
METHOD = "<FILL IN: MethodName>"

# Test data (update after test data creation)
TEST_DATA = {
    "scenario_1": "<FILL IN: test data identifier>",
    "scenario_2": "<FILL IN: test data identifier>",
    "invalid": "<FILL IN: invalid data for edge case>",
}

# ANSI color codes for output
class Colors:
    GREEN = '\033[0;32m'
    RED = '\033[0;31m'
    YELLOW = '\033[1;33m'
    BLUE = '\033[0;34m'
    NC = '\033[0m'  # No Color

def call_grpc(request_data: dict) -> Tuple[bool, dict, str, float]:
    """
    Call gRPC endpoint using grpcurl
    Returns: (success, response_dict, error_message, response_time_ms)
    """
    start_time = datetime.now()
    
    cmd = [
        "grpcurl",
        "-plaintext",
        "-d", json.dumps(request_data),
        ENDPOINT,
        f"{SERVICE}/{METHOD}"
    ]
    
    try:
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=10
        )
        
        end_time = datetime.now()
        response_time_ms = (end_time - start_time).total_seconds() * 1000
        
        if result.returncode == 0:
            response = json.loads(result.stdout) if result.stdout else {}
            return (True, response, "", response_time_ms)
        else:
            return (False, {}, result.stderr, response_time_ms)
    except subprocess.TimeoutExpired:
        return (False, {}, "Request timed out after 10s", 10000)
    except Exception as e:
        return (False, {}, str(e), 0)

def test_main_scenario_1() -> Tuple[bool, str, float]:
    """<FILL IN: Test description>"""
    test_name = "TC-1: <FILL IN: test case name>"
    print(f"{Colors.BLUE}{test_name}{Colors.NC}... ", end="", flush=True)
    
    success, response, error, response_time = call_grpc({
        "<FILL IN: field>": TEST_DATA["scenario_1"]
    })
    
    if not success:
        result = f"{Colors.RED}✗ FAILED{Colors.NC}: {error}"
        print(result)
        return (False, result, response_time)
    
    # <FILL IN: Validation logic>
    expected_count = 3  # Example
    actual_count = len(response.get("items", []))
    
    if actual_count == expected_count:
        result = f"{Colors.GREEN}✓ PASSED{Colors.NC} ({actual_count} items, {response_time:.0f}ms)"
        print(result)
        return (True, result, response_time)
    else:
        result = f"{Colors.RED}✗ FAILED{Colors.NC} (expected {expected_count}, got {actual_count})"
        print(result)
        return (False, result, response_time)

def test_edge_case_invalid() -> Tuple[bool, str, float]:
    """<FILL IN: Edge case description>"""
    test_name = "TC-X: <FILL IN: edge case name>"
    print(f"{Colors.BLUE}{test_name}{Colors.NC}... ", end="", flush=True)
    
    success, response, error, response_time = call_grpc({
        "<FILL IN: field>": TEST_DATA["invalid"]
    })
    
    # Should fail with error or return empty
    if not success or response.get("items", []) == []:
        result = f"{Colors.GREEN}✓ PASSED{Colors.NC} (handled correctly, {response_time:.0f}ms)"
        print(result)
        return (True, result, response_time)
    else:
        result = f"{Colors.RED}✗ FAILED{Colors.NC} (should return error or empty)"
        print(result)
        return (False, result, response_time)

def run_all_tests() -> None:
    """Run all test cases and report results"""
    print("=" * 70)
    print(f"  Testing <FILL IN: Assignment Name> (<FILL IN: ID>)")
    print(f"  Endpoint: {ENDPOINT}")
    print(f"  Date: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    print("=" * 70)
    print()
    
    # Check if grpcurl is available
    try:
        subprocess.run(["grpcurl", "--version"], capture_output=True, check=True)
    except (subprocess.CalledProcessError, FileNotFoundError):
        print(f"{Colors.RED}✗ ERROR: grpcurl is not installed{Colors.NC}")
        print("Install with: brew install grpcurl")
        sys.exit(1)
    
    # Check if test data is configured
    if any("<FILL IN" in str(v) for v in TEST_DATA.values()):
        print(f"{Colors.YELLOW}⚠ WARNING: Test data not fully configured!{Colors.NC}")
        print("Update TEST_DATA dict with actual values from test-data-<id>.md")
        print()
    
    print(f"{Colors.YELLOW}Running main test cases...{Colors.NC}")
    print()
    
    main_tests = [
        test_main_scenario_1,
        # <FILL IN: Add more main test functions>
    ]
    
    results = []
    response_times = []
    
    for test_func in main_tests:
        try:
            passed, message, response_time = test_func()
            results.append(passed)
            if response_time > 0:
                response_times.append(response_time)
        except Exception as e:
            print(f"{Colors.RED}✗ FAILED{Colors.NC} with exception: {e}")
            results.append(False)
    
    print()
    print(f"{Colors.YELLOW}Running edge case tests...{Colors.NC}")
    print()
    
    edge_tests = [
        test_edge_case_invalid,
        # <FILL IN: Add more edge case test functions>
    ]
    
    for test_func in edge_tests:
        try:
            passed, message, response_time = test_func()
            results.append(passed)
            if response_time > 0:
                response_times.append(response_time)
        except Exception as e:
            print(f"{Colors.RED}✗ FAILED{Colors.NC} with exception: {e}")
            results.append(False)
    
    print()
    print("=" * 70)
    print("  Test Results Summary")
    print("=" * 70)
    print(f"  Passed: {Colors.GREEN}{sum(results)}{Colors.NC}/{len(results)}")
    print(f"  Failed: {Colors.RED}{len(results) - sum(results)}{Colors.NC}/{len(results)}")
    
    if response_times:
        print()
        print("  Performance Metrics:")
        print(f"    Average response time: {sum(response_times)/len(response_times):.0f}ms")
        print(f"    Min response time: {min(response_times):.0f}ms")
        print(f"    Max response time: {max(response_times):.0f}ms")
    
    print("=" * 70)
    print()
    
    if all(results):
        print(f"{Colors.GREEN}✓ All tests passed!{Colors.NC}")
        sys.exit(0)
    else:
        print(f"{Colors.RED}✗ Some tests failed!{Colors.NC}")
        print()
        print("Troubleshooting steps:")
        print("1. Verify test data is configured correctly in TEST_DATA")
        print("2. Check test-data-<id>.md for correct identifiers")
        print("3. Ensure environment variables are loaded (source .env)")
        print("4. Verify VPN connection is active")
        print("5. Confirm endpoint is deployed and accessible")
        sys.exit(1)

if __name__ == "__main__":
    run_all_tests()
```

**Setup and Run:**

```bash
# Make executable
chmod +x test_<assignment>_<id>.py

# Update TEST_DATA dictionary after test data creation

# Run tests
source .env  # Load environment variables
python3 test_<assignment>_<id>.py
```

### Bash Script (Simple Tasks Only)

For very simple scenarios, create `test-<assignment>.sh`:

```bash
#!/bin/bash
# Simple test script for <FILL IN: assignment>

set -e

ENDPOINT="<FILL IN>"
GREEN='\033[0;32m'
RED='\033[0;31m'
NC='\033[0m'

echo "Test 1: <FILL IN>"
grpcurl -plaintext -d '{"field": "value"}' ${ENDPOINT} package.Service/Method
echo "${GREEN}✓ Test 1 passed${NC}"

echo "${GREEN}All tests passed!${NC}"
```

### CI/CD Integration (Optional)

Add to `.gitlab-ci.yml` for automated testing:

```yaml
test:<assignment>:staging:
  stage: test
  image: python:3.11
  before_script:
    - apt-get update && apt-get install -y grpcurl
    - pip install grpcio grpcio-tools
  script:
    - source .env
    - python3 test_<assignment>_<id>.py
  only:
    - merge_requests
  when: manual
  environment:
    name: staging
  allow_failure: true
  artifacts:
    when: always
    reports:
      junit: test-results.xml
```

### Adjacent Feature Testing (Optional)

Test related endpoints to ensure no regressions:

```bash
#!/bin/bash
# Test adjacent features

echo "Testing adjacent features for regressions..."
echo ""

# <FILL IN: Commands to test related endpoints>
# Example:
# - Test GetUserByID still works
# - Test SearchUsers still works
# - Test authentication flows

echo ""
echo "Adjacent feature testing complete"
```

---

## Troubleshooting

### Troubleshooting Structure

For each issue, document:
- **Issue:** Clear symptom description
- **Solution:** Step-by-step resolution with commands and verification

### Connection Issues

**Issue:** Cannot connect to staging endpoints
**Solution:**
- Check VPN connection is active:
  ```bash
  # Verify VPN status
  ifconfig | grep tun  # Look for tunnel interface
  ```
- Verify endpoint URL and port
- Try DNS resolution:
  ```bash
  nslookup <endpoint-hostname>
  ```
- Test with known working endpoint first

### API Request Failures (GitHub Services: Gozer/Jefe)

**Issue:** API requests to staging protected endpoints fail with 404, 502, or routing errors
**Solution:**
- **⚠️ MOST COMMON:** Missing `Host: localhost` header
  ```bash
  # ❌ INCORRECT - Will fail or route incorrectly
  curl 'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/...'
  
  # ✅ CORRECT - Include Host header
  curl 'http://vulcand-protected.service.staging.mailforce:9001/v1/auth/management/...' \
    --header 'Host: localhost'
  ```
- Verify VPN connection is active (required for protected endpoints)
- Check service is deployed and healthy in Nomad UI
- Verify endpoint path is correct (check API documentation)
- Test with health check endpoint first:
  ```bash
  curl --header 'Host: localhost' \
    http://vulcand-protected.service.staging.mailforce:9001/_ping
  ```

**Why Host header is required:**
- All staging protected services share the same internal DNS
- The Vulcand proxy uses the Host header for service routing
- Without it, requests cannot be routed to the correct service

### Authentication/Authorization Errors

**Issue:** Permission denied or authentication errors
**Solution:**
- Verify environment variables are loaded:
  ```bash
  env | grep -E "AUTH0_|MSI_" | cut -d= -f1
  ```
- Check token expiration (Auth0 tokens typically expire after 24h)
- Verify credentials in Auth0 dashboard
- Re-obtain tokens if expired:
  ```bash
  # <FILL IN: Token refresh command>
  ```

### User Not Found

**Issue:** User not found in Auth0 or system
**Solution:**
- Check Auth0 dashboard: [Auth0 Staging Users](https://manage.auth0.com/dashboard/eu/staging-sinch/users)
- Verify domain (staging.sinch.com vs sinch.com)
- Search by email if user ID unknown
- Check user ID format (e.g., `auth0|...`)
- Create user manually if needed

### Data Not Found / Empty Response

**Issue:** Expected data not returned or empty response
**Solution:**
- Verify test data exists:
  ```bash
  # <FILL IN: Command to verify data>
  ```
- Check test-data-<id>.md for correct identifiers
- Cross-verify with related endpoints
- Check application logs for upstream errors
- Verify request parameters match proto field names

### Configuration Errors

**Issue:** Service configuration error or feature not available
**Solution:**
- Verify deployment is up-to-date:
  ```bash
  # <FILL IN: Command to check deployment version>
  ```
- Check feature flag is enabled (if applicable)
- Verify service dependencies are running
- Check application logs for initialization errors

### Installation Problems

**Issue:** grpcurl or other tools not found
**Solution:**
- Install grpcurl:
  ```bash
  # macOS
  brew install grpcurl
  
  # Or with Go
  go install github.com/fullstorydev/grpcurl/cmd/grpcurl@latest
  
  # Verify
  grpcurl --version
  ```
- Install Python dependencies:
  ```bash
  pip install grpcio grpcio-tools
  ```

### Environment Variable Issues

**Issue:** Variables not loading or showing as empty
**Solution:**
- Verify .env file location:
  ```bash
  ls -la .env
  ```
- Check .env format (no spaces around `=`):
  ```bash
  cat .env
  ```
- Source explicitly:
  ```bash
  source ./.env
  ```
- Export manually if needed:
  ```bash
  export AUTH0_CLIENT_ID="<value>"
  ```

### Performance Issues

**Issue:** Response times consistently slow (> expected threshold)
**Solution:**
- Check network latency:
  ```bash
  ping <endpoint-hostname>
  ```
- Verify VPN performance
- Test at different times (avoid peak hours)
- Check upstream service health
- Review application logs for slow queries
- Compare with baseline from similar endpoints

### Assignment-Specific Issues

**<FILL IN: Issue Category>**
**Issue:** <FILL IN: Symptom description>
**Solution:**
- <FILL IN: Step 1>
- <FILL IN: Step 2>
- <FILL IN: Verification>

---

## Rollback Procedures

**Note:** Rollback procedures are for post-deployment testing. This section is optional but recommended for production-readiness verification.

### When to Rollback

- Critical bugs discovered during testing
- Performance degradation beyond acceptable limits
- Security vulnerabilities identified
- Data integrity issues
- Deployment failures

### Rollback Decision Matrix (Optional)

Consider creating a decision matrix for your specific assignment:

| Issue Type | Severity | Rollback? | Alternative | Response Time |
|------------|----------|-----------|-------------|---------------|
| Data corruption | Critical | ✅ Yes immediately | None | < 5 min |
| Security vulnerability | Critical | ✅ Yes immediately | None | < 5 min |
| Service completely down | Critical | ✅ Yes immediately | Restart pods first | < 10 min |
| Performance > 5s | High | ✅ Yes | Investigate first | < 30 min |
| Wrong data returned | High | ✅ Yes | None | < 15 min |
| One edge case fails | Low | ❌ No | Document workaround | Monitor |
| Logging issues | Low | ❌ No | Fix forward | Next release |

### Rollback Steps

**1. Immediate Actions:**
- [ ] Stop ongoing tests
- [ ] Document the issue:
  - Screenshot error messages
  - Copy full error logs
  - Record request/response that caused issue
  - Note timestamp of failure
- [ ] Notify team via Slack/Email:
  - Identity team channel
  - Tag: @on-call-engineer

**2. Verify Issue Severity:**
- [ ] Does it affect other endpoints?
- [ ] Is it data corruption or just this feature?
- [ ] Can workaround be used temporarily?
- [ ] Check impact on users/accounts

**3. Deployment Rollback (if needed):**
```bash
# <FILL IN: Commands to rollback deployment>
# Example for Helm:
# helm list -n <namespace>
# helm history <release-name> -n <namespace>
# helm rollback <release-name> -n <namespace>

# Example for Kubernetes:
# kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

**4. Data Cleanup (if needed):**
- [ ] Remove test data created during testing
- [ ] Verify no test data in production accounts
- [ ] Check for any orphaned records
- [ ] Update test-data-<id>.md with cleanup status

**5. Verification After Rollback:**
- [ ] Previous version is running:
  ```bash
  # <FILL IN: Command to verify version>
  ```
- [ ] Test core functionality
- [ ] Confirm new feature is no longer available (expected)
- [ ] Check logs for errors
- [ ] Verify no data loss or corruption

**6. Post-Rollback Actions:**
- [ ] Create incident report (use template below)
- [ ] Update GitLab issue with findings
- [ ] Document in lessons learned
- [ ] Plan remediation:
  - Root cause analysis
  - Fix implementation
  - Additional tests needed
  - Enhanced monitoring
- [ ] Schedule re-test after fixes

### Incident Report Template (Optional)

```markdown
# Incident Report: <Assignment ID> <Feature Name> Rollback

## Summary
- **Date/Time:** YYYY-MM-DD HH:MM UTC
- **Duration:** X minutes
- **Severity:** [Critical/High/Medium/Low]
- **Impact:** [Description of what was affected]
- **Status:** Resolved via rollback

## Timeline
- HH:MM - Issue first observed
- HH:MM - Testing stopped
- HH:MM - Team notified
- HH:MM - Rollback decision made
- HH:MM - Rollback executed
- HH:MM - Service restored
- HH:MM - Verification complete

## Issue Description
[Detailed description of what went wrong]

### Symptoms
- [Symptom 1]
- [Symptom 2]

### Evidence
- [Link to logs or screenshots]
- [Error messages]
- [Test case that failed]

## Root Cause
[Analysis of why it happened]

### Technical Details
- [Code/configuration issue]
- [Dependency problem]
- [Data issue]

## Impact Assessment
- **Users affected:** [Number/description]
- **Accounts affected:** [Number/description]
- **Data integrity:** [OK/Compromised/Unknown]
- **Other services:** [Affected/Not affected]

## Resolution
Rolled back to revision X of <service-name>.

### Rollback Commands Used
```bash
<actual commands executed>
```

### Verification Steps
- [Step 1]
- [Step 2]

## Prevention Measures
[How to prevent this in the future]

### Code Changes Needed
- [ ] [Change 1] - Owner: ____ - Due: ____
- [ ] [Change 2] - Owner: ____ - Due: ____

### Process Improvements
- [ ] [Improvement 1]
- [ ] [Improvement 2]

### Additional Testing
- [ ] [Test 1]
- [ ] [Test 2]

## Action Items
- [ ] Fix root cause - Owner: ____ - Due: ____
- [ ] Add regression test - Owner: ____ - Due: ____
- [ ] Update documentation - Owner: ____ - Due: ____
- [ ] Review deployment process - Owner: ____ - Due: ____
- [ ] Schedule re-test - Owner: ____ - Due: ____

## Lessons Learned
- [Lesson 1]
- [Lesson 2]
- [Lesson 3]

## Sign-off
- **Incident Commander:** [Name]
- **Reviewed by:** [Name]
- **Date:** YYYY-MM-DD
```

### Emergency Contacts

- **Identity Team Lead:** <FILL IN: Contact info>
- **On-Call Engineer:** <FILL IN: Check Slack channel or PagerDuty>
- **DevOps/Platform Team:** <FILL IN: Contact info>
- **Engineering Manager:** <FILL IN: Contact info>

**Escalation Path:**
1. Team Lead (first 15 min)
2. Engineering Manager (if critical, after 30 min)
3. Director of Engineering (if prolonged outage)

---

## Assumptions Made (Optional)

Document key assumptions made during test planning:

1. **[Category]:** [Assumption statement] - [How to verify]
   - Example: **Endpoint Access:** Staging endpoint is accessible via VPN - Verify with ping/curl

2. **[Category]:** [Assumption statement] - [How to verify]

3. **[Category]:** [Assumption statement] - [How to verify]

<FILL IN: Add 8-10 assumptions covering:>
- Endpoint availability
- Authentication methods
- Test data creation capabilities
- Network access
- Upstream service availability
- Performance expectations
- Data format
- Token validity duration
- Test isolation
- No impact on production

---

## Notes for Template Improvement

**After using this template, consider:**
- What sections were most helpful?
- What information was missing?
- What could be clearer?
- What automation worked well?
- Were the test case counts appropriate?
- Was the rollback guidance useful?

**Suggest improvements by:**
- Adding comments to this template
- Updating the template directly
- Documenting lessons learned in LESSONS-LEARNED-<id>.md

---

*Copy this template for each assignment and fill in <FILL IN> sections as needed. Preserve this original template for future use.*

**Template Metadata:**
- **Version:** 3.0
- **Last Updated:** March 16, 2026
- **Status:** Template (Ready for Use)
- **Location:** AGENT-TEMPLATE/ (protected by .gitignore)
- **Testing Scope:** Post-deployment external testing (not unit/integration testing)

---

## Template Changelog

### Version 3.0 (February 3, 2026)
- Added document metadata section at top
- Added proto field naming verification section
- Added environment variable verification with expected output
- Added test data documentation checkpoint
- Enhanced test automation with Python-first approach
- Added response time column to verification matrix
- Added test case count guidance (10-13 minimum)
- Restructured troubleshooting with Issue/Solution format
- Enhanced rollback procedures with decision matrix
- Added full incident report template
- Added optional assumptions section
- Improved security cross-references

### Version 2.0 (February 2026)
- Added security notes and guidance.md references
- Enhanced environment configuration section
- Added test automation examples

### Version 1.0 (Initial)
- Base template created
