# Jefe

**Repo:** git@github.com:mailgun/jefe.git
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
- Go 1.23+ (go.mod specifies 1.23.0, toolchain 1.23.6)
- MongoDB 4+
- Docker and Docker Compose (for local MongoDB)
- Make utility for build automation
- golangci-lint for code quality checks
- Git for version control

**Required Access:**
- GitHub access to mailgun/jefe repository
- Auth0 management API credentials (from Vault)
- Honeycomb API key for tracing (optional for local dev)

**Note:** If prerequisites are missing, builds and tests will fail with clear error messages.

---

## Project Overview

Jefe is a Go-based authentication and user management service that acts as the centralized entry point into Auth0's Management API for other Mailgun services. It provides a unified interface for user authentication, organization management, SAML integration, MFA operations, and user migration workflows.

**Purpose:** Centralized Auth0 Management API gateway providing user lifecycle management, organization operations, SAML SSO configuration, and MFA for Mailgun's platform.

**Business Context:** Critical authentication infrastructure primarily consumed by Gozer service, serving frontend portals including Self Management Application (SMA) and Organization Management Application (OMA).

**Service Ecosystem:**
- **Upstream Consumers:** Gozer (primary), other Mailgun services
- **Downstream Dependencies:** Auth0 Management API, MongoDB
- **Service Role:** API gateway and business logic layer between frontend services and Auth0

---

## Classification of Assignment

**Repository URL:** https://github.com/mailgun/jefe  
**Classification:** Backend Microservice / API Gateway

---

## System Architecture

**Architecture Pattern:** Layered gateway service built on Mailgun's scaffold framework

**Core Layers:**
- **API Layer:** RESTful HTTP endpoints exposed via scaffold framework
- **Manager Layer:** Business logic orchestration and Auth0 client management
- **Client Layer:** Auth0 Management API client wrapper with error translation
- **Model Layer:** Data models and MongoDB repository interfaces
- **Database Layer:** MongoDB persistence for user data and audit trails

**Technology Stack:**
- **Language:** Go 1.23.0 (toolchain 1.23.6)
- **Framework:** Mailgun scaffold framework for service structure
- **Database:** MongoDB for user data persistence
- **Authentication:** Auth0 Management API integration
- **Tracing:** OpenTelemetry with Honeycomb backend
- **HTTP Framework:** Gorilla toolkit for routing

**Data Flow:**
1. Gozer (or other services) sends HTTP requests to Jefe
2. API handlers validate requests and delegate to manager layer
3. Manager layer orchestrates business logic and calls Auth0 client
4. Client layer wraps Auth0 SDK, handles authentication, and translates errors
5. MongoDB stores user data and activity logs
6. OpenTelemetry spans track request lifecycle
7. Responses returned with appropriate status codes and data

**Key Relationships:**
- Primary consumer: Gozer service (frontend authentication portal)
- Auth0 Management API: Identity provider and user data store
- MongoDB: Local user data cache and activity tracking
- SMA/OMA: Self Management and Organization Management Applications served via Gozer

---

## Key Components

### API Modules
- **ManagerAPI**: Core Auth0 management operations (user search, creation, deletion, batch migration, job tracking)
- **UserAPI**: User-specific operations and profile management
- **MfaAPI**: Multi-factor authentication enrollment, configuration, factor validation, recovery
- **SamlAPI**: SAML integration and SSO management (connection configuration, flow management)
- **OrganizationAPI**: Multi-tenant organization operations (creation, management, user associations)
- **ConnectionAPI**: Auth0 connection management (database connections, testing, validation)

### Core Packages
- [`manager/`](manager/): Auth0 management API client and business logic orchestration
- [`api/`](api/): HTTP API endpoint implementations for all service operations
- [`model/`](model/): Data models and MongoDB database schemas
- [`mongodb/`](mongodb/): Database operations, connection management, and indexing
- [`config/`](config/): Service configuration management and YAML parsing
- [`logging/`](logging/): Structured logging and audit trail utilities
- [`security/`](security/): Cryptographic operations and security utilities
- [`activity/`](activity/): User activity tracking and comprehensive audit logging
- [`org/`](org/): Organization-specific business logic
- [`saml/`](saml/): SAML-specific operations and utilities
- [`client/`](client/): Go client library for consuming Jefe service
- [`scripts/`](scripts/): Migration automation and reporting utilities
- [`fixtures/`](fixtures/): Test data and fixtures for testing
- [`generics/`](generics/): Generic utility functions and helpers

---

## Development Environment

**Setup Steps:**

1. **Clone Repository:**
   ```bash
   git clone git@github.com:mailgun/jefe.git
   cd jefe
   ```

