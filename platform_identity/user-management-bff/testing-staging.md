# HOW-TO: Identity Testing in Staging (Template for GitLab Projects)

---

## Document Metadata

**Version:** 1.0  
**Created:** February 2026  
**Status:** Template (Ready for Use)  
**Location:** AGENT-TEMPLATE/ (protected by .gitignore)  
**Purpose:** Standardized post-deployment testing guide for Identity assignments in GitLab-managed projects

---

## Description of Purpose
This template provides a standardized, step-by-step guide for testing Identity-related features and assignments in the Sinch staging environment for GitLab-based projects. It is designed to be copied and filled in for each specific assignment, ensuring consistency and completeness in the testing process.

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
11. Refined testing steps

---

## Scope
**Teams:** Identity
**Services:** Sinch Identity, User Management BFF, Auth0, IAM Build Adapter (MSI)
**Primary Repo:**
- `git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/internal-user-management-bff.git`
  - Service: Internal User Management BFF (gRPC-based BFF for user management UI and admin tools)
  - Working directory: ID-51_create_endpoint_find_users_roles_across_accounts

**Related Repos/Submodules:**
- `common/` submodule - Proto definitions and shared clients
  - Location: `gitlab.com/sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/common`
  - Contains IAM client, Auth0 client, shared services
- IAM Build Adapter (MSI) - Upstream service for role management
  - Proto: `common/proto/vendor/sinch/beehive/iambuildadapter/v1beta1/iambuildadapter.proto`

**Environments:** 
- Staging: us1tst-eks001 (Kubernetes cluster)
- Namespace: identity
- Endpoint: `internal-user-management-bff.us1tst.identity.staging.sinch.com:443`

**Use Case:** Testing new features, bug fixes, or assignments related to identity management, authentication, and user flows.

---

