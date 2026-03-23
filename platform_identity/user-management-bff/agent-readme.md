# User Management BFF - AGENT README

**Repo:** git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/user-management-bff.git  
**Last Updated:** 2026-03-20

---

## Table of Contents
- [AI Agent Implementation Guidelines](#ai-agent-implementation-guidelines)
- [Consistency and Pattern Adherence](#consistency-and-pattern-adherence)
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
- [Proto Management](#proto-management)
- [Logging Strategy](#logging-strategy)

---

## AI Agent Implementation Guidelines

### Consistency and Pattern Adherence

All new features and changes must follow the existing patterns and implementation styles in this repository. Review the current codebase for conventions and structure, and match these in your work unless a justified, documented exception is required.

### Critical Requirements
- **Read and adhere** to this file for all units of work and editing sessions.
- **Repository validation:** Ensure this document matches the actual repository structure and purpose. Notify the user of any discrepancies.

### Focus Areas
1. Service Layer: Primary business logic location
2. gRPC & REST Handlers: API endpoint implementations
3. MongoDB Models: Data persistence patterns (users and invitations)
4. Auth0 Integration: Authentication flows and user management
5. IAM Integration: Role-based access control and authorization
6. Email Workflows: Mailgun integration for invitation emails
7. Token Management: MSI bearer tokens for service-to-service communication

### Security Requirements
- No hardcoded secrets in any checked-in files
- Input validation on all API endpoints (email format, GUID format, required fields)
- Auth0 JWT validation for authenticated endpoints via interceptors
- Service Control validation for internal service-to-service calls
- Secure email templates for invite notifications
- MongoDB injection prevention in queries
- PII hashing in all logs (use `logging.Hash()` for emails, user IDs)

### Architectural Patterns
- Dependency Injection: Constructor-based DI throughout (main.go orchestrates all initialization)
- Interface Segregation: Use established service interfaces (UserService, InviteService, RoleService, AccountService)
- Error Wrapping: Consistent error handling with context using dedicated error translation functions
- Logging: Structured logging with correlation IDs propagated via context
- Testing: Mock external dependencies (Auth0, IAM, Account, Email), test business logic with table-driven tests
- Code Quality: Static analysis with golangci-lint enforced in build process

### Change Implementation Strategy
1. Analyze existing patterns in target service/handler (examine similar operations first)
2. Follow established error handling conventions (use TranslateModelError, TranslateAuth0Error, TranslateIAMError)
3. Maintain interface compatibility with existing consumers (proto changes require coordination)
4. Add comprehensive tests alongside implementation (>90% unit coverage for new features, 100% for defects)
5. Update proto definitions if new gRPC methods added (use buf for generation)
6. Ensure correlation ID propagation in all service calls
7. Validate input at handler layer before delegating to services

### Testing Ecosystem
- Unit tests: `go test ./...` (>90% coverage for new features, 100% for defects)
- Table-driven tests: Use testify/suite with SetupSubTest patterns
- Mock generation: Use gomock for external client mocks
- Integration tests: Test MongoDB operations with testcontainers (future)
- Test coverage: `go test -cover ./...` to measure coverage
- Test utilities: Use `internal/test/testutils` for common patterns

### Performance and Error Handling
- MongoDB indexing: Unique index on users.email, index on invites.invitee_token
- MongoDB connection pooling: Configured with timeout settings
- Prometheus metrics: Request duration, count, and error rates for gRPC and REST
- Service errors wrapped with context using pkg/errors patterns
- gRPC status codes mapped from business errors via ErrAsGRPC helper
- REST status codes mapped from business errors via ErrAsREST helper
- Structured logs with correlation IDs for request tracing
- Input validation at API boundaries before service calls

### Deployment & Build Integration
- Make-based build system (see Makefile)
- Docker support: `make image` for containerized builds (Alpine 3.18 base)
- Proto generation: Automated with buf (`make protos`)
- CGO_ENABLED=0 for static binary linking
- Multi-stage build pattern for optimized container size

### Safety Constraints
- rm commands cannot be auto-accepted; require explicit user confirmation
- Destructive operations must be user-approved (delete user, cancel invite, etc.)
- All code must pass code quality checks (go vet, golangci-lint) with no critical issues
- Proto changes must be coordinated with consumers

## Logging and Commit Practices

- When logging, PII (personally identifiable information) such as email addresses and phone numbers must be hashed before logging. Never log raw PII. Use `logging.Hash("email_hash", email)` instead of `logging.Field("email", email)`.
- For automated git commits, always use very terse commit messages (e.g., `feat: add X`, `fix: Y`, `chore: Z`).

## Deployment

Staging and production deployments are handled via GitLab CI/CD. Deployments are triggered automatically or via manual approval on the **Merge Request (MR) page** for the relevant work. See the `.gitlab-ci.yml` in the repository for pipeline details.

---

## Project Overview

The **user-management-bff** is a Backend For Frontend microservice for the Sinch Beehive identity platform. It provides user and invite management functionality for the account dashboard UI and admin tools:

- User CRUD operations (create, read, update, delete)
- User invite workflows with pending/accepted states
- Role-based access control via IAM
- Project assignment for users
- Auth0 integration for authentication and GUID management
- Email delivery for invitations via Mailgun

---

## System Architecture

Layered architecture with two server interfaces:

- **REST Layer** (Gin, port 8080): Auth0 callback flows, admin endpoints, health/metrics
- **gRPC Layer** (port 9090): Primary service interface with proto-defined contracts
- **Service Layer** (`common/pkg/services/`): Business logic — UserService, InviteService, RoleService, AccountService
- **Data Layer**: MongoDB repositories for Users and Invites collections
- **External Integrations**:
  - Auth0 — user authentication and GUID management
  - IAM Build Adapter (gRPC) — authorization and role resolution
  - Account Service (gRPC) — account/project operations
  - Service Control Service (gRPC) — service-to-service validation
  - Mailgun — email delivery for invite workflows
- **Infrastructure**: Kubernetes/Istio with sealed secrets, network policies, and service accounts

---

## Development Environment

- **Go Version**: 1.25.4
- **Required Tools**: `buf`, `protoc-gen-go` v1.33.0, `protoc-gen-go-grpc` v1.3.0, `docker`, `kubectl`, `helm`, `kubeseal`
- **Local Config**: `helm/config/dev.yaml`
- **Environment Variables**: `.env` file (see `.env.example`)
- **Local MongoDB**: `localhost:27017`
- **Auth0 Credentials**: CLIENT_ID, CLIENT_SECRET, AUDIENCE, DOMAIN (from sealed secrets or `.env`)

---

## Build & Deployment

| Target | Description |
|--------|-------------|
| `make build` | Compile binary for current OS |
| `make image` | Cross-compile for Linux + build Docker image |
| `make run` | Build and run locally with dev config |
| `make run-local` | Run with `.env` file overrides |
| `make test` | Run all tests (`go test ./...`) |
| `make protos` | Full protobuf workflow (install tools, generate, fetch vendor deps) |
| `make generate` | Generate protobuf code only |
| `make clean` | Clean build artifacts |

**Docker**: Multi-stage Alpine 3.18 build, CGO_ENABLED=0 for static binary.

**CI/CD** (`.gitlab-ci.yml`): verify → build → test → publish → deploy. Publishes to Nexus registry. Deploys to us1tst (staging) and us1 (production) via Helm with manual approval triggers.

---

## Project Structure

```
user-management-bff/
├── cmd/server/main.go              # Entry point — initializes all services, servers, graceful shutdown
├── internal/
│   ├── api/handlers/
│   │   ├── router.go               # Gin router setup (public health + protected /auth0 routes)
│   │   ├── rest_handler.go         # REST endpoint handlers
│   │   ├── grpc_handler.go         # gRPC service implementation
│   │   ├── middleware.go           # Auth middleware (Basic Auth for REST)
│   │   ├── errors.go              # Error translation to gRPC/REST codes
│   │   └── translate.go           # Proto ↔ service model translation
│   └── config/config.go           # Config loading (YAML + env var merge)
├── proto/api/user_service.proto    # gRPC service definition
├── gen/go/                         # Generated proto code (this service)
├── common/                         # Shared library (git submodule)
│   ├── pkg/services/              # Business logic implementations
│   ├── pkg/clients/               # gRPC clients (auth0, iam, account, servicecontrol)
│   ├── pkg/auth/                  # Authentication validators
│   ├── pkg/logging/               # Structured logging with correlation IDs
│   ├── pkg/metrics/               # Prometheus metrics
│   ├── pkg/mongodb/               # MongoDB utilities and repositories
│   ├── proto/sinchid/rpc/v1/      # Custom annotations (IAM context, resource defs)
│   └── gen/go/                    # Generated gRPC code (shared)
├── helm/
│   ├── templates/
│   │   ├── api-pub.yaml           # Istio VirtualService — public API (CORS config)
│   │   ├── api-int.yaml           # Istio VirtualService — private API (CORS config)
│   │   ├── grpc-web.yaml          # EnvoyFilter for gRPC-Web support
│   │   ├── deployment.yaml        # Kubernetes Deployment
│   │   ├── service.yaml           # Kubernetes Service (ClusterIP)
│   │   ├── app-config.yaml        # ConfigMap
│   │   └── sealed-secrets.yaml    # Sealed secrets management
│   ├── values.yaml                # Default Helm values
│   └── config/
│       ├── dev.yaml               # Local development
│       ├── us1tst.yaml            # Staging environment
│       └── us1.yaml               # Production environment
├── Makefile                       # Build automation
├── Dockerfile                     # Container image definition
└── .gitlab-ci.yml                 # CI/CD pipeline
```

---

## Key Files

| File | Why It Matters |
|------|----------------|
| `cmd/server/main.go` | All service wiring and initialization |
| `internal/config/config.go` | Configuration loading (YAML + env var precedence) |
| `internal/api/handlers/router.go` | Route definitions — public health + protected /auth0 |
| `proto/api/user_service.proto` | gRPC contract — 15+ RPC methods |
| `helm/templates/api-pub.yaml` | Istio VirtualService — public CORS config |
| `helm/templates/api-int.yaml` | Istio VirtualService — private CORS config |
| `helm/config/us1tst.yaml` | Staging CORS allowOrigins + env config |
| `helm/config/us1.yaml` | Production CORS allowOrigins + env config |
| `common/pkg/services/` | Business logic (UserService, InviteService, etc.) |

---

## Testing Strategy

- **Unit tests**: `go test ./...` (>90% coverage for new features, 100% for defects)
- **Table-driven tests**: testify/suite with SetupSubTest patterns
- **Mock generation**: gomock for external client mocks
- **Test files**: `internal/api/handlers/translate_test.go`, `common/pkg/services/model/user_mongo_test.go`, `common/pkg/services/model/invite_mongo_test.go`, `common/pkg/services/role_test.go`
- **CI**: Docker-based MongoDB 5.0 service for integration tests in pipeline

---

## Security & Compliance

- **gRPC Auth**: Auth0 JWT validation via `auth.Interceptor()` (Auth0Validator + ServiceControlValidator)
- **REST Auth**: Basic Auth (username: `auth0_flow_user`, password from REST_API_KEY secret)
- **Authorization**: IAM context annotations on gRPC methods; permissions: `account:user_management:read`, `account:user_management:write`
- **Secrets**: Sealed secrets in `helm/sealedsecrets/{us1,us1tst}/` — sealed per-namespace and per-cluster
- **PII**: Email hashing in logs via `logging.Hash("email_hash", email)` — never log raw PII
- **Service Accounts**: Kubernetes workload identity with projected token volumes (3600s expiration)

---

## Monitoring & Operations

- **Liveness**: `GET /health/liveness` — simple 200 response
- **Readiness**: `GET /health/readiness` — checks MongoDB and external service connectivity
- **Metrics**: Prometheus on `GET /metrics` (port 8080); gRPC and REST request duration, count, error rates
- **Pod annotations**: `prometheus.io/scrape: "true"`, port 8080, path `/metrics`
- **Logging**: Structured (zap) with correlation IDs for request tracing
- **Shutdown**: Graceful with 5-second timeout

---

## Technical Stack & Dependencies

| Dependency | Version | Purpose |
|------------|---------|---------|
| `gin-gonic/gin` | v1.11.0 | HTTP framework |
| `google.golang.org/grpc` | v1.77.0 | gRPC framework |
| `go.mongodb.org/mongo-driver` | v1.17.6 | MongoDB client |
| `auth0/go-auth0` | v1.32.0 | Auth0 SDK |
| `golang-jwt/jwt/v5` | v5.3.0 | JWT validation |
| `prometheus/client_golang` | v1.23.2 | Metrics |
| `go.uber.org/zap` | v1.27.0 | Structured logging |
| `mailgun/mailgun-go/v5` | v5.8.1 | Email delivery |
| `stretchr/testify` | v1.11.1 | Testing |

---

## Service Contract

**REST Endpoints** (port 8080):

- `POST /auth0/v1/users/guid` — generate user GUID from email
- `GET /auth0/v1/users/guid/:email` — get GUID by email
- `POST /auth0/v1/invites/:inviteId/accept` — accept invite
- `GET /auth0/v1/invites/pending` — get pending invites
- `GET /auth0/v1/invites/by-token` — get invite by token
- `POST /auth0/admin/generate-guids` — batch GUID generation (admin)

**gRPC Methods** (`userpb.UserService`, port 9090):

GetUserByID, GetUserByEmail, GetUsers, SearchUsers, UpdateUser, InviteUser, RemoveUser, GetPendingUserInvite, GetPendingInvites, SearchInvites, AcceptInvite, AcceptInviteByToken, CancelInvite, GenerateNewUserGUID, GetAvailableRoles, GetInviteByToken, GetProjectsForAccount

---

## Core Business Functions

- **UserService**: CreateUser, GetUserByID/Email, GetUsers, UpdateUser, SearchUsers, GetUserRole
- **InviteService**: InviteUser, GetPendingUserInvite/GetPendingInvites, AcceptInvite/AcceptInviteByToken, CancelInvite, SearchInvites, SendInviteEmail
- **RoleService**: GetAvailableRoles (from IAM adapter)
- **AccountService**: GetProjectsForAccount
- **Auth0GUIDService**: IterateAndGenerateGUIDs (batch GUID generation)

---

## Configuration

**Precedence**: Environment variables override YAML config.

**Config files**: `helm/config/{dev,us1tst,us1}.yaml` (local → staging → production).

**Key settings**:

```yaml
appConfig:
  httpPort: 8080
  grpcPort: 9090
  auth0: { client_id, client_secret, audience, domain }
  mongo: { host, port, appName, dbName }
  grpcServices:
    iambuildadapter: { url, useTls: true }
    accountservice: { url, useTls: true }
    servicecontrolservice: { url, useTls: true }
  mailgunApiKey: <secret>
  inviteUrl: https://account.sinch.com/invite/
```

---

## Critical Commands

```bash
make build          # Compile binary
make test           # Run all tests
make image          # Build Docker image
make run            # Build + run locally (dev config)
make run-local      # Run with .env overrides
make protos         # Full proto generation workflow
make generate       # Generate proto code only
make clean          # Clean artifacts
```

---

## Proto Management

- **Buf workspace**: `buf.work.yaml` defines proto directories (`proto/api`, `common/proto`)
- **Generator**: `buf.gen.yaml` generates Go code with source-relative paths to `./gen/go`
- **Vendor deps**: Fetched from upstream GitLab via `scripts/proto/fetch.sh` (requires `GITLAB_TOKEN`)
- **Custom annotations**: `common/proto/sinchid/rpc/v1/annotations.proto` — IAM context, resource defs, account ID validation

---

## Logging Strategy

- **Framework**: `go.uber.org/zap` for high-performance structured logging
- **Context propagation**: `logging.From(ctx)` for request-scoped loggers
- **Correlation IDs**: Every request gets a correlation ID in context, propagated across service boundaries
- **PII protection**: `logging.Hash("email_hash", email)` — never log raw PII
- **Log levels**: Entry/completion at info; outbound calls at debug; errors logged where they occur
- **Standard fields**: correlationId, timestamp, service name, user_id, account_id, email_hash