2. **Start MongoDB:**
   ```bash
   docker-compose up -d
   # OR manually:
   docker run -d -p 27017:27017 --name jefe-mongo mongo:latest
   ```

3. **Copy Configuration:**
   ```bash
   cp config/dev_template.yaml config/dev.yaml
   ```

4. **Configure Secrets:**
   Edit `config/dev.yaml` and update the `sinch_management` section:
   - `client_id`: Copy from staging Vault
   - `client_secret`: Copy from staging Vault
   - Other values can use template defaults for local development

5. **Install Dependencies:**
   ```bash
   go mod download
   ```

6. **Verify Setup:**
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
- **Primary Database:** `mg_dev` (user data, activity logs)
- MongoDB running on `mongodb://127.0.0.1:27017/`

**No Additional Containerization for Local Dev:** Service runs directly via `go run` or `make run` for development

**Detailed Setup Guide:** See [QUICKSTART-DEV.md](QUICKSTART-DEV.md) for comprehensive setup instructions including debugging configuration.

---

## Build & Deployment

**Build Commands:**
```bash
# Build binary
go build -o bin/jefe ./cmd/jefe

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
go test -v ./manager/...
```

**Deployment:**
- **Platform:** Nomad orchestration
- **Container:** Docker image for containerized deployment
- **Configuration:** Nomad job file at [`.director/job.nomad`](.director/job.nomad)
- **Secrets:** Pulled from Vault at runtime for Auth0 credentials
- **Update Strategy:** Rolling updates with auto-revert for production stability

**CI/CD:**
- GitHub Actions for CI/CD pipeline
- Automated testing on pull requests
- Container builds on merge to main
- Deployment triggered via Nomad job updates

---

## Project Structure

**Directory and File Roles:**
- [`cmd/jefe/`](cmd/jefe/): Application entry point and main function
- [`api/`](api/): HTTP API handlers for all service endpoints
- [`manager/`](manager/): Business logic orchestration and Auth0 client management
- [`client/`](client/): Go client library for consuming Jefe service
- [`model/`](model/): Data models and MongoDB repository interfaces
- [`mongodb/`](mongodb/): Database operations, connection management, and indexing
- [`config/`](config/): Configuration files and loading
- [`logging/`](logging/): Structured logging utilities
- [`security/`](security/): Cryptographic operations and security utilities
- [`activity/`](activity/): User activity tracking and audit logging
- [`org/`](org/): Organization-specific business logic
- [`saml/`](saml/): SAML-specific operations and utilities
- [`scripts/`](scripts/): Migration automation and reporting utilities
- [`fixtures/`](fixtures/): Test data and fixtures for testing
- [`generics/`](generics/): Generic utility functions and helpers
- [`.director/`](.director/): Nomad deployment configuration
- [`.github/`](.github/): GitHub Actions workflows
- [`Makefile`](Makefile): Build automation targets
- [`docker-compose.yaml`](docker-compose.yaml): Local MongoDB setup

---

## Key Files

**Entry Point:**
- [`cmd/jefe/main.go`](cmd/jefe/main.go): Application bootstrap and initialization

**Core Service:**
- [`service.go`](service.go): Service structure, lifecycle, and API initialization

**Business Logic:**
- [`manager/manager.go`](manager/manager.go): Business logic orchestration, error handling, and tracing
- [`manager/client.go`](manager/client.go): Auth0 SDK wrapper, mockable interface, error translation

**API Implementations:**
- [`api/manager.go`](api/manager.go): Core Auth0 management API endpoints
- [`api/user.go`](api/user.go): User-specific operations and profile management
- [`api/mfa.go`](api/mfa.go): Multi-factor authentication operations
- [`api/saml.go`](api/saml.go): SAML authentication integration
- [`api/organization.go`](api/organization.go): Organization management endpoints
- [`api/connection.go`](api/connection.go): Auth0 connection management endpoints
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
- [`QUICKSTART-DEV.md`](QUICKSTART-DEV.md): Comprehensive development guide with debugging setup

---

## Testing Strategy

**Testing Philosophy:** Comprehensive test coverage with unit tests alongside implementation files

**Test Organization:**
- Test files co-located with source: `*_test.go` alongside implementation
- Table-driven tests using testify/assert
- Mock external dependencies (Auth0 Management API)
- Use real MongoDB for integration-style tests

**Test Coverage Areas:**
- All API endpoints have corresponding test files
- Manager layer orchestration logic tested thoroughly
- Auth0 client wrapper with error translation tested
- Model layer with MongoDB operations tested
- MFA, SAML, Organization operations covered

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
go test -v ./manager/...

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
- Auth0 Management API authentication via client credentials
- OAuth2 client credentials flow for service-to-service authentication
- JWT token handling for Auth0 integrations

