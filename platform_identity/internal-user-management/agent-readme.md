# Internal User Management BFF

**Repo:** git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/internal-user-management-bff.git
**Last Updated:** 2026-02-24

---

## Table of Contents
- [AI Agent Implementation Guidelines](#ai-agent-implementation-guidelines)
- [Consistency and Pattern Adherence](#consistency-and-pattern-adherence)
- [Relationship to Common Submodule (IAM Client)](#relationship-to-common-submodule-iam-client)
- [Logging and Commit Practices](#logging-and-commit-practices)
- [Project Overview](#project-overview)
- [System Architecture](#system-architecture)
- [Development Environment](#development-environment)
- [Build & Deployment](#build--deployment)
- [Project Structure](#project-structure)
- [Key Files](#key-files)
- [Testing Strategy](#testing-strategy)
- [Security & Compliance](#security--compliance)
- [Monitoring & Operations](#monitoring--operations)
- [Technical Stack & Dependencies](#technical-stack--dependencies)
- [Service Contract](#service-contract)
- [Core Business Functions](#core-business-functions)
- [Configuration](#configuration)
- [Critical Commands](#critical-commands)
- [Additional Technical Details](#additional-technical-details)
- [Common Submodule Workflow](#common-submodule-workflow)

---

## AI Agent Implementation Guidelines

### Consistency and Pattern Adherence

All new features and changes must follow the existing patterns and implementation styles in this repository. Review the current codebase for conventions and structure, and match these in your work unless a justified, documented exception is required.

## Relationship to Common Submodule (IAM Client)

The internal-user-management-bff uses the `common` submodule as a Git submodule for shared code including the IAM client, service interfaces, utilities, and proto definitions. Understanding this relationship is critical for proper development:

**Architecture:**
- The BFF implements gRPC handlers that receive and validate requests
- Business logic (user management, role operations) is delegated to the common submodule's services and IAM client
- The BFF returns results from the common services back to gRPC callers
- This ensures clean separation between API interface (BFF) and reusable business logic (common)

**Repository Separation:**
- **Root Repo:** `internal-user-management-bff` (this repository) - API layer and BFF-specific logic
- **Common Repo:** `common` (separate repository, included as submodule) - Shared services, IAM client, utilities
- **URL:** `git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/common.git`

**Development Workflow:**
- Common has its own repository, PR process, and approval workflow
- Changes to common must be reviewed and merged independently
- Common PRs must be merged to main **before** root repo PRs reference them
- Root repo always points to common main branch, never feature branches
- See [Common Submodule Workflow](#common-submodule-workflow) section for detailed procedures

**Design Benefits:**
- BFF focuses on API orchestration, validation, and security
- Common encapsulates reusable business logic and external integrations
- Multiple services can share common code without duplication
- Clear dependency boundaries between API and business layers

### Critical Requirements
- **Read and adhere** to this file for all units of work and editing sessions.
- **Repository validation:** Ensure this document matches the actual repository structure and purpose. Notify the user of any discrepancies.

### Focus Areas
1. Service Layer: Primary business logic location (in common submodule)
2. gRPC Handler: API endpoint implementations (in root repo)
3. MongoDB Models: Data persistence patterns
4. Auth0 Integration: Authentication flows
5. Email Workflows: Mailgun integration patterns

### Security Requirements
- No hardcoded secrets in any checked-in files
- Input validation on all API endpoints
- Auth0 JWT validation for authenticated endpoints
- Secure email templates for invite notifications
- MongoDB injection prevention in queries

### Architectural Patterns
- Dependency Injection: Constructor-based DI throughout
- Interface Segregation: Use established service interfaces
- Error Wrapping: Consistent error handling with context
- Logging: Structured logging with correlation IDs
- Testing: Mock external dependencies, test business logic
- Code Quality: Static analysis with golangci-lint enforced

### Change Implementation Strategy
1. Analyze existing patterns in target service/handler
2. Follow established error handling conventions
3. Maintain interface compatibility with existing consumers
4. Add comprehensive tests alongside implementation (>90% unit coverage for new features, 100% for defects)
5. Update integration tests if new gRPC methods added (100% coverage for new features)

### Testing Ecosystem
- Unit tests: go test ./... (>90% coverage for new features, 100% for defects)
- Integration tests: Not currently implemented in Makefile
- Inline tests: internal/api/*_test.go
- Test coverage: `go test -cover ./...`

### Performance and Error Handling
- MongoDB indexing and connection pooling
- Prometheus metrics for key operations
- Service errors wrapped with context using pkg/errors patterns
- gRPC status codes mapped from business errors  
- Structured logs with correlation IDs
- Input validation at API boundaries

### Deployment & Build Integration
- Make-based build system
- Docker support: make image for containerization
- Proto generation: Automated with buf

### Safety Constraints
- rm commands cannot be auto-accepted; require explicit user confirmation
- Destructive operations must be user-approved
- All code must pass code quality checks (go vet, golangci-lint) with no critical issues

## Logging and Commit Practices

- When logging, PII (personally identifiable information) such as email addresses and phone numbers must be hashed before logging. Never log raw PII.
- For automated git commits, always use very terse commit messages (e.g., `feat: add X`, `fix: Y`, `chore: Z`).

---

## Prerequisites

**Required Software:**
- Go 1.25+ (go.mod specifies 1.25.7)
- MongoDB for data persistence
- Docker and Docker Compose (for local MongoDB and containerization)
- Make utility for build automation
- buf CLI for protocol buffer generation
- golangci-lint for code quality checks
- Git with submodule support
- envconfig for environment variable management

**Required Access:**
- GitLab access to sinch/internal-user-management-bff repository
- GitLab access to sinch/common submodule repository
- GITLAB_TOKEN environment variable for proto fetching
- Auth0 client credentials (from sealed secrets or dev configuration)
- MSI bearer token for outbound service calls

**Required Tools:**
- kubectl for Kubernetes operations (local deployment)
- helm for Kubernetes deployment
- minikube for local Kubernetes cluster (optional for local deployment)
- kubeseal for managing sealed secrets
- sinch CLI with sealed plugin for secret management

**Note:** If prerequisites are missing, builds and tests will fail with clear error messages.
## Project Overview

**Internal User Management BFF** is a Backend-for-Frontend (BFF) service that provides user management capabilities for Sinch's identity platform. This microservice acts as an intermediary between admin UI applications and backend identity services, handling user lifecycle management, invitation workflows, and role-based access control.

**Purpose:** Simplify user management operations for admin tools by aggregating multiple backend services (Auth0, IAM, Account services) into cohesive business operations.

**Business Context:** This service operates in the user-management domain within Sinch's enterprise messaging platform, handling sensitive user data and authentication flows. It supports multi-tenant account management with role-based permissions and project-level access control.


**Repo**: git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/internal-user-management-bff.git
**Repo Classification**: backend middleware (BFF)

## System Architecture

**Architecture Pattern:** Layered Architecture with clear separation between API, business logic, and data access layers.

**Core Components:**
- **API Layer:** primarily gRPC (and very few REST)git handlers for client communication
- **Service Layer:** Business logic for user, invite, role, and account operations  
- **Repository Layer:** Data access abstractions for MongoDB persistence
- **Client Layer:** External service integrations (Auth0, IAM, Account services)

**Data Flow:**
1. API requests received via gRPC or REST endpoints
2. Handlers delegate to service layer for business logic
3. Services coordinate between external clients and local repositories
4. MongoDB provides persistent storage for users and invitations
5. External services (Auth0, IAM) handle authentication and authorization

**Key Relationships:**
- Users belong to accounts with specific roles
- Invitations create pending user relationships
- Projects provide granular access control within accounts
- Auth0 manages authentication and user credentials
- IAM service handles role-based authorization

## Development Environment

**Prerequisites:**
- Go 1.25.1+
- MongoDB (local or containerized)
- Docker for containerization
- Make utility for build automation
- Auth0 tenant with configured database connection

**Local Setup:**
1. Clone repository with submodules: `git clone --recursive git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/internal-user-management-bff.git`
2. Navigate to project root
3. Copy and configure secrets: `cp helm/devsecrets/dev/auth0-secret.CHANGEME helm/devsecrets/dev/auth0-secret.yaml`
4. Set environment variables for Auth0 and MSI services
5. Install dependencies: `go mod download`
6. Run locally: `make run` (uses dev configuration)

**Note:** Always use `--recursive` when cloning to automatically initialize the `common` submodule. If already cloned without `--recursive`, run `git submodule update --init --recursive` to initialize submodules.

**Configuration:**
- Dev config: `helm/config/dev.yaml`
- Environment overrides supported for all configuration values
- Secrets managed via Kubernetes sealed secrets in production

## Build & Deployment

**Build Commands:**
- `make build` - Compile binary with formatting and dependency management (`go mod tidy`, `go fmt`, build to `bin/internal-user-management-bff`)
- `make image` - Build containerized image for deployment (CGO_ENABLED=0, Linux AMD64)
- `make run` - Build and run with APP_ENV=dev
- `make run-local` - Build and run with environment from `.env` file
- `make protos` - Generate protobuf code (install tools, clean, generate from buf configuration)
- `make generate` - Generate protos using `buf generate proto/api`

**Testing:**
- Unit tests: `make test` - runs `go test ./...`
- Test coverage: `go test -cover ./...`
- Coverage target: >90% for new features, 100% for defect fixes

**Code Quality:**
- `make code-check` - Full code quality analysis (go vet, golangci-lint with duplication/complexity/cognition checks, test coverage)
- `make code-check-quick` - Quick critical issues check for CI/CD (go vet, errcheck, unused)
- `make clean` - Clean build artifacts (`go clean`, remove binary)

**Deployment:**
- **Local Kubernetes:** Use minikube and helm for local deployment
  - `minikube start` - Start local cluster
  - `eval $(minikube -p minikube docker-env)` - Configure to use minikube's Docker daemon
  - `make image` - Build local Docker image
  - `helm upgrade --install internal-user-management-bff ./helm -n identity --values ./helm/config/dev.yaml`
  - `minikube service internal-user-management-bff --url -n identity` - Expose endpoints
- **Production:** Kubernetes with Helm charts and sealed secrets via GitLab CI/CD pipeline
- **CI/CD:** Automated builds and deployments via [`.gitlab-ci.yml`](.gitlab-ci.yml)

**Container Strategy:**
- Dockerfile for optimized image build
- Health check endpoints for container orchestration
- Graceful shutdown handling for rolling deployments

## Project Structure

- [`cmd/server`](cmd/server) - Application entry point and server initialization
- [`internal/api`](internal/api) - gRPC API handlers and routing (handlers/ and docs/)
- [`internal/config`](internal/config) - Configuration management and validation
- [`proto/`](proto/) - Protocol buffer definitions (api/ subdirectory)
- [`gen/`](gen/) - Generated code from protocol buffers
- [`common/`](common/) - **Git submodule** containing shared IAM client, services, and utilities (separate repository)
- [`helm/`](helm/) - Kubernetes deployment charts and configurations (config/, devsecrets/, sealedsecrets/)
- [`scripts/proto/`](scripts/proto/) - Protocol buffer management automation (fetch.sh for proto dependencies)
- [`.env.example`](.env.example) - Example environment variables file
- [`Dockerfile`](Dockerfile) - Container build instructions
- [`Makefile`](Makefile) - Build automation and development tasks
- [`buf.gen.yaml`](buf.gen.yaml) - Buf configuration for proto generation
- [`buf.work.yaml`](buf.work.yaml) - Buf workspace configuration
- [`.gitlab-ci.yml`](.gitlab-ci.yml) - GitLab CI/CD pipeline configuration

## Key Files

- [`cmd/server/main.go`](cmd/server/main.go) - Application bootstrap and dependency injection
- [`internal/api/handlers/`](internal/api/handlers/) - gRPC service handlers implementation
- [`internal/api/docs/`](internal/api/docs/) - API documentation
- [`internal/config/config.go`](internal/config/config.go) - Configuration loading and validation
- [`proto/api/`](proto/api/) - Protocol buffer service definitions
- [`helm/config/dev.yaml`](helm/config/dev.yaml) - Development configuration
- [`helm/config/us1tst.yaml`](helm/config/us1tst.yaml) - Testing environment configuration
- [`helm/devsecrets/`](helm/devsecrets/) - Development secret templates
- [`helm/sealedsecrets/`](helm/sealedsecrets/) - Encrypted production secrets
- [`Dockerfile`](Dockerfile) - Production container configuration
- [`Makefile`](Makefile) - Build automation
- [`README.md`](README.md) - Project documentation
- [`README-Claude.md`](README-Claude.md) - Additional AI agent documentation

## Testing Strategy

**Current State:**
- Test infrastructure configured with `make test` target executing `go test ./...`
- golangci-lint integration for static analysis and code quality checks
- testify v1.11.1 and mock v0.5.0 available for test development

**Recommended Testing Approach:**

**Unit Testing:**
- Place unit tests alongside source code using `*_test.go` naming convention
- Use `github.com/stretchr/testify` for assertions and mocking
- Test business logic validation in service layer independently
- Mock external service dependencies using go.uber.org/mock
- Validate error handling and edge cases thoroughly

**Integration Testing:**
- Test end-to-end gRPC API workflows
- Use MongoDB test containers for repository interface testing
- Verify external service integration (Auth0, IAM service)
- Test authentication and authorization flows with real JWT validation

**Coverage Requirements:**
- Target >90% coverage for new features
- Require 100% coverage for defect fixes
- Focus critical path testing on user and invitation workflows
- Run coverage reports with `go test -cover ./...`

**Test Organization Best Practices:**
- Co-locate unit tests with source code (`*_test.go`)
- Consider `internal/test/` directory for integration test suites
- Create `internal/test/fixtures/` for test data and utilities
- Generate mocks in `internal/test/mocks/` or alongside tested code

## Security & Compliance

**Authentication:**
- Auth0 JWT token validation for all protected endpoints
- Service-to-service authentication via MSI bearer tokens
- Database connection authentication with MongoDB credentials

**Authorization:**
- Role-based access control (RBAC) via IAM service integration
- Account-level and project-level permission validation
- Admin role protection against self-modification

**Data Protection:**
- PII hashing in all log outputs using `logging.Hash()`
- Structured logging with correlation IDs for audit trails
- Input validation and sanitization for all user inputs
- Password reset functionality through Auth0 secure channels

**Security Standards:**
- No hardcoded credentials in source code
- Environment-based secret management
- Sealed secrets for Kubernetes deployments
- Security scanning via static analysis tools

## Monitoring & Operations

**Logging:**
- Structured logging with JSON format via `go.uber.org/zap`
- Correlation ID propagation for request tracing
- PII-safe logging with hash functions for sensitive data
- Configurable log levels for production debugging

**Metrics:**
- Prometheus metrics for HTTP and gRPC endpoints
- Request duration, count, and error rate tracking
- Custom business metrics for user operations
- Health check endpoints for container orchestration

**Observability:**
- Request correlation across service boundaries
- Error tracking with detailed context propagation
- Performance monitoring for database and external service calls
- Audit logging for security-relevant operations

**Troubleshooting:**
- Health endpoints: `/health/liveness` and `/health/readiness`
- Metrics endpoint: `/metrics` for Prometheus scraping
- Debug-level logging for external service interactions
- Graceful shutdown handling with proper cleanup

## Technical Stack & Dependencies

**Runtime Environment:**
- Go 1.25.7 with module support
- Docker containers for deployment
- Kubernetes orchestration with Helm charts

**Core Framework & Libraries:**
- google.golang.org/grpc v1.78.0: gRPC server and client
- google.golang.org/protobuf v1.36.11: Protocol buffer support
- github.com/gin-gonic/gin v1.11.0: HTTP framework (REST endpoints)
- Common submodule: Shared IAM client, services, and utilities

**Configuration & Environment:**
- github.com/kelseyhightower/envconfig v1.4.0: Environment variable configuration
- github.com/joho/godotenv v1.5.1: .env file support
- gopkg.in/yaml.v3 v3.0.1: YAML configuration parsing

**Database:**
- go.mongodb.org/mongo-driver v1.17.6: MongoDB client for user data persistence
- Connection pooling and transaction support
- Index optimization for query performance

**Authentication & Authorization:**
- github.com/auth0/go-auth0 v1.32.0: Auth0 integration for identity management
- github.com/golang-jwt/jwt/v5 v5.3.0: JWT token validation
- github.com/lestrrat-go/jwx/v2 v2.1.6: Advanced JWT operations
- IAM service integration via common submodule for role-based authorization

**Observability:**
- github.com/prometheus/client_golang v1.23.2: Prometheus metrics collection and exposition
- go.uber.org/zap v1.27.0: Structured logging

**Email Integration:**
- github.com/mailgun/mailgun-go/v5 v5.8.1: Mailgun API integration for email delivery

**Testing & Quality:**
- github.com/stretchr/testify v1.11.1: Test assertions and mocking
- go.uber.org/mock v0.5.0: Mock generation
- golangci-lint: Static analysis and code quality checks

**Development Tools:**
- buf: Protocol buffer management and code generation
- buf.build/gen/go/bufbuild/protovalidate: Proto validation

## Service Contract

**Primary Service:** Internal User Management via gRPC API (`InternalUserService`)

**gRPC Service Definition:**
- Protocol buffer definition: [`proto/api/user_service.proto`](proto/api/user_service.proto)
- Generated code output: [`gen/`](gen/) directory via buf
- IAM integration using `sinch_id.rpc.v1` annotations for permission enforcement

**API Categories:**

**User Operations:**
- `GetUserByID` - Retrieve user by internal ID
- `GetUserByEmail` - Retrieve user by email address
- `GetUsers` - List users with filtering
- `SearchUsers` - Search users across accounts
- `UpdateUser` - Update user details and role assignments
- `RemoveUser` - Remove user with admin protection
- `GenerateNewUserGUID` - Generate unique user identifier
- `SearchUserRolesAcrossAccounts` - Query all role assignments for a user

**Invitation Operations:**
- `InviteUser` - Create email-based invitation with role and project assignment
- `GetInviteByToken` - Retrieve invitation details by token
- `GetPendingUserInvite` - Get specific pending invitation
- `GetPendingInvites` - List pending invitations
- `SearchInvites` - Search invitations with filtering
- `AcceptInvite` - Accept invitation and link to user account
- `CancelInvite` - Cancel pending invitation

**Role & Project Management:**
- `GetAvailableRoles` - List available roles in system
- `GetProjectsForAccount` - List projects for specific account
- `SearchProjectsForAccount` - Search projects within account

**Security Operations:**
- `SendPasswordResetEmail` - Trigger password reset via Auth0
- `ResetUserMFA` - Reset multi-factor authentication for user
- `UpdateUserBlockStatus` - Block or unblock user access

**API Standards:**
- Proto-first design with buf code generation
- IAM permission annotations on all operations (`iacuser/users.*`, `iacuser/invites.*`, `iacuser/userRoles.*`)
- Account-scoped operations with `ACCOUNT_ID_INTERNAL` flag
- Consistent error handling with gRPC status codes
- Request correlation and structured response formats

**Data Consistency:**
- MongoDB transactions for multi-document operations
- Eventual consistency with Auth0 synchronization
- Idempotent operations where applicable

## Core Business Functions

**User Lifecycle Management:**
- **User Retrieval:** Query users by ID, email, or search criteria with shallow/deep loading options
- **User Updates:** Modify user details including role changes and project assignments
- **User Removal:** Remove users with admin role protection and cascade handling
- **GUID Generation:** Generate unique user identifiers for new user provisioning
- **Block Management:** Control user access by blocking/unblocking accounts
- **Cross-Account Queries:** Search for user role assignments across multiple accounts

**Invitation Workflow:**
- **Invitation Creation:** Email-based invitation with role and project assignment (`InviteUser`)
- **Token-Based Acceptance:** Accept invitations using secure tokens (`AcceptInvite`)
- **Invitation Retrieval:** Query pending invitations by token, ID, or search criteria
- **Cancellation Management:** Cancel pending invitations with proper cleanup (`CancelInvite`)
- **Duplicate Prevention:** Conflict resolution and validation for existing users
- **Expiration Handling:** Time-based invitation lifecycle management

**Role & Permission Management:**
- **Role Listing:** Retrieve available roles in the system (`GetAvailableRoles`)
- **Role Assignment:** Link users to roles at account and project levels
- **Permission Validation:** IAM service integration for role-based access control
- **Admin Protection:** Prevent unauthorized changes to admin roles
- **Role Queries:** Search and list user role assignments across accounts

**Project Access Control:**
- **Project Listing:** Query projects associated with specific accounts
- **Project Search:** Search projects within account boundaries with filtering
- **Project Assignment:** Link users to specific projects within account context
- **Access Isolation:** Ensure multi-tenant data isolation at project level

**Security & Account Management:**
- **Password Reset:** Initiate password reset workflows through Auth0 (`SendPasswordResetEmail`)
- **MFA Management:** Reset multi-factor authentication for users (`ResetUserMFA`)
- **Account Status:** Block/unblock user access for security purposes
- **Auth0 Synchronization:** Maintain consistency with external identity provider

**Multi-tenant Support:**
- **Account-Scoped Operations:** All operations respect account boundaries for data isolation
- **Cross-Account Prevention:** Enforce restrictions on unauthorized cross-account access
- **Account-Level Filtering:** Search and filter users/invitations within account context

## Configuration

**Environment Configuration Files:**
- [`helm/config/dev.yaml`](helm/config/dev.yaml) - Development environment settings
- [`helm/config/us1tst.yaml`](helm/config/us1tst.yaml) - Testing environment configuration

**Secrets Management:**
- [`helm/devsecrets/`](helm/devsecrets/) - Development secret templates
- [`helm/sealedsecrets/`](helm/sealedsecrets/) - Encrypted production secrets
- Environment variable overrides for all configuration values

**Key Configuration Areas:**
- Auth0 client credentials and database connection settings
- MongoDB connection parameters and authentication
- External service URLs and TLS configuration
- Server ports and runtime behavior settings

## Critical Commands

**Build & Development:**
- `make build` - Compile with formatting and dependency resolution (`go mod tidy`, `go fmt`, build binary)
- `make run` - Build and run with APP_ENV=dev configuration
- `make run-local` - Build and run with `.env` file environment variables
- `make image` - Container image creation for deployment (CGO_ENABLED=0, Linux AMD64)
- `make clean` - Clean build artifacts

**Testing & Quality:**
- `make test` - Execute test suite (`go test ./...`)
- `make code-check` - Full code quality analysis (go vet, golangci-lint with duplication/complexity/cognition, test coverage)
- `make code-check-quick` - Quick critical issues check for CI/CD
- `go test -cover ./...` - Test execution with coverage analysis

**Protocol Buffer Management:**
- `make protos` - Full proto workflow: install tools, verify toolset, clean, fetch dependencies, generate code
- `make fetch` - Fetch proto dependencies from remote GitLab repos (requires GITLAB_TOKEN)
- `make generate` - Generate Go code from proto definitions (`buf generate proto/api`)
- `make update` - Update proto dependency lock file (`buf dep update`)
- `make prune_vendor` - Remove unused vendor proto files
- `make clean_vendor` - Remove entire vendor directory
- `make clean_proto` - Remove generated code from gen directory

**Submodule Management:**
- `git clone --recursive <repo-url>` - Clone with submodules initialized
- `git submodule update --init --recursive` - Initialize submodules after clone
- `cd common && git status` - Check common submodule status
- `cd common && git checkout main && git pull` - Update common to latest main
- `git add common && git commit -m "chore: update common submodule"` - Commit submodule pointer update

**Local Kubernetes Deployment:**
- `minikube start` - Start local Kubernetes cluster
- `eval $(minikube -p minikube docker-env)` - Configure Docker daemon for minikube
- `kubectl create namespace identity` - Create namespace if necessary
- `helm upgrade --install internal-user-management-bff ./helm -n identity --values ./helm/config/dev.yaml` - Deploy to minikube
- `minikube service internal-user-management-bff --url -n identity` - Expose service and get URL

**Secret Management:**
- `kubeseal` - Seal Kubernetes secrets (install via `brew install kubeseal`)
- `sinch plugin install sealed` - Install sinch CLI sealed plugin
- `sinch sealed auth0-client-secret --set client_secret=test --namespace identity --env us1tst > us1tst/auth0-secret.yaml` - Create sealed secret

## Additional Technical Details

**Service Architecture Specifics:**
- **gRPC Service**: `InternalUserService` with 22 methods including user CRUD, invitations, password reset, and MFA management
- **REST Endpoints**: Limited REST API for password reset functionality at `/v1/users/password-reset`
- **Database Collections**: Users and Invitations stored in MongoDB with proper indexing
- **Authentication Flow**: Auth0 JWT validation with service control integration for internal APIs

**Current API Operations:**
- User management: GetUserByID, GetUserByEmail, GetUsers, UpdateUser, RemoveUser
- Search capabilities: SearchUsers, SearchInvites with account-scoped filtering
- Invitation workflow: InviteUser, AcceptInvite, CancelInvite, GetPendingInvites
- Security operations: SendPasswordResetEmail, ResetUserMFA, UpdateUserBlockStatus
- Phone number management: GetUserPhoneMetadata, UpdateUserPhoneNumber
- Role management: GetAvailableRoles with IAM integration
- Account operations: GetProjectsForAccount for project-level access control

**Testing Architecture:**
- Table-driven tests with `testutils.SetupFn` and `testutils.AssertFn` patterns
- Mock implementations in `internal/test/mocks/` for Auth0, IAM, and Account clients
- Test fixtures in `internal/test/fixtures/` for consistent user and invitation data
- Integration tests cover database operations and external service interactions

**Configuration Management:**
- Environment-specific YAML files: `dev.yaml` for development, `us1tst.yaml` for staging
- Sealed secrets via Kubernetes for production credential management
- Auth0 database connection configurable via `AUTH0_DATABASE_CONNECTION`
- Service URLs and TLS configuration for external gRPC services

**Container and Deployment:**
- Alpine-based Docker image (3.18) with single binary deployment
- Multi-stage builds for production optimization (implied by Makefile)
- Health check endpoints for Kubernetes liveness and readiness probes
- Prometheus metrics exposed at `/metrics` for monitoring integration

## Common Submodule Workflow

The `common` submodule contains shared code (IAM client, services, utilities) used by this BFF. Understanding the submodule workflow is critical for proper development and deployment.

### Repository Structure
- **Root Repo:** `internal-user-management-bff` (this repository)
- **Submodule Repo:** `common` (located at `./common/`)
- **Submodule URL:** `git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/common.git`

### Initial Clone
Always clone with `--recursive` to automatically initialize submodules:
```sh
git clone --recursive git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/internal-user-management-bff.git
```

If already cloned without `--recursive`:
```sh
git submodule update --init --recursive
```

### Branch Mirroring Strategy
When working on an assignment (e.g., ID-48), create matching branch names in both repositories:

1. **Root repo branch:** `ID-48_defect_promoted_admin`
2. **Common submodule branch:** `ID-48_defect_promoted_admin` (same name)

This strategy:
- Keeps related changes logically grouped
- Makes it clear which common changes belong to which feature
- Simplifies PR review and dependency tracking

**Creating matching branches:**
```sh
# In root repo
git checkout -b ID-48_defect_promoted_admin

# In common submodule
cd common
git checkout -b ID-48_defect_promoted_admin
cd ..
```

### Working with Common Submodule

**Building and testing common:**
```sh
cd common
make protos test  # Required after any common changes
cd ..
```

**Note:** The common submodule has its own:
- Repository and codebase
- PR and review process
- Build and test requirements (`make protos test`)
- Merge approval workflow

Changes to common must be handled **in isolation** from root repo work.

### PR Dependency Flow

When your assignment requires changes to both common and the root repo, follow this strict workflow:

**Step 1: Common Changes First**
1. Make changes in `common/` submodule on the feature branch
2. Test thoroughly: `cd common && make protos test`
3. Commit and push common branch: `git push origin ID-48_defect_promoted_admin`
4. Create PR for common repository
5. Get PR reviewed and approved
6. **Merge common PR to main**
7. Wait for common main branch to update

**Step 2: Capture Common Main Hash**
```sh
cd common
git checkout main
git pull origin main
git log -1 --format="%H"  # Copy this commit hash
cd ..
```

**Step 3: Update Root to Use Common Main**
```sh
# In root repo
cd common
git checkout main
git pull origin main
cd ..

git add common
git commit -m "chore: update common submodule to main (hash: <commit-hash>)"
```

**Step 4: Finalize Root Repo Work**
1. Ensure all root repo changes are complete
2. Run full test suite: `make clean protos build test`
3. Commit remaining root repo changes
4. Push root repo branch
5. Create PR for root repository
6. Reference common PR in root PR description

**Critical Rules:**
- ✅ Common PRs must be merged to main BEFORE root repo PRs
- ✅ Root repo must always point to common main, never a feature branch
- ❌ Never create a root PR while common is still on a feature branch
- ❌ Never update the submodule pointer to an unmerged common branch

This ensures CI/CD always uses reviewed, stable versions of common code.

### Updating Common After Upstream Changes

When common main has been updated by other developers:

```sh
cd common
git checkout main
git pull origin main
cd ..

git add common
git commit -m "chore: update common submodule to latest main"
git push
```

### Submodule Status Checks

**Check submodule state:**
```sh
git submodule status
```

**Check if submodule has uncommitted changes:**
```sh
cd common && git status
```

**Verify submodule is on correct branch:**
```sh
cd common && git branch --show-current
```

### Common Submodule Best Practices

1. **Always sync before starting work:** Ensure both root and common are up to date with their respective main branches
2. **Test common changes in isolation:** Run `make protos test` in common before integrating with root
3. **Mirror branch names:** Use identical branch names for root and common during feature development
4. **Finalize common first:** Complete, review, and merge all common changes before finalizing root changes
5. **Document dependencies:** In PRs, clearly state which common PR (if any) must merge first
6. **Never skip validation:** Always run `make clean protos build test` after updating common
7. **Commit submodule updates separately:** Use dedicated commits for submodule pointer updates

## Deployment

Staging and production deployments are handled via GitLab CI/CD. Deployments are triggered automatically or via manual approval on the **Merge Request (MR) page** for the relevant work. See the `.gitlab-ci.yml` in the repository for pipeline details.
