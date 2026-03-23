# Gozer

**Repo:** git@github.com:mailgun/gozer.git  
**Last Updated:** 2026-02-24

---

## Table of Contents
- [LLM Agent Preamble](#llm-agent-preamble)
- [Prerequisites](#prerequisites)
- [Project Overview](#project-overview)
- [Classification of Assignment](#classification-of-assignment)
- [System Architecture](#system-architecture)
- [Key Components](#key-components)
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
- [General Guidance for AI Agents](#general-guidance-for-ai-agents)

---

## LLM Agent Preamble

**CRITICAL REQUIREMENTS:**
- **READ AND ADHERE** → This file MUST be read and followed for ALL units of work and ALL editing sessions
- **REPOSITORY VALIDATION** → Review this document to ensure it accurately describes the repository you are working in
- **MISMATCH NOTIFICATION** → If this document does not match the actual repository structure, dependencies, or purpose, IMMEDIATELY notify the user of the discrepancy

**AGENT RESPONSIBILITY:** Before beginning any work, validate that this guide matches the current repository context and report any inconsistencies to the user.

---

## Prerequisites

**Required Software:**
- Go 1.23+ (go.mod specifies 1.23.0, toolchain 1.23.11)
- MongoDB 4+
- Docker and Docker Compose (for local MongoDB)
- Make utility for build automation
- golangci-lint for code quality checks
- Git for version control

**Required Access:**
- GitHub access to mailgun/gozer repository
- Auth0 staging credentials (from Vault)
- Honeycomb API key for tracing (optional for local dev)

**Note:** If prerequisites are missing, builds and tests will fail with clear error messages.

---

## Project Overview

Gozer is Mailgun's authentication and domain management service that provides secure user authentication, session management, and domain control capabilities. It serves as the central authentication hub for Mailgun's ecosystem, integrating with Auth0 for identity management and providing APIs for user management, domain verification, SAML authentication, and multi-factor authentication.

**Purpose:** Central authentication and domain management service for Mailgun's platform

**Business Context:** Critical authentication infrastructure serving multiple Mailgun portals and dashboards including Sinch Dashboard, Mailgun Dashboard, and Engage Dashboard.

---

## Classification of Assignment

**Repository URL:** https://github.com/mailgun/gozer  
**Classification:** Backend Microservice / Backend-for-Frontend (BFF)

---

## System Architecture

**Architecture Pattern:** Layered microservice architecture built on Mailgun's scaffold framework

**Core Components:**
- **API Layer:** RESTful HTTP endpoints exposed via scaffold framework
- **Service Layer:** Business logic orchestration in api/ package
- **Repository Layer:** Data access through model/ package
- **Database Layer:** MongoDB persistence with connection pooling
- **External Integrations:** Auth0 (identity), Jefe (user management), OpenTelemetry (tracing)

**Technology Stack:**
- **Language:** Go 1.23.0 (toolchain 1.23.11)
- **Framework:** Mailgun scaffold framework for service structure
- **Database:** MongoDB for user, session, and domain data
- **Authentication:** Auth0 with JWT token validation and OAuth2 flows
- **Tracing:** OpenTelemetry with Honeycomb backend
- **Session Management:** Custom secure cookie-based sessions with Gorilla toolkit

**Data Flow:**
1. HTTP requests received via scaffold HTTP server
2. Middleware validates Auth0 JWT tokens and sessions
3. API handlers delegate to business logic
4. Model layer interfaces with MongoDB for persistence
5. Sessions stored in separate MongoDB database
6. OpenTelemetry spans track request lifecycle
7. Responses returned with appropriate status codes and headers

**Key Relationships:**
- Integrates with Jefe service for user management operations
- Auth0 serves as identity provider for authentication
- Multiple portal applications (SMA, OMA) rely on Gozer for auth
- External dashboards (Sinch, Mailgun, Engage) use Gozer for OIDC login

---

## Key Components

### API Modules
- **ManagerAPI**: Core user and authentication management operations
- **PortalAPI**: Portal session management and OAuth2 login flows for SMA/OMA portals
- **MfaAPI**: Multi-factor authentication enrollment and challenge operations
- **SamlAPI**: SAML-based authentication integration for enterprise SSO
- **DomainControlAPI**: Domain management and control operations for DNS and tenant domains
- **DomainVerificationAPI**: Domain ownership verification workflows
- **BlockingAPI**: User and domain blocking functionality for security and compliance

### Core Packages
- [`auth/`](auth/): Authentication logic, JWT validation, and unknown user handling
- [`sessions/`](sessions/): Session management with secure cookies and MongoDB persistence
- [`domaincontrol/`](domaincontrol/): Domain control business logic and validation
- [`domainverification/`](domainverification/): Domain ownership verification workflows
- [`model/`](model/): Data models and MongoDB repository interfaces (User, Domain, Session, Verification, Blocking)
- [`mongodb/`](mongodb/): Database connection management, indexing, and operations
- [`config/`](config/): Service configuration loading and validation
- [`logging/`](logging/): Structured logging utilities with Auth0 integration
- [`manager/`](manager/): Business logic coordination between services
- [`activity/`](activity/): Activity tracking and auditing
- [`apps/`](apps/): Application-specific integrations
- [`fixtures/`](fixtures/): Test data and fixtures for testing
- [`generics/`](generics/): Generic utility functions and helpers

---

## Development Environment

**Setup Steps:**

1. **Start MongoDB:**
   ```bash
   docker-compose up -d
   # OR manually:
   docker run -d -p 27017:27017 --name gozer-mongo mongo:4.2.1
   ```

2. **Copy Configuration:**
   ```bash
   cp config/dev_template.yaml config/dev.yaml
   ```

3. **Configure Secrets:**
   Edit `config/dev.yaml` and update the `_SECRETS` section:
   - SMA portal client_id and client_secret (copy from staging Vault)
   - OMA portal client_id and client_secret (copy from staging Vault)
   - Other values can use template defaults for local development

4. **Install Dependencies:**
   ```bash
   go mod download
   ```

5. **Verify Setup:**
   ```bash
   # Verify Go version
   go version  # Should be 1.23+
   
   # Verify MongoDB is running
   docker ps | grep mongo
   
   # Verify configuration
   cat config/dev.yaml
   ```

**Environment Variables (Optional for Tracing):**
```bash
export MG_ENV=dev
export OTEL_EXPORTER_OTLP_ENDPOINT=https://api.honeycomb.io
export OTEL_EXPORTER_OTLP_HEADERS="x-honeycomb-team=<API_KEY>"
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
export OTEL_TRACES_EXPORTER=otlp
export OTEL_TRACES_SAMPLER=parentbased_traceidratio
export OTEL_TRACES_SAMPLER_ARG=0.1  # Sample 10% of requests
```

**Configuration Files:**
- [`config/dev_template.yaml`](config/dev_template.yaml): Template for local development configuration
- `config/dev.yaml`: Local development configuration (git-ignored, copy from template)
- Production configurations managed via Nomad job deployment

**Database Requirements:**
- **Primary Database:** `mg_dev` (users, domains, verifications, blocking)
- **Session Database:** `sessions` (user sessions)
- Both databases run on MongoDB at `mongodb://127.0.0.1:27017/`

**No Containerization for Local Dev:** Service runs directly via `go run` or `make run` for development

---

## Build & Deployment

**Build Commands:**
```bash
# Build binary
go build -o gozer cmd/gozer/main.go

# Run locally with dev config
make run

# Run tests
make test

# Run linting
make lint
```

**Testing:**
```bash
# Run all tests with race detection
make test
# Equivalent to: MallocNanoZone=0 go test -v -race -timeout 90s -count 1 -p 1 ./...

# Run tests with coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html
open coverage.html

# Run specific package tests
go test -v ./api/...
go test -v ./sessions/...
```

**Deployment:**
- **Platform:** Nomad orchestration
- **Container:** Docker image pushed to ghcr.io/mailgun/gozer
- **Configuration:** Nomad job file at [`.director/job.nomad`](.director/job.nomad)
- **Secrets:** Pulled from Vault at runtime
- **Update Strategy:** Rolling updates with auto-revert (max_parallel: 4, healthy_deadline: 90s)
- **Health Checks:** Service registration via Consul with health endpoint monitoring

**CI/CD:**
- GitHub Actions for CI/CD pipeline
- Automated testing on pull requests
- Container builds on merge to main
- Deployment triggered via Nomad job updates

---

## Project Structure

**Directory and File Roles:**
- [`cmd/gozer/`](cmd/gozer/): Application entry point and main function
- [`api/`](api/): HTTP API handlers for all endpoints
- [`auth/`](auth/): Authentication logic and JWT validation
- [`sessions/`](sessions/): Session management and secure cookies
- [`domaincontrol/`](domaincontrol/): Domain control business logic
- [`domainverification/`](domainverification/): Domain verification workflows
- [`model/`](model/): Data models and MongoDB repositories
- [`mongodb/`](mongodb/): Database connection and operations
- [`config/`](config/): Configuration files and loading
- [`logging/`](logging/): Logging utilities
- [`manager/`](manager/): Business logic orchestration
- [`activity/`](activity/): Activity tracking
- [`apps/`](apps/): Application-specific integrations
- [`fixtures/`](fixtures/): Test data and fixtures
- [`generics/`](generics/): Generic utility functions
- [`.director/`](.director/): Nomad deployment configuration
- [`.github/`](.github/): GitHub Actions workflows
- [`Makefile`](Makefile): Build automation targets
- [`docker-compose.yaml`](docker-compose.yaml): Local MongoDB setup

---

## Key Files

**Entry Point:**
- [`cmd/gozer/main.go`](cmd/gozer/main.go): Application bootstrap and initialization

**Core Service:**
- [`service.go`](service.go): Service structure, lifecycle, and API initialization

**API Implementations:**
- [`api/manager.go`](api/manager.go): Core management API endpoints
- [`api/portal.go`](api/portal.go): Portal authentication flows
- [`api/mfa.go`](api/mfa.go): MFA operations
- [`api/saml.go`](api/saml.go): SAML authentication
- [`api/domain_control.go`](api/domain_control.go): Domain control endpoints
- [`api/domain_verification.go`](api/domain_verification.go): Domain verification endpoints
- [`api/blocking.go`](api/blocking.go): Blocking operations
- [`api/middleware.go`](api/middleware.go): Authentication middleware
- [`api/constants.go`](api/constants.go): API constants and definitions
- [`api/docs.go`](api/docs.go): API documentation

**Configuration:**
- [`config/config.go`](config/config.go): Configuration structures and loading
- [`config/dev_template.yaml`](config/dev_template.yaml): Development configuration template

**Database:**
- [`mongodb/mongodb.go`](mongodb/mongodb.go): MongoDB connection and operations

**Deployment:**
- [`.director/job.nomad`](.director/job.nomad): Nomad job specification
- [`Makefile`](Makefile): Build and test automation
- [`docker-compose.yaml`](docker-compose.yaml): Local MongoDB setup
- [`README.md`](README.md): Project documentation
-[`QUICKSTART-DEV.md`](QUICKSTART-DEV.md): Quick start guide for developers

---

## Testing Strategy

**Testing Philosophy:** Comprehensive test coverage with unit tests alongside implementation files

**Test Organization:**
- Test files co-located with source: `*_test.go` alongside implementation
- Table-driven tests using testify/assert
- Mock external dependencies (Auth0, Jefe)
- Use real MongoDB for integration-style tests

**Test Coverage Areas:**
- All API endpoints have corresponding test files
- Session management logic tested thoroughly
- Domain control and verification flows covered
- Authentication and middleware tested
- Model layer with MongoDB operations tested

**Coverage Requirements:**
- **New Features:** >90% coverage
- **Defect Fixes:** 100% coverage
- Run `go test -cover ./...` to verify

**Test Execution:**
```bash
# Run all tests with race detection
make test

# Run specific package tests
go test -v ./api/...
go test -v ./sessions/...

# Run with coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html
```

**Test Configuration:**
- Tests use `MallocNanoZone=0` flag for macOS compatibility
- Race detection enabled (`-race` flag)
- Timeout set to 90 seconds
- Sequential execution (`-p 1`) for database tests
- Single iteration (`-count 1`) to avoid caching issues

---

## Security & Compliance

**Authentication:**
- Auth0 JWT token validation for all authenticated endpoints
- OAuth2 authorization code flow for portal logins
- SAML authentication for enterprise SSO
- Secure session cookies with encryption and signing

**Authorization:**
- Role-based access control via Auth0 tokens
- API key authentication for service-to-service calls
- Middleware enforces authentication before handler execution

**Data Protection:**
- Secure cookie encryption using authenticated encryption (AEAD)
- Hash and block keys for cookie signing
- TLS certificates mounted from shared volume
- MongoDB credentials stored in Vault

**Security Standards:**
- No hardcoded secrets (all from Vault or config)
- Input validation on all API endpoints
- CORS configuration for portal origins
- Session expiration and rotation
- MFA support for enhanced security

**Compliance:**
- Audit logging for security-relevant operations
- User blocking capabilities for compliance
- Domain verification for ownership proof

---

## Monitoring & Operations

**Logging:**
- Structured logging via logrus
- Context-aware logging with request IDs
- Auth0 integration logging for debugging
- Log level configurable per environment (debug for dev)

**Tracing:**
- OpenTelemetry instrumentation throughout codebase
- Spans for all API requests and database operations
- Export to Honeycomb for distributed tracing
- Configurable sampling rate (default 100% for dev, lower for prod)
- Trace configuration via environment variables

**Metrics:**
- Prometheus metrics exposed at `/metrics` endpoint
- HTTP request metrics (count, duration, status codes)
- MongoDB operation metrics
- Service health indicators

**Health Checks:**
- Metrics endpoint doubles as health check
- Consul service registration with health monitoring
- Liveness and readiness probes in Nomad

**Troubleshooting:**
- Check logs for authentication failures
- Verify MongoDB connectivity
- Confirm Auth0 configuration and secrets
- Test with curl: `curl http://localhost:4001/metrics`
- Use Honeycomb traces for request debugging

---

## Technical Stack & Dependencies

**Runtime:**
- Go 1.23.0 (toolchain 1.23.11)
- MongoDB 4+

**Core Framework:**
- mailgun/scaffold v1.76.0: Service framework and common utilities
- mailgun/holster/v4 v4.16.4: Utility library
- mailgun/jefe v1.26.0: User management client

**HTTP & Routing:**
- gorilla/mux v1.8.1: HTTP routing
- gorilla/securecookie v1.1.2: Secure cookie handling

**Authentication:**
- auth0/go-auth0 v1.26.0: Auth0 API client
- golang-jwt/jwt/v5 v5.2.1: JWT token handling
- coreos/go-oidc/v3 v3.9.0: OIDC client
- MicahParks/keyfunc/v2 v2.1.0: JWT key management
- golang.org/x/oauth2 v0.30.0: OAuth2 client

**Database:**
- go.mongodb.org/mongo-driver v1.14.0: MongoDB client with connection pooling

**Observability:**
- go.opentelemetry.io/otel v1.24.0: OpenTelemetry SDK
- go.opentelemetry.io/otel/trace v1.24.0: Tracing API
- prometheus/client_golang v1.13.0: Prometheus metrics
- sirupsen/logrus v1.9.3: Structured logging

**Testing:**
- stretchr/testify v1.10.0: Test assertions and mocking

**Utilities:**
- google/uuid v1.6.0: UUID generation
- pkg/errors v0.9.1: Error handling with stack traces

---

## Service Contract

**Primary Service:** Authentication and domain management for Mailgun platform

**Key Operations:**
- User authentication via Auth0 login flows
- Portal session management with secure cookies
- OAuth2 authorization code flow for SMA/OMA portals
- SAML authentication for enterprise SSO
- MFA enrollment and challenge flows
- Domain ownership verification workflows
- Domain control and management operations
- User and domain blocking for security

**API Standards:**
- RESTful HTTP endpoints using Gorilla mux
- JSON request/response bodies
- JWT bearer token authentication
- Secure cookie-based sessions
- Prometheus metrics at `/metrics`
- Standard HTTP status codes

**Integration Points:**
- Auth0: Identity provider and JWT issuer
- Jefe: User management operations
- MongoDB: Data persistence
- Portal Applications: SMA, OMA portals
- External Dashboards: Sinch, Mailgun, Engage

---

## Core Business Functions

**User Authentication:**
- Auth0-based login flows with JWT tokens
- OAuth2 authorization code flow for portals
- Unknown user handling and registration
- Session creation and management

**Portal Integration:**
- SMA (Sinch Marketing Automation) portal authentication
- OMA portal authentication
- Callback handling for OAuth2 flows
- Portal URL redirection after authentication

**Multi-Factor Authentication:**
- MFA enrollment for users
- MFA challenge and verification
- Integration with Auth0 MFA capabilities

**SAML Authentication:**
- SAML SSO for enterprise customers
- SAML request processing
- SAML response validation
- Certificate management for SAML

**Domain Management:**
- Domain ownership verification via DNS records
- Domain control assignment to users/accounts
- Domain verification status tracking
- DNS record validation

**User Blocking:**
- Administrative blocking of users
- Domain-level blocking
- Block status tracking and enforcement

**Session Management:**
- Secure cookie-based sessions
- Session persistence in MongoDB
- Session expiration and rotation
- Cross-portal session sharing

---

## Configuration

**Environment Configuration Files:**
- [`config/dev_template.yaml`](config/dev_template.yaml): Template for local development (checked into git)
- `config/dev.yaml`: Local development configuration (git-ignored, created from template)
- Production configurations managed via Nomad job and Vault

**Key Configuration Sections:**

**Service Settings:**
- HTTP port (default: 4001)
- Public and protected API hosts
- Feature flags (e.g., enable_dual_auth)

**Database Configuration:**
- MongoDB URI for main database
- MongoDB URI for sessions database

**Portal Configuration:**
- SMA portal: audience, domain, callback URL, portal URL
- OMA portal: audience, domain, callback URL, portal URL

**Application Integrations:**
- Sinch Dashboard: URL and client ID
- Mailgun Dashboard: URL and client ID
- Engage Dashboard: URL and client ID

**Secrets Management:**
- Development: `_SECRETS` section in dev.yaml (git-ignored)
- Production: Vault integration via Nomad templates
- Required secrets: portal client IDs/secrets, API keys, cookie encryption keys

**Environment Variables:**
- `MG_ENV`: Environment name (dev, staging, prod)
- OpenTelemetry tracing configuration (OTEL_*)
- Set in job.nomad for production

---

## Critical Commands

**Build & Development:**
```bash
# Run service locally
make run

# Build binary
go build -o gozer cmd/gozer/main.go

# Install dependencies
go mod download

# Setup configuration
cp config/dev_template.yaml config/dev.yaml

# Start MongoDB
docker-compose up -d
```

**Testing & Quality:**
```bash
# Run all tests with race detection
make test

# Run tests with coverage
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out -o coverage.html

# Run linting
make lint

# Run specific package tests
go test -v ./api/...
go test -v ./sessions/...
```

**Docker & Database:**
```bash
# Start MongoDB via compose
docker-compose up -d

# Start MongoDB manually
docker run -d -p 27017:27017 --name gozer-mongo mongo:4.2.1

# Stop MongoDB
docker-compose down

# View MongoDB logs
docker logs gozer-mongo
```

**Verification:**
```bash
# Test service health
curl http://localhost:4001/metrics

# Test specific metrics
curl http://localhost:4001/metrics | grep gozer

# Verify Go version
go version

# Verify Docker status
docker ps
```

---

## General Guidance for AI Agents

### Code Standards and Architecture

---

## General Guidance for AI Agents

### Code Standards and Architecture
- Follow existing abstraction layers and maintain the established separation of concerns
- Adhere to Go language coding standards and conventions throughout the implementation
- Use the coding style found in each file; avoid deviations from established patterns
- Preserve existing functions, methods, procedures, and unit tests unless explicitly required for the task
- Follow Mailgun scaffold framework patterns for service structure
- Maintain layered architecture: API → Manager → Model → MongoDB

### Dependencies and Libraries
- Use current library versions from go.mod; avoid downgrading
- Only use official, well-maintained, and highly-rated libraries for new solutions
- Maintain consistency with existing dependency patterns
- Leverage Mailgun's internal libraries (scaffold, holster) when applicable
- Check for local replacements in go.mod before adding external dependencies

### Error Handling and Security
- Apply consistent error handling strategies as established in the codebase
- Wrap errors with context using pkg/errors for stack traces
- Follow security best practices for authentication and domain management systems
- Never log sensitive data (passwords, tokens, API keys, PII)
- Implement performance-conscious solutions for high-traffic endpoints
- Validate all inputs at API boundaries
- **Do not document secrets or credentials** in checked-in files (README.md, AGENT.md)
- For secrets documentation, create AGENT-ASSIGNMENT-RESOURCES.md (git-ignored) and mark "DO NOT SHARE" at top

### Concurrency and Race Condition Prevention
- Implement resource-specific synchronization to avoid global blocking where possible
- Use `sync.Map.LoadOrStore()` atomically rather than separate `Load()` + conditional `LoadOrStore()` patterns
- Apply the "correctness first, optimization second" principle: solve race conditions first, then optimize
- When translating patterns between synchronization primitives, rethink the approach rather than literal translation
- Test race conditions with concurrent requests using `go test -race`
- MongoDB operations should use appropriate locking and transactions where needed

### Testing Requirements
- Write tests alongside implementation (`*_test.go` files)
- Use table-driven tests with testify assertions
- Mock external dependencies (Auth0, Jefe, external APIs)
- Use real MongoDB for integration-style tests
- **Coverage Standards:** >90% for new features, 100% for defect fixes
- Run `make test` before committing to verify all tests pass with race detection
- Test edge cases, error conditions, and happy paths

### Code Quality
- Write specific, accurate, and high-value comments; avoid unnecessary or low-value commentary
- Make changes only to files directly related to your assigned task
- Maintain code clarity and readability
- Use descriptive variable and function names
- Conduct post-implementation optimization passes to identify redundant patterns
- Run `make lint` before committing to catch code quality issues

### Communication Guidelines
- Ask questions when requirements or implementation details are unclear
- Notify when additional information is needed rather than making assumptions
- Do not create solutions without proper clarification when facing ambiguity
- Document architectural decisions in assignment.md
- Provide clear, concise commit messages

### Safety Constraints
- **`rm` commands CANNOT be auto-accepted** → Always require explicit user confirmation for file deletion
- **Destructive operations** → Any command that deletes, removes, or irreversibly modifies files must be user-approved
- **Code Quality Gates** → All code must pass `make lint` and `make test` with no critical issues

## Deployment

### Staging

After a GitHub PR is created, go to Slack `#eng-director-dev`:

1. Check current version:
   ```
   @Director-Develop current version svc=gozer
   ```
   Result: creates a thread with current deployed version info.

2. In the thread, deploy:
   ```
   @Director-Develop deploy svc=gozer tag=PR<NUMBER> env=mfstaging region=us-east4
   ```
   - `tag`: GitHub PR number in `PR###` format (e.g. `PR118`)
   - On success: is-healthy message shown in thread
   - On failure: evaluate whether this is an infrastructure or service failure
   - Rollback: TBD

### Production

After the GitHub PR is merged, go to Slack `#chatops`:

1. Check current version:
   ```
   @Director current version svc=gozer
   ```

2. Deploy to each region **one at a time** in order:
   ```
   @Director deploy svc=gozer tag=PR<NUMBER> env=mfproduction region=us-east4
   @Director deploy svc=gozer tag=PR<NUMBER> env=mfproduction region=us-west1
   @Director deploy svc=gozer tag=PR<NUMBER> env=mfproduction region=europe-west1
   ```
   - On success: is-healthy message shown in thread
   - On failure: evaluate whether this is an infrastructure or service failure
   - Rollback: TBD