**Authorization:**
- API key or bearer token authentication for Jefe endpoints
- Auth0 role-based access control (RBAC) management
- Organization-level permissions and scoping

**Data Protection:**
- Secure storage of Auth0 credentials in Vault
- PII handling with appropriate logging safeguards
- Encrypted communication with Auth0 Management API (HTTPS)

**Security Standards:**
- No hardcoded secrets (all from Vault or config)
- Input validation on all API endpoints
- Audit logging for security-relevant operations
- User activity tracking for compliance

**Compliance:**
- Comprehensive audit logging in activity package
- User lifecycle event tracking
- SAML SSO support for enterprise security requirements

---

## Monitoring & Operations

**Logging:**
- Structured logging via logrus
- Context-aware logging with request IDs
- Activity logging for audit trails
- Log level configurable per environment

**Tracing:**
- OpenTelemetry instrumentation throughout codebase
- Spans for all API requests, Auth0 calls, and database operations
- Export to Honeycomb for distributed tracing
- Configurable sampling rate (default 100% for dev, lower for prod)
- Trace configuration via environment variables

**Metrics:**
- Prometheus metrics exposed at `/metrics` endpoint
- HTTP request metrics (count, duration, status codes)
- Auth0 API call metrics
- MongoDB operation metrics
- Service health indicators

**Health Checks:**
- Metrics endpoint doubles as health check
- Service registration with health monitoring

**Troubleshooting:**
- Check logs for Auth0 API failures
- Verify MongoDB connectivity
- Confirm Auth0 credentials configuration
- Verify client_id and client_secret from Vault
- See [QUICKSTART-DEV.md](QUICKSTART-DEV.md) for debugging guide

---

## Technical Stack & Dependencies

**Runtime:**
- Go 1.23.0 (toolchain 1.23.6)
- MongoDB 4+

**Core Framework:**
- mailgun/scaffold v1.76.0: Service framework and common utilities
- mailgun/holster/v4 v4.16.0: Utility library
- gorilla/mux: HTTP routing (via scaffold)

**Authentication:**
- auth0/go-auth0 v1.26.0: Auth0 Management API client

**Database:**
- go.mongodb.org/mongo-driver v1.11.6: MongoDB client with connection pooling

**Observability:**
- go.opentelemetry.io/otel v1.19.0: OpenTelemetry SDK
- go.opentelemetry.io/otel/trace v1.19.0: Tracing API
- prometheus/client_golang v1.13.0: Prometheus metrics
- sirupsen/logrus v1.9.2: Structured logging

**Testing:**
- stretchr/testify v1.10.0: Test assertions and mocking

**Utilities:**
- google/uuid v1.6.0: UUID generation
- pkg/errors v0.9.1: Error handling with stack traces
- golang.org/x/crypto v0.36.0: Cryptographic utilities
- pb33f/libopenapi v0.9.2: OpenAPI specification handling

---

## Service Contract

**Primary Service:** Auth0 Management API gateway providing user lifecycle management, organization operations, and authentication configuration

**Key Operations:**
- User lifecycle management (search, create, update, delete, batch migration)
- Organization management (create, update, member management)
- SAML configuration and SSO setup
- MFA enrollment, configuration, and validation
- Auth0 connection management and testing
- User activity tracking and audit logging

**API Standards:**
- RESTful HTTP endpoints using scaffold framework
- All endpoints prefixed with `/v1/auth/management/`
- JSON request/response bodies
- Standard HTTP status codes
- Comprehensive error messages with context

**Integration Points:**
- Gozer: Primary consumer for frontend portal authentication
- Auth0 Management API: Downstream identity provider
- MongoDB: Local data persistence and activity logs
- Vault: Secrets management for Auth0 credentials

---

## Core Business Functions

**User Lifecycle Management:**
- User creation, retrieval, update, and deletion via Auth0
- User search with filtering and pagination
- Bulk user migration with job tracking
- User profile management
- User authentication state management

**Organization Management:**
- Multi-tenant organization creation and configuration
- User-organization association management
- Organization member operations
- Organization metadata management

**Multi-Factor Authentication:**
- MFA enrollment flows
- Factor configuration (OTP, SMS, etc.)
- Factor validation and challenge
- MFA recovery operations

**SAML Authentication:**
- SAML connection configuration for enterprise SSO
- SAML metadata management
- SSO flow setup and testing
- Certificate management for SAML

**Connection Management:**
- Auth0 database connection configuration
- Connection testing and validation
- Connection metadata management

**Activity Tracking:**
- Comprehensive user activity logging
- Admin operation audit trails
- Activity history and reporting

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
- Sinch Management API configuration

**Database Configuration:**
- MongoDB URI for main database (default: `mongodb://127.0.0.1:27017/mg_dev`)