## Resource Prerequisite Checklist
- [ ] Staging VPN access (Sinch VPN)
- [ ] Staging user credentials (or ability to create new user)
- [ ] Access to Auth0 management portal (staging) - [Auth0 Staging](https://manage.auth0.com/dashboard/eu/staging-sinch/users)
- [ ] Access to GitLab repo: `internal-user-management-bff`
- [ ] GitLab permissions to view merge requests and pipelines
- [ ] kubectl access to us1tst-eks001 cluster
- [ ] kubectl context: `sinch/sinch-projects/gitlab-kubernetes-agent:us1tst-eks001`
- [ ] Docker or Podman installed (for local testing - optional)
- [ ] Minikube installed (for local Kubernetes testing - optional)
- [ ] Helm 3.x installed
- [ ] grpcui installed (for manual API testing) - `brew install grpcui`
- [ ] grpcurl installed (for automation) - `brew install grpcurl`
- [ ] Python 3.8+ (for test automation)
- [ ] Go 1.25+ (for local development)
- [ ] Authentication tokens configured (see Environment Configuration below)
- [ ] Git submodules initialized (`git submodule update --init --recursive`)
- [ ] Git access to review commit history (`git log`, `git show` commands)

---

## Environment Configuration

### Proto Field Naming Verification

⚠️ **CRITICAL:** Before filling in request/response examples, verify exact field names from proto files.

**This repo uses snake_case in proto definitions** (e.g., `user_id`, `page_size`, not camelCase).

**Steps:**
1. Locate the proto definition:
   ```bash
   # Search for the endpoint definition
   grep -r "rpc <EndpointName>" proto/
   
   # Example for SearchUserRolesAcrossAccounts:
   grep -A 20 "SearchUserRolesAcrossAccounts" proto/api/user_service.proto
   ```

2. Open the proto file and note exact field names (case-sensitive)

3. **Verify field naming convention:**
   - This repo uses **snake_case** in proto files: `user_id`, `page_size`, `page_token`
   - Generated Go code uses camelCase: `UserId`, `PageSize`, `PageToken`
   - JSON over gRPC uses camelCase: `"userId"`, `"pageSize"`, `"pageToken"`

4. Document proto file location and line number for reference

**Proto File Locations:**
- Service definitions: `proto/api/user_service.proto`
- Generated Go code: `gen/go/user_service.pb.go`, `gen/go/user_service_grpc.pb.go`
- Vendor/IAM protos: `common/proto/vendor/sinch/beehive/iambuildadapter/v1beta1/`
- Service manifest: `proto/api/iacuser.yaml` (MSI service registration)

**Proto Generation:**
```bash
# Update submodules first (required for common/ protos)
git submodule update --init --recursive

# Generate proto code
make protos
```

### Authentication Setup

⚠️ **Security Note:** Reference the `.env` file location only. Never display actual token values in documentation.

**Required Environment Variables:**

The repo uses a `.env` file in project root (DO NOT COMMIT - already in .gitignore).

**For Local Development (.env file):**
```bash
# Auth0 Configuration
AUTH0_CLIENT_ID=<from-auth0-dashboard>
AUTH0_CLIENT_SECRET=<from-auth0-dashboard>
AUTH0_DATABASE_CONNECTION=Sinch-Users
AUTH0_AUDIENCE=staging-sinch.eu.auth0.com
AUTH0_DOMAIN=staging-sinch.eu.auth0.com

# MSI/IAM Configuration
# This token is used for outbound calls to MSI services
MSI_BEARER_TOKEN_OVERRIDE=<token-from-oauth-exchange>

# GitLab (for fetching protos via submodules)
GITLAB_TOKEN=<check-if-already-configured>
MSI_FETCH=<check-if-already-configured>
```

**For Helm Deployment (Kubernetes secrets):**
- Auth0 secrets: `helm/devsecrets/dev/auth0-secret.yaml` (base64 encoded)
- MongoDB secrets: `helm/devsecrets/dev/mongo-secret.yaml`
- Sealed secrets for us1tst: `helm/sealedsecrets/us1tst/`
  - `internal-auth0-secret.yaml`
  - `internal-mongo-secret.yaml`
  - `mailgun-secret.yaml`

**Setup Dev Secrets (Local):**
```bash
# Copy template
cp helm/devsecrets/dev/auth0-secret.CHANGEME helm/devsecrets/dev/auth0-secret.yaml

# Encode your secret
echo -n '<your-auth0-secret>' | openssl base64

# Edit auth0-secret.yaml and paste base64 value
# Update helm/config/dev.yaml with your client ID and audience
```

**Token Acquisition:**

1. **Auth0 Credentials:** 
   - Dashboard: [Auth0 Staging](https://manage.auth0.com/dashboard/eu/staging-sinch)
   - Find application with your client ID
   - Copy client secret

2. **MSI Bearer Token (OAuth Exchange):**
   ```bash
   curl --request POST \
     --url https://staging-sinch.eu.auth0.com/oauth/token \
     --header 'content-type: application/json' \
     --data '{
       "client_id":"<AUTH0_CLIENT_ID>",
       "client_secret":"<AUTH0_CLIENT_SECRET>",
       "audience":"staging-sinch.eu.auth0.com",
       "grant_type":"client_credentials"
     }'
   ```
   - Extract `access_token` from response
   - Add to `.env` as `MSI_BEARER_TOKEN_OVERRIDE`

**Load Environment Variables:**
```bash
# Source .env
source .env

# Verify variables are loaded (shows names only, not values)
env | grep -E "AUTH0_|MSI_|GITLAB_" | cut -d= -f1
```

**Expected Output:**
```
AUTH0_CLIENT_ID
AUTH0_CLIENT_SECRET
AUTH0_DATABASE_CONNECTION
AUTH0_AUDIENCE
AUTH0_DOMAIN
MSI_BEARER_TOKEN_OVERRIDE
GITLAB_TOKEN
MSI_FETCH
```

---

## Test Data Setup

- [ ] **Test Users:** <FILL IN>
- [ ] **Test Accounts:** <FILL IN>
- [ ] **Test Roles:** <FILL IN>
- [ ] **Other Resources:** <FILL IN>

**Test Data Creation Steps:**
1. <FILL IN: Step 1>
2. <FILL IN: Step 2>
3. **Verification:** <FILL IN: How to verify test data is created correctly>
4. **Document Test Data:**
   - Create `test-data-<assignment-id>.md` in AGENT-TEMPLATE/ directory (not committed)

---

## Prioritized Step-by-Step Process

### Entry Prerequisite
- Ensure all items in the Resource Prerequisite Checklist are available and working.
- Confirm assignment details and required test cases.

### Steps

#### 1. Connect to Staging VPN
- Use VPN client to connect to Sinch VPN for infrastructure tools
- Verify connectivity:
  ```bash
  ping internal-user-management-bff.us1tst.identity.staging.sinch.com
  ```

#### 2. Verify Kubectl Access
```bash
# Set context to staging
kubectl config use-context sinch/sinch-projects/gitlab-kubernetes-agent:us1tst-eks001
kubectl config set-context --current --namespace=identity

# Verify access
kubectl get pods -n identity -l app=internal-user-management-bff
kubectl get svc -n identity internal-user-management-bff
```

#### 3. Check Deployment Status

**Deployment Process via GitLab:**

This service uses GitLab merge request deployments for staging:

1. **Pipeline Execution:**
   - Merge request triggers GitLab CI/CD pipeline
   - Pipeline stages: verify → build → test → publish → deploy
   - Pipeline builds Docker image tagged with `CI_COMMIT_SHORT_SHA`
   - Image pushed to Nexus registry

2. **Manual Deployment to Staging:**
   - Navigate to merge request in GitLab web interface
   - If permissions are configured: Manual deployment button appears
   - Click "Deploy to us1tst" (or equivalent staging environment button)
   - GitLab runs Helm upgrade to deploy new image to staging

3. **Verify Deployment:**
```bash
# Check recent Helm deployments
helm history internal-user-management-bff -n identity

# Verify current image version
kubectl get deployment internal-user-management-bff -n identity -o yaml | grep "image:"

# Check pod health
kubectl get pods -n identity -l app=internal-user-management-bff
kubectl describe pod -n identity -l app=internal-user-management-bff

# Check recent logs
kubectl logs -n identity -l app=internal-user-management-bff --tail=50 --since=5m
```

**Alternative: Check GitLab Pipeline Status:**
- Go to merge request → Pipelines tab
- View deployment status and logs
- Check environment status (Deployments → us1tst)

#### 4. Prepare Test User
- Use existing staging credentials or sign up at [dashboard.staging.sinch.com](https://dashboard.staging.sinch.com/)
- If user does not appear, check Auth0: [Auth0 Staging Users](https://manage.auth0.com/dashboard/eu/staging-sinch/users)
- Create user if needed via Auth0 dashboard or grpcui

#### 5. Assignment-Specific Testing

**Using grpcui (Manual Testing):**
```bash
# Connect to staging endpoint
grpcui -plaintext internal-user-management-bff.us1tst.identity.staging.sinch.com:443

# Available in browser UI:
# - InternalUserService.GetAvailableRoles
# - InternalUserService.InviteUser
# - InternalUserService.GetUserByEmail
# - InternalUserService.GetUserByID
# - InternalUserService.SearchUsers
# - InternalUserService.SearchUserRolesAcrossAccounts (THE ENDPOINT BEING TESTED)
```

**Using grpcurl (Automated Testing):**
```bash
# List services
grpcurl -plaintext internal-user-management-bff.us1tst.identity.staging.sinch.com:443 list

# Describe service
grpcurl -plaintext internal-user-management-bff.us1tst.identity.staging.sinch.com:443 describe internaluserpb.InternalUserService

# Example call
grpcurl -plaintext \
  -d '{"user_id": "auth0|...", "page_size": 10}' \
  internal-user-management-bff.us1tst.identity.staging.sinch.com:443 \
  internaluserpb.InternalUserService/SearchUserRolesAcrossAccounts
```

**Git History Context:**

*Use git history to understand implementation details and fill in test guidance:*

```bash
# Find assignment-related commits
git log --grep="<ASSIGNMENT-ID>|<EndpointName>" --oneline

# Example for ID-51:
git log --grep="ID-51|SearchUserRolesAcrossAccounts" --oneline
# Output: 14bff6b feat: add SearchUserRolesAcrossAccounts endpoint and tests

# View commit details
git show <commit-hash> --stat

# View proto changes
git show <commit-hash> proto/api/user_service.proto

# View handler implementation
git show <commit-hash> internal/api/handlers/grpc_handler.go

# View test implementation (helpful for test case ideas)
git show <commit-hash> internal/api/handlers/*_test.go
```

**Example Git History Analysis (ID-51):**
- Commit: `14bff6b` - "feat: add SearchUserRolesAcrossAccounts endpoint and tests"
- Files changed:
  - `proto/api/user_service.proto` - Added service definition (line 33+)
  - `internal/api/handlers/grpc_handler.go` - Added handler (line 37+)
  - `internal/api/handlers/grpc_handler_search_test.go` - Added tests
  - Generated files: `gen/go/user_service.pb.go`, `gen/go/user_service_grpc.pb.go`

**Proto Definition (from git history):**
```protobuf
// Returns all role assignments for the specified user across all accounts
rpc SearchUserRolesAcrossAccounts(SearchUserRolesAcrossAccountsRequest) 
    returns (SearchUserRolesAcrossAccountsResponse);

message SearchUserRolesAcrossAccountsRequest {
    string user_id = 1;      // Note: snake_case in proto
    int32 page_size = 2;
    string page_token = 3;
}

message SearchUserRolesAcrossAccountsResponse {
    repeated UserRoleAcrossAccount user_roles = 1;
    string next_page_token = 2;
}

message UserRoleAcrossAccount {
    string account_id = 1;
    string role_id = 2;
    string user_id = 3;
}
```

#### 6. Confirm Results
- Check Auth0 logs for user activity: [Auth0 Logs](https://manage.auth0.com/dashboard/eu/staging-sinch/logs)
- Verify endpoint functionality matches expected behavior
- Check application logs:
  ```bash
  kubectl logs -n identity -l app=internal-user-management-bff | grep -i "searching user roles"
  ```
- Document results in Verification Matrix

### Exit Conditions
- All required test cases executed and results documented.
- Assignment-specific acceptance criteria met.
- No critical issues blocking deployment or release.

---

## Verification Matrix

| Test Case | Description | Expected Result | Actual Result | Response Time | Status | Notes |
|-----------|-------------|-----------------|---------------|---------------|--------|-------|
| TC-1 | <FILL IN> | <FILL IN> | <FILL IN> | <FILL IN> | ⬜ Pass / ❌ Fail | |
| TC-2 | <FILL IN> | <FILL IN> | <FILL IN> | <FILL IN> | ⬜ Pass / ❌ Fail | |

---

## Performance Expectations

- Endpoint: `<FILL IN: endpoint name>`
- Expected latency: <FILL IN>
- Timeout threshold: <FILL IN>
- Concurrent requests: <FILL IN>
- Throughput: <FILL IN>

---

## Test Automation

**Primary Language:** Python (for consistency and advanced features)
**Alternative:** Bash (for simple, single-operation tasks only)

### Available Make Commands

```bash
make run          # Run service locally (uses helm/config/dev.yaml)
make test         # Run unit tests (with MongoDB)
make protos       # Generate proto files (requires submodule update)
make image        # Build Docker image locally
```

### GitLab CI/CD Pipeline

**Current Pipeline Stages (from .gitlab-ci.yml):**

1. **verify** - Proto generation and service lint
   ```yaml
   verify:
     image: golang:1.25
     script: make protos
   
   service-lint:
     image: nexus.../msi-cli:main-latest
     script: msi servicemanagement lint --verbose proto/api/iacuser.yaml
   ```

2. **build** - Go build
   ```yaml
   build:
     image: golang:1.25
     script:
       - go mod tidy
       - CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o ./bin/internal-user-management-bff ./cmd/server
     artifacts:
       paths: ./bin/internal-user-management-bff
   ```

3. **test** - Unit tests with MongoDB
   ```yaml
   test:
     image: golang:1.25
     services:
       - name: mongo:5.0
         alias: mongo
     script: make test
   ```

4. **publish** - Docker image to Nexus registry
   ```yaml
   publish-nexus:
     image: docker:25.0
     services: docker:25.0-dind
     script:
       - docker build -t "${NEXUS_REGISTRY}/dev/beehive/internal-user-management-bff:${CI_COMMIT_SHORT_SHA}" .
       - docker push "${NEXUS_REGISTRY}/dev/beehive/internal-user-management-bff:${CI_COMMIT_SHORT_SHA}"
   ```

5. **deploy** - Helm deployment to us1tst (manual trigger)
   ```yaml
   deploy:us1tst:
     stage: deploy
     when: manual
     environment:
       name: us1tst-eks001
     script:
       - kubectl config use-context sinch/sinch-projects/gitlab-kubernetes-agent:us1tst-eks001
       - kubectl config set-context --current --namespace=identity
       - helm upgrade --install internal-user-management-bff ./helm
         --values ./helm/config/us1tst.yaml
         --set image.repository="${NEXUS_REGISTRY}/dev/beehive/internal-user-management-bff"
         --set image.tag=${CI_COMMIT_SHORT_SHA}
         --set-file serviceDefinition.iacuser.file=proto/api/iacuser.yaml
   ```

**Key Pipeline Variables:**
- `NEXUS_REGISTRY` - Container registry URL
- `NEXUS_USER`, `NEXUS_PASSWORD` - Registry credentials
- `CI_COMMIT_SHORT_SHA` - Used as image tag
- `GO_BUILD_OUTPUT=internal-user-management-bff`
- `IMAGE_NAME=internal-user-management-bff`

### Adding Staging Tests to CI/CD

**Add to `.gitlab-ci.yml` after `test:` stage:**

```yaml
test:<assignment>:staging:
  stage: test
  image: python:3.11
  before_script:
    - apt-get update && apt-get install -y grpcurl
    - pip install grpcio grpcio-tools
  script:
    - source .env
    - python3 AGENT-TEMPLATE/test_<assignment>_<id>.py
  only:
    - merge_requests
  when: manual
  environment:
    name: us1tst-eks001
  allow_failure: true
  artifacts:
    when: always
    paths:
      - test-results/
    reports:
      junit: test-results.xml
```

### Python Test Script Template

Create `AGENT-TEMPLATE/test_<assignment>_<id>.py` following the v3 template pattern:
- Performance metrics tracking (avg/min/max response times)
- Color-coded output (GREEN/RED/YELLOW/BLUE)
- grpcurl via subprocess
- Test data from `test-data-<id>.md`
- Organized sections (main/edge/pagination tests)

### Local Testing Workflow

```bash
# 1. Run locally
make run

# 2. Test with grpcui (in another terminal)
grpcui -plaintext localhost:9090

# 3. Or test staging
grpcui -plaintext internal-user-management-bff.us1tst.identity.staging.sinch.com:443

# 4. Run automated tests
python3 AGENT-TEMPLATE/test_<assignment>_<id>.py
```

---

## Troubleshooting

### Connection Issues

**Issue:** Cannot connect to staging gRPC endpoint
**Solution:**
```bash
# Check VPN
ifconfig | grep tun

# Check DNS
nslookup internal-user-management-bff.us1tst.identity.staging.sinch.com

# Test with grpcui
grpcui -plaintext internal-user-management-bff.us1tst.identity.staging.sinch.com:443

# Check service in Kubernetes
kubectl config use-context sinch/sinch-projects/gitlab-kubernetes-agent:us1tst-eks001
kubectl get svc -n identity internal-user-management-bff
kubectl get ingress -n identity
```

### Authentication Errors

**Issue:** Permission denied or authentication errors
**Solution:**
```bash
# Verify environment variables
env | grep -E "AUTH0_|MSI_" | cut -d= -f1

# Check token expiration (Auth0 tokens typically 24h)
# Refresh MSI bearer token
curl --request POST \
  --url https://staging-sinch.eu.auth0.com/oauth/token \
  --header 'content-type: application/json' \
  --data '{"client_id":"<from-.env>","client_secret":"<from-.env>","audience":"staging-sinch.eu.auth0.com","grant_type":"client_credentials"}'
```

### Proto Generation Fails

**Issue:** `make protos` fails
**Solution:**
```bash
# Update submodules (common/ contains shared protos)
git submodule update --init --recursive --force
git submodule foreach git pull origin main

# Clean and regenerate
rm -rf gen/go/
make protos

# Check buf.gen.yaml and buf.work.yaml are correct
```

### IAM Client Error

**Issue:** "userService does not expose IAMClient; cannot call SearchUserRoles"
**Solution:**
- This indicates configuration issue in deployment
- Check `common/pkg/clients/iam` is properly initialized
- Verify `MSI_BEARER_TOKEN_OVERRIDE` is set in deployment
- Check handler implementation in `internal/api/handlers/grpc_handler.go` line ~37-90
- Review dependency injection of IAM client in user service

### User Not Found

**Issue:** User not found in Auth0 or system
**Solution:**
```bash
# Check Auth0 dashboard
# https://manage.auth0.com/dashboard/eu/staging-sinch/users

# Verify user ID format (should be auth0|...)
# Search by email if ID unknown
# Check domain: staging.sinch.com vs sinch.com
```

### Data Not Found / Empty Response

**Issue:** Expected user roles not returned
**Solution:**
```bash
# Cross-verify with GetUserByID
grpcui -plaintext internal-user-management-bff.us1tst.identity.staging.sinch.com:443
# Call GetUserByID with user_id and account_id

# Check application logs
kubectl logs -n identity -l app=internal-user-management-bff --tail=100 | grep -i "search.*roles"

# Verify IAM Build Adapter is accessible
# Check upstream service health
```

### Helm Deployment Fails

**Issue:** Helm upgrade/install fails
**Solution:**
```bash
# Check secrets exist
kubectl get secrets -n identity

# Verify sealed secrets are applied
ls helm/sealedsecrets/us1tst/
kubectl get sealedsecrets -n identity

# Verify config file
cat helm/config/us1tst.yaml

# Check Helm release status
helm list -n identity
helm status internal-user-management-bff -n identity

# Debug deployment
helm upgrade --install internal-user-management-bff ./helm \
  --values ./helm/config/us1tst.yaml \
  --dry-run --debug
```

### Local Minikube Issues

**Issue:** Local deployment not working
**Solution:**
```bash
# Check minikube status
minikube status

# Verify context
kubectl config current-context  # Should be minikube

# Check ingress
kubectl get pods -n ingress-nginx

# Verify Docker environment
eval $(minikube -p minikube docker-env)
docker images | grep internal-user-management-bff

# Check namespace
kubectl get namespaces | grep identity

# View service URL
minikube service internal-user-management-bff --url -n identity
```

### Performance Issues

**Issue:** Response times > 2s
**Solution:**
```bash
# Check network latency
ping internal-user-management-bff.us1tst.identity.staging.sinch.com

# Check VPN performance
# Test at different times

# Check pod resources
kubectl top pods -n identity -l app=internal-user-management-bff

# Check pod logs for slow queries
kubectl logs -n identity -l app=internal-user-management-bff | grep -i "slow\|timeout"

# Check IAM Build Adapter health (upstream service)
```

### GitLab CI/CD Pipeline Fails

**Issue:** Pipeline job fails
**Solution:**
```bash
# Check job logs in GitLab UI
# Common issues:
# - Submodule not updated (verify stage)
# - Go dependencies (go mod tidy)
# - Docker build fails (check Dockerfile)
# - Nexus registry auth (check NEXUS_USER/PASSWORD variables)
# - kubectl context (check gitlab-kubernetes-agent connection)
```

---

## Rollback Procedures

**Note:** Rollback is handled via GitLab CI/CD or Helm/Kubernetes commands.

### When to Rollback

- Critical bugs discovered during staging testing
- IAM Build Adapter integration failures
- Performance degradation (> 5s response times)
- Data inconsistency issues
- Security vulnerabilities in the endpoint
- Deployment failures

### Kubernetes Context Setup

```bash
# Set kubectl context for us1tst staging
kubectl config use-context sinch/sinch-projects/gitlab-kubernetes-agent:us1tst-eks001

# Set namespace
kubectl config set-context --current --namespace=identity

# Verify context
kubectl config current-context
kubectl config view --minify | grep namespace
```

### Rollback via Helm

**Check Release History:**
```bash
# List all releases
helm list -n identity

# Check release history
helm history internal-user-management-bff -n identity
```

**Example Output:**
```
REVISION  UPDATED                   STATUS      CHART                           APP VERSION  DESCRIPTION
1         Mon Feb 03 10:15:43 2026  superseded  internal-user-management-bff-1.0.0  1.0.0       Install complete
2         Mon Feb 03 14:30:22 2026  deployed    internal-user-management-bff-1.0.0  1.0.0       Upgrade complete
```

**Rollback Commands:**
```bash
# Rollback to previous release
helm rollback internal-user-management-bff -n identity

# Rollback to specific revision
helm rollback internal-user-management-bff 1 -n identity

# Rollback with debug output
helm rollback internal-user-management-bff -n identity --debug
```

**Verify Rollback:**
```bash
# Check deployment image tag
kubectl get deployment internal-user-management-bff -n identity -o yaml | grep "image:"

# Check pods are running
kubectl get pods -n identity -l app=internal-user-management-bff

# Check pod logs
kubectl logs -n identity -l app=internal-user-management-bff --tail=50

# Check service endpoints
kubectl get svc -n identity internal-user-management-bff
```

### Rollback via GitLab UI

**Alternative Method:**
1. Go to GitLab project: `sinch-identity/internal-user-management-bff`
2. Navigate to: **Deployments → Environments → us1tst-eks001**
3. Find previous successful deployment
4. Click **Re-deploy** button
5. Verify in kubectl that rollback succeeded

### Rollback via kubectl (Emergency)

```bash
# Rollback deployment (if Helm unavailable)
kubectl rollout undo deployment/internal-user-management-bff -n identity

# Check rollout status
kubectl rollout status deployment/internal-user-management-bff -n identity

# Check rollout history
kubectl rollout history deployment/internal-user-management-bff -n identity
```

### Post-Rollback Actions

1. **Verify Service:**
   ```bash
   # Test endpoint with grpcui
   grpcui -plaintext internal-user-management-bff.us1tst.identity.staging.sinch.com:443
   
   # Test core functionality (GetUserByID, etc.)
   ```

2. **Check Logs:**
   ```bash
   kubectl logs -n identity -l app=internal-user-management-bff --tail=100
   ```

3. **Monitor for Errors:**
   - Check Grafana/monitoring dashboards
   - Monitor error rates
   - Verify response times are normal

4. **Notification:**
   - Comment on GitLab merge request with rollback status
   - @ mention relevant team members in GitLab
   - Update GitLab issue status
   - Consider team communication channel if applicable

5. **Incident Documentation:**
   - Create incident report
   - Update `LESSONS-LEARNED-<id>.md`
   - Add rollback details to GitLab issue
   - Plan remediation and re-test

### Emergency Contacts

**Identity Team:**
- GitLab: @ mention team members in merge request or issue
- On-Call: Check PagerDuty or team roster
- Team communication channel (if configured)

**Platform/DevOps:**
- GitLab: Create issue in platform/infrastructure project
- Kubernetes support: Check team documentation for escalation path

**Escalation Path:**
1. Team Lead (first 15 min) - @ mention in GitLab MR
2. Engineering Manager (if critical, after 30 min)
3. Director of Engineering (if prolonged outage)

---

*Copy this template for each assignment and fill in <FILL IN> sections as needed. Preserve this original template for future use.*

---

## Local Development (Optional)

For testing changes locally before staging deployment.

### Prerequisites
- Minikube installed: `brew install minikube`
- Helm 3.x installed: `brew install helm`
- Docker running
- Go 1.25+: `brew install go`

### Quick Local Run (Without Kubernetes)

```bash
# 1. Setup environment
cp .env.example .env
# Edit .env with your Auth0 credentials and MSI token

# 2. Source environment
source .env

# 3. Run locally (uses helm/config/dev.yaml)
make run

# 4. Test in another terminal
grpcui -plaintext localhost:9090
```

### Full Local Kubernetes Setup

```bash
# 1. Start minikube
minikube start

# 2. Switch kubectl context
kubectl config use-context minikube
kubectl config current-context  # Verify

# 3. Enable ingress
minikube addons enable ingress
kubectl get pods -n ingress-nginx  # Verify health

# 4. Configure to use minikube's Docker daemon
eval $(minikube -p minikube docker-env)

# 5. Build Docker image locally
make image

# 6. Setup dev credentials
cp helm/devsecrets/dev/auth0-secret.CHANGEME helm/devsecrets/dev/auth0-secret.yaml

# Encode your Auth0 secret
echo -n '<your-auth0-client-secret>' | openssl base64

# Edit auth0-secret.yaml:
# apiVersion: v1
# kind: Secret
# metadata:
#   name: auth0-secret
# type: Opaque
# data:
#   AUTH0_CLIENT_SECRET: <base64-encoded-value>

# 7. Update helm/config/dev.yaml with your:
#    - AUTH0_CLIENT_ID
#    - AUTH0_AUDIENCE
#    - AUTH0_DOMAIN

# 8. Create namespace
kubectl create namespace identity
kubectl get namespaces  # Verify

# 9. Deploy with Helm
helm upgrade --install internal-user-management-bff ./helm \
  -n identity \
  --values ./helm/config/dev.yaml

# 10. Check deployment
kubectl get pods -n identity
kubectl logs -n identity -l app=internal-user-management-bff

# 11. Expose service and get URL
minikube service internal-user-management-bff --url -n identity

# Example output: http://192.168.49.2:30123

# 12. Test with grpcui (use the URL from step 11)
grpcui -plaintext 192.168.49.2:30123
```

### Cleanup Local Environment

```bash
# Stop minikube
minikube stop

# Delete minikube cluster (if needed)
minikube delete

# Or just uninstall Helm release
helm uninstall internal-user-management-bff -n identity
```

---
# Refined testing steps
1. Get test user from: https://account.admin.stg.saas.sinch.com/accounts/1c8e24c7-dba4-48bb-8210-23869841eb1f/user-management
1. Open Browser tab and press 12 for Dev tools; go to Network tab
1. Log into : https://account.admin.stg.saas.sinch.com/accounts/1c8e24c7-dba4-48bb-8210-23869841eb1f/user-management
1. FInd "GetAvaialbleRoles" and get the "authorization: Bearer" token value
1. put token value into file: MSI

## Console Query:

        ``` 
        grpcurl -insecure -vv  -format-error \
        -H "authorization: Bearer $(cat MSI)" \                
        -d '{
          "user_id": "8dad5baa-d596-4bfa-ac26-72aefe535cfb",
          "page_size" : 1
        }' internal-user-management-bff.us1tst.identity.staging.sinch.com:443 internaluserpb.InternalUserService.SearchUserRolesAcrossAccounts
        ```

    - Note: for this unit of work, "page_size" is a required field by the client.
    - Note: for this unit of work, "page_token" is not required but I don't know what string value to.

## Local GUI Query:

    Alternatively, you can access the full service interface with:
    ```
    grpcui internal-user-management-bff.us1tst.identity.staging.sinch.com:443
    ```

    Then access the local GUI from http://127.0.0.1:52543/ or similar, 
    configure teh authorization, user_id and page_size as before and invoke the endpoint.
1. Access Grafana logs: https://grafana.int.prod.sinch.com/d/70cc461c-3fdc-4279-b6a6-0d15cfabb833/internal-user-management?orgId=1&from=now-30m&to=now&timezone=utc&viewPanel=panel-1

---

## Template Changelog

### Version 2.1 (February 4, 2026)
- Added git history access to prerequisites
- Added git history analysis instructions (git log, git show commands)
- Updated deployment section with GitLab merge request deployment process
- Removed Slack-specific references, replaced with GitLab-centric workflow
- Updated notification procedures to use GitLab merge request comments
- Updated emergency contacts to be GitLab-first (@ mentions, issues)
- Clarified manual deployment via GitLab web interface

### Version 2.0 (February 4, 2026)
- Updated with actual repo details from `internal-user-management-bff`
- Added correct GitLab repo URL and structure
- Added proto field naming verification (snake_case in this repo)
- Added actual environment variables from .env and Helm secrets
- Added GitLab CI/CD pipeline stages (verify, build, test, publish, deploy)
- Added kubectl context: `sinch/sinch-projects/gitlab-kubernetes-agent:us1tst-eks001`
- Added namespace: identity
- Added Makefile commands (run, test, protos, image)
- Added comprehensive rollback procedures (Helm, kubectl, GitLab UI)
- Added local Minikube development setup
- Added repo-specific troubleshooting (proto generation, IAM client, Helm deployment)
- Added git history context for ID-51 implementation (commit 14bff6b)
- Added proto definitions from actual implementation
- Added authentication architecture (Auth0 client credentials, MSI bearer token)
- Updated test automation with Python-first approach
- Added grpcui/grpcurl examples with actual endpoint
- Added sealed secrets locations for us1tst environment

### Version 1.0 (February 2026)
- Initial version for GitLab-managed projects
- Adapted from GitHub/Director-based template
- CI/CD and rollback steps updated for GitLab workflows
