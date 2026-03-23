
# LLM Agent Preamble

**CRITICAL REQUIREMENTS:**
- **READ AND ADHERE** → This file MUST be read and followed for ALL units of work and ALL editing sessions
- **REPOSITORY VALIDATION** → Review this document to ensure it accurately describes the repository you are working in
- **MISMATCH NOTIFICATION** → If this document does not match the actual repository structure, dependencies, or purpose, IMMEDIATELY notify the user of the discrepancy

**AGENT RESPONSIBILITY:** Before beginning any work, validate that this guide matches the current repository context and report any inconsistencies to the user.

---

# Prerequisites
- Go 1.25.7+ (for module analysis)
- Valid go.mod (Go module root)
- Make utility (for automation)
- jq (for JSON validation)
- Unix environment (bash/zsh, find, grep, wc)

Git Repo: gitlab.com/sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/common
Date Last Updated: 2026-03-21 00:00

---

## Table of Contents
- LLM Agent Preamble
- Prerequisites
- Project Overview
- Classification of Assignment
- System Architecture
- Key Components
- Project Structure
- Key Files
- Development Environment
- Build & Deployment
- Testing Strategy
- Security & Compliance
- Monitoring & Operations
- Technical Stack & Dependencies
- Service Contract & Core Business Functions
- Configuration
- Critical Commands
- General Guidance for AI Agents
- Additional Notes

---

## Project Overview
Common libraries and utilities for various projects. Each subdirectory contains a specific library or utility for Go projects. Used as a submodule in multiple services for shared authentication, logging, metrics, and more.

## Classification of Assignment
- Repository URL: gitlab.com/sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/common
- Classification: Go Library/SDK, Backend Utilities

## System Architecture
- Modular Go package structure under `pkg/`
- Each subdirectory provides a focused utility (auth, logging, metrics, etc.)
- Used as a submodule in consuming services
- No standalone deployment; always integrated

## Key Components
- **auth/**: Authentication logic, JWT validation, and helpers
- **basicauth/**: Basic authentication middleware and interceptors
- **clients/**: Service clients for account, auth0, email, IAM, servicecontrol
- **correlation/**: Correlation ID management utilities
- **logging/**: Zap-based logging utilities and gRPC interceptors
- **metrics/**: Prometheus metrics for gRPC and REST
- **mongodb/**: MongoDB connection, config, and index helpers
- **permissions/**: Permission management utilities
- **services/**: Service-layer business logic for account, user, invite, role, etc.
- **token/**: Token validation, JWKS, claims, and refresh logic
- **validation/**: Input validation utilities

## Project Structure
```
pkg/
	annotations/         # Protocol buffer annotation utilities
	auth/                # Authentication logic and helpers
	basicauth/           # Basic authentication middleware
	clients/             # Service clients (account, auth0, email, iam, servicecontrol)
	correlation/         # Correlation ID management
	cors/                # CORS middleware
	generics/            # Generic helpers
	logging/             # Logging utilities and interceptors
	metrics/             # Prometheus metrics for gRPC/REST
	mongodb/             # MongoDB connection and index helpers
	permissions/         # Permission management
	services/            # Business logic for account, user, invite, role, etc.
	test/                # Test fixtures, mocks, and testutils
	token/               # Token validation and refresh
	validation/          # Input validation
```

## Key Files
- `Makefile`: Build, test, and proto generation automation
- `README.md`: Project documentation and submodule workflow
- `go.mod`: Go module definition and dependencies
- `pkg/auth/auth0.go`: Auth0 JWT validation logic
- `pkg/auth/serviceControl.go`: ServiceControl token validation
- `pkg/logging/log.go`: Zap logger setup
- `pkg/metrics/grpc.go`, `pkg/metrics/rest.go`: Prometheus metrics collectors
- `pkg/mongodb/mongodb.go`: MongoDB connection and config
- `pkg/services/account.go`, `user.go`, `invite.go`, `role.go`: Service interfaces and implementations
- `pkg/services/model/`: Data models for users, invites, errors
- `pkg/test/`: Test fixtures, mocks, and test utilities

## Development Environment
- Go 1.25.7+
- Makefile for common tasks
- No database required for core utilities (MongoDB only for consuming services)
- Environment variables: only as required by consuming service
- No containerization required for library itself

## Build & Deployment
- Use `make` for branch switching, syncing, and status
- No direct deployment; update submodule pointer in consuming repo
- CI/CD handled by consuming service

## Testing Strategy
**Philosophy:**
- Unit tests co-located with implementation files (`*_test.go`)
- Table-driven tests using testify/assert
- Mocks for external dependencies (auth0, IAM, email)
- Real MongoDB for integration tests (in consuming service context)

**Coverage Requirements:**
- New features: >90% coverage
- Defect fixes: 100% coverage

**Test Execution:**
- `make test` — run all tests
- `go test -v ./pkg/services/...` — run service tests
- `go test -coverprofile=coverage.out ./...` — coverage report

## Security & Compliance
- Auth0 JWT validation for authentication utilities
- Role-based access control helpers
- No hardcoded secrets; all secrets/config provided by consuming service
- Input validation utilities for API boundaries
- No direct data persistence; all DB access via consuming service

## Monitoring & Operations
- Prometheus metrics collectors for gRPC and REST
- Zap-based structured logging utilities
- Correlation ID propagation for distributed tracing
- No direct health checks; provided by consuming service

## Technical Stack & Dependencies
- Go 1.25.7+
- go.uber.org/zap: Structured logging
- github.com/prometheus/client_golang: Prometheus metrics
- github.com/golang-jwt/jwt/v5: JWT handling
- go.mongodb.org/mongo-driver: MongoDB client
- github.com/stretchr/testify: Test assertions and mocking
- google.golang.org/grpc: gRPC framework

## Service Contract & Core Business Functions
- Provides authentication, logging, metrics, and service-layer business logic as a Go library
- Consumed as a submodule in multiple services
- No standalone API or deployment
- Key operations: token validation, logging, metrics, account/user/invite/role management helpers

## Configuration
- No direct configuration; all config/env vars provided by consuming service
- MongoDB config only used in consuming service context
- No secrets or credentials stored in this repo

## Critical Commands
```bash
# Run all tests
make test

# Switch to a feature branch
make switch-branch b=feat/my-feature

# Sync current branch
make sync-branch

# Check current state
make git-status
```

## General Guidance for AI Agents
- Follow Go language coding standards and conventions
- Maintain established abstraction layers and separation of concerns
- Use the coding style found in each file; avoid deviations
- Preserve existing functions, methods, and tests unless explicitly required
- Only use official, well-maintained libraries
- Never log sensitive data (passwords, tokens, PII)
- Validate all inputs at API boundaries
- Write tests alongside implementation; use table-driven tests
- Mock external dependencies in tests
- Do not document secrets or credentials in checked-in files

## Additional Notes
- See README.md in repo root for detailed usage and workflow
- For submodule integration, always update and commit the submodule pointer in the parent repo