**Sinch/Auth0 Management API:**
- Audience: Auth0 API audience
- Domain: Auth0 domain (e.g., `staging-id.sinch.com`)
- Connection: Default connection name (e.g., `Sinch-Users`)
- Connection ID: Auth0 connection identifier
- Migration batch size: User migration batch size (default: 100)

**SAML Configuration:**
- Default enabled clients for SAML connections
- Client IDs for Dashboard, Sinch ID Management Portal, Mailgun

**Secrets Management:**
- Development: Edit `dev.yaml` with staging Vault credentials
- Production: Vault integration via Nomad templates
- Required secrets: Auth0 client_id, client_secret, encryption keys

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
go build -o bin/jefe ./cmd/jefe

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
go test -v ./manager/...
```

**Docker & Database:**
```bash
# Start MongoDB via compose
docker-compose up -d

# Stop MongoDB
docker-compose down

# View MongoDB logs
docker logs jefe-mongo
```

**Verification:**
```bash
# Verify Go version
go version

# Verify Docker status
docker ps

# View service logs (when running)
# Check terminal output for service logs
```

---

## General Guidance for AI Agents

### Code Standards and Architecture
- Follow existing abstraction layers and maintain established separation of concerns
- Adhere to Go language coding standards and conventions
- Use coding style found in each file; avoid deviations from established patterns
- Preserve existing functions, methods, procedures, and unit tests unless explicitly required
- Follow Mailgun scaffold framework patterns for service structure
- Maintain layered architecture: API → Manager → Client → Auth0/MongoDB

### Dependencies and Libraries
- Use current library versions from go.mod; avoid downgrading
- Only use official, well-maintained, and highly-rated libraries
- Maintain consistency with existing dependency patterns
- Leverage Mailgun's internal libraries (scaffold, holster) when applicable

### Error Handling and Security
- Apply consistent error handling strategies as established in codebase
- Wrap errors with context using pkg/errors for stack traces
- Follow security best practices for authentication services
- Never log sensitive data (passwords, tokens, API keys, PII)
- Implement performance-conscious solutions
- Validate all inputs at API boundaries
- **Do not document secrets or credentials** in checked-in files
- For secrets documentation, create AGENT-ASSIGNMENT-RESOURCES.md (git-ignored) and mark "DO NOT SHARE"

### Testing Requirements
- Write tests alongside implementation (`*_test.go` files)
- Use table-driven tests with testify assertions
- Mock external dependencies (Auth0 Management API)
- Use real MongoDB for integration-style tests
- **Coverage Standards:** >90% for new features, 100% for defect fixes
- Run `make test` before committing to verify all tests pass with race detection
- Test edge cases, error conditions, and happy paths

### Code Quality
- Write specific, accurate, and high-value comments
- Make changes only to files directly related to assigned task
- Maintain code clarity and readability
- Use descriptive variable and function names
- Run `make lint` before committing to catch code quality issues
- Follow golangci-lint standards configured in [`.golangci.yml`](.golangci.yml)

### Communication Guidelines
- Ask questions when requirements or implementation details are unclear
- Notify when additional information is needed rather than making assumptions
- Do not create solutions without proper clarification when facing ambiguity
- Document architectural decisions in assignment.md
- Provide clear, concise commit messages

### Safety Constraints
- **`rm` commands CANNOT be auto-accepted** → Always require explicit user confirmation
- **Destructive operations** → Any command that deletes or irreversibly modifies files must be user-approved
- **Code Quality Gates** → All code must pass `make lint` and `make test` with no critical issues


## Deployment

### Staging

After a GitHub PR is created, go to Slack `#eng-director-dev`:

1. Check current version:
   ```
   @Director-Develop current version svc=jefe
   ```
   Result: creates a thread with current deployed version info.

2. In the thread, deploy:
   ```
   @Director-Develop deploy svc=jefe tag=PR<NUMBER> env=mfstaging region=us-east4
   ```
   - `tag`: GitHub PR number in `PR###` format (e.g. `PR118`)
   - On success: is-healthy message shown in thread
   - On failure: evaluate whether this is an infrastructure or service failure
   - Rollback: TBD

### Production

After the GitHub PR is merged, go to Slack `#chatops`:

1. Check current version:
   ```
   @Director current version svc=jefe
   ```

2. Deploy to each region **one at a time** in order:
   ```
   @Director deploy svc=jefe tag=PR<NUMBER> env=mfproduction region=us-east4
   @Director deploy svc=jefe tag=PR<NUMBER> env=mfproduction region=us-west1
   @Director deploy svc=jefe tag=PR<NUMBER> env=mfproduction region=europe-west1
   ```
   - On success: is-healthy message shown in thread
   - On failure: evaluate whether this is an infrastructure or service failure
   - Rollback: TBD
