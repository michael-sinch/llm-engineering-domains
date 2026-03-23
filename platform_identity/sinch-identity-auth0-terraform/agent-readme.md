# Sinch Identity Auth0 Terraform - AGENT README

**Repository:** git@github.com:mailgun/sinch-identity-auth0-terraform.git
**Last Updated:** 2026-02-24

---

## Table of Contents
- [Project Overview](#project-overview)
- [Prerequisites](#prerequisites)
- [Classification of Assignment](#classification-of-assignment)
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
- [Known Issues & Considerations](#known-issues--considerations)
- [Code Standards and Architecture](#code-standards-and-architecture)
- [Dependencies and Libraries](#dependencies-and-libraries)
- [Error Handling and Security](#error-handling-and-security)
- [Testing Requirements](#testing-requirements)
- [Formatting and Linting (CRITICAL)](#formatting-and-linting-critical)

---

## Project Overview
This repository contains Terraform configurations and Node.js-based Auth0 Actions for managing Sinch/Mailgun's Auth0 identity platform. It handles user authentication flows, MFA enforcement, invitation workflows, account provisioning, cross-boarding between products, and custom JWT claims. The system integrates with external services including RudderStack (analytics), Sentry (error tracking), ConfigCat (feature flags), and Mailgun/Sinch APIs.

**Repo**: git@github.com:mailgun/sinch-identity-auth0-terraform.git
**Repo Classification**: Infrastructure as Code (Terraform) + Serverless Functions (Auth0 Actions)

## Prerequisites
- Terraform CLI (latest recommended)
- Node.js 22.17.0+ (for Auth0 Actions development)
- npm (comes with Node.js)
- Auth0 account with management API access
- Google Cloud CLI (for accessing remote state in staging/prod environments)
- jq (for JSON validation)
- Unix environment (bash/zsh, find, grep, wc)

Note: If prerequisites are missing, discovery fails with a clear error message.

## Classification of Assignment
Repository URL: https://github.com/mailgun/sinch-identity-auth0-terraform
Classification: Infrastructure as Code (Terraform) + Serverless Functions (Auth0 Actions)

## System Architecture
**Components:**
- Terraform configurations (.tf files) defining Auth0 resources (tenants, clients, actions, connections, flows, email templates, MFA policies)
- Auth0 Actions (JavaScript/Node.js) deployed as serverless functions within Auth0 authentication flows
- Custom UI components for Auth0 Universal Login (HTML/JavaScript/CSS)
- Liquid templates for email notifications with multi-language support
- Environment-specific configurations (dev, staging, prod via .tfvars files)

**Relationships & Data Flow:**
1. Users authenticate via Auth0 Universal Login
2. Auth0 flows trigger custom Actions (Node.js serverless functions)
3. Actions interact with external services:
   - RudderStack for event tracking
   - Sentry for error monitoring
   - ConfigCat for feature flags
   - Mailgun APIs for email delivery
   - Sinch APIs for SMS delivery
   - Customer provisioning services
4. Terraform manages deployment and configuration of all Auth0 resources
5. CI/CD pipeline auto-deploys on merges to main branch

## Development Environment
**Setup Steps:**
1. Install Terraform CLI: https://developer.hashicorp.com/terraform/tutorials/gcp-get-started/install-cli
2. Install Node.js 22.17.0+ from https://nodejs.org/
3. Set up Auth0 application with management API access (see README.md for detailed steps)
   - Note: Uncheck `delete:connections` API permission for safety
4. For staging/prod environments: Install Google Cloud CLI and authenticate
5. Install npm dependencies in `sources/` directory: `cd sources && npm install`
6. Copy `.backend_override.tf` to `backend_override.tf` for local state (or configure remote backend)

**Environment Variables:**
```bash
# Required for local Terraform operations
export TF_VAR_auth0_domain={AUTH0_DOMAIN}
export TF_VAR_auth0_client_id={CLIENT_ID}
export TF_VAR_auth0_client_secret={CLIENT_SECRET}
```

**Optional Environment Variables (GitHub Secrets for pipeline):**
- Sinch SMS action configuration
- Mailgun email provider settings
- Various API keys for integrations (RudderStack, Sentry, ConfigCat)

**Remote State (Staging/Prod):**
- Google Cloud Storage backend for Terraform state
- Requires Google Cloud access and group membership (request via MG-servicedesk@sinch.com)
- Bucket: `mailforce-staging-auth0-tf` (staging), similar for prod
- Authentication: `gcloud auth login` before Terraform operations

**No Database Required:**
Auth0 manages user data; no separate database needed for this repository.

## Build & Deployment

**Local Development (Terraform):**
```bash
# Copy local state override (for local testing only)
cp .backend_override.tf backend_override.tf

# Initialize Terraform
terraform init

# Plan changes (review carefully before applying)
terraform plan

# Apply changes (use with extreme caution - modifies live Auth0 config)
terraform apply
```

**Targeting Specific Environments:**
```bash
# Staging
gcloud auth login
terraform init --backend-config="bucket=mailforce-staging-auth0-tf"
terraform plan --var-file=env/staging.tfvars

# Production (similar process with prod bucket and tfvars)
```

**Auth0 Actions Development (sources/ directory):**
```bash
cd sources/
npm install                   # Install dependencies
npm run build                # Build Actions with Rollup
npm test                     # Run tests with Vitest
npm run ci:test              # Run tests in CI mode (silent)
npm run lint                 # ESLint check
npm run fix                  # Auto-fix linting issues
npm run prettier:check       # Verify code formatting
npm run prettier:write       # Format all code files
```

**CRITICAL PRE-COMMIT VERIFICATION:**
Before committing any changes to Auth0 Actions (sources/), you MUST run:
```bash
cd sources/
npm run prettier:write       # Format code
npm run lint                 # Verify no linting errors
npm test                     # Verify all tests pass
```
The CI/CD pipeline will fail if code is not properly formatted with Prettier.

**CI/CD Pipeline:**
- GitHub Actions automatically runs on pull requests and merges to main
- Pipeline runs: Prettier formatting check, ESLint, tests
- On merge to main: Terraform automatically applies changes to Auth0
- **WARNING:** Terraform auto-deploys on merges to main - review changes carefully!

## Project Structure
**Directory and File Roles:**
- [`actions.tf`](actions.tf): Auth0 Actions definitions and code references
- [`backend.tf`](backend.tf): Terraform backend configuration for state management
- [`connections.tf`](connections.tf): Auth0 database connections (user storage)
- [`custom_apis.tf`](custom_apis.tf): Custom API configurations
- [`email_templates.tf`](email_templates.tf): Email template definitions
- [`flows.tf`](flows.tf): Auth0 flow configurations (login, signup, etc.)
- [`mfa.tf`](mfa.tf): MFA settings and policies
- [`tenant.tf`](tenant.tf): Auth0 tenant configuration
- [`universal_login.tf`](universal_login.tf): Universal Login customization
- [`variables.tf`](variables.tf): Terraform variable definitions
- [`versions.tf`](versions.tf): Provider version constraints
- [`sources/`](sources/): Auth0 Actions source code (Node.js)
- [`sources/src/`](sources/src/): Action implementation files
- [`sources/tests/`](sources/tests/): Test files for Actions
- [`sources/shared/`](sources/src/shared/): Shared utilities and helpers
- [`custom_components/`](custom_components/): Custom UI components for Universal Login
- [`liquid_template/`](liquid_template/): Email templates with language files
- [`env/`](env/): Environment-specific tfvars files (dev, staging, prod)

## Key Files
**Important Terraform Files:**
- [`main.tf`](main.tf): Main Terraform configuration and provider setup
- [`flows.tf`](flows.tf): Auth0 flow orchestration
- [`actions.tf`](actions.tf): Auth0 Actions deployment configuration

**Important Auth0 Action Files:**
- [`sources/src/enforceMFA.js`](sources/src/enforceMFA.js): MFA enforcement logic with grace period
- [`sources/src/provisionAccount.js`](sources/src/provisionAccount.js): Account provisioning workflows
- [`sources/src/handleInvitationToBuild.js`](sources/src/handleInvitationToBuild.js): Build invitation flow
- [`sources/src/handleAccountCenterInvitation.js`](sources/src/handleAccountCenterInvitation.js): Account Center invitations
- [`sources/src/customClaims.js`](sources/src/customClaims.js): Custom JWT claims injection
- [`sources/src/checkBlocklist.js`](sources/src/checkBlocklist.js): Blocklist verification
- [`sources/src/securityCheckpoint.js`](sources/src/securityCheckpoint.js): Security validation
- [`sources/src/handleCrossboarding.js`](sources/src/handleCrossboarding.js): Cross-boarding between products

**Configuration Files:**
- [`sources/package.json`](sources/package.json): Node.js dependencies and scripts
- [`sources/rollup.config.js`](sources/rollup.config.js): Build configuration for Actions
- [`sources/vitest.config.js`](sources/vitest.config.js): Test configuration
- [`sources/eslint.config.js`](sources/eslint.config.js): Linting rules

## Testing Strategy
**Testing Approaches:**
- Unit tests for Auth0 Actions using Vitest
- Mocked external dependencies (@rudderstack, @sentry, configcat-node) via `__mocks__/` directory
- Test files mirror source structure in `sources/tests/`
- Each Action should have corresponding test file (e.g., `enforceMFA.js` → `enforceMFA.test.js`)
- Silent CI mode for pipeline execution (`npm run ci:test`)

**Test Commands:**
```bash
cd sources/
npm test                      # Run tests with output
npm run ci:test              # Run tests in silent mode (CI)
npm test <filename>          # Run specific test file
```

**Test Coverage:**
Tests validate core business logic including:
- User provisioning flows
- Invitation handling (Build, Account Center, legacy)
- MFA enforcement with various scenarios
- Account creation and verification
- Email and phone verification
- Cross-boarding workflows
- Custom claims generation
- Security checkpoints

**Coverage Goals:**
- Aim for comprehensive test coverage of all Actions
- Test edge cases, error conditions, and happy paths
- Mock external API calls to ensure test reliability
- Verify backward compatibility when making changes

## Security & Compliance
**Security Requirements:**
- Auth0 Management API access with appropriate scopes (excluding `delete:connections` for safety)
- Google Cloud IAM permissions for remote state access in staging/prod
- Secure management of Auth0 client credentials via environment variables
- Protection against accidental deletion of user database connections (lifecycle prevent_destroy in Terraform)
- Never commit secrets, API keys, or credentials to the repository
- Use GitHub Secrets for sensitive CI/CD variables

**Standards & Best Practices:**
- OAuth2/OIDC standards for authentication
- SAML support for enterprise SSO
- MFA enforcement policies with grace periods for new users
- Email verification requirements
- Risk assessment integration
- Secure session management with "remember browser" functionality

**Security Considerations:**
- Deleting a database connection in Auth0 will delete associated users - use extreme caution
- Review Terraform plan output carefully before applying changes
- Test changes in dev environment before deploying to staging/prod
- Monitor Auth0 logs for suspicious activity

## Monitoring & Operations
**Logging:**
- Console.log statements in Auth0 Actions for debugging
- Auth0 real-time logs available in Auth0 dashboard (Monitoring → Logs)
- Structured logging with context for troubleshooting

**Error Tracking:**
- Sentry integration for error tracking (@sentry/aws-serverless)
- Errors automatically captured and reported
- Stack traces available in Sentry dashboard

**Analytics:**
- RudderStack event tracking for user actions
- Track signup, login, MFA enrollment, invitation flows
- Custom event properties for business intelligence

**Feature Flags:**
- ConfigCat feature flag management
- Toggle features without code deployment
- Environment-specific flag configurations

**Troubleshooting:**
- Check Auth0 dashboard logs for Action execution details
- Review Terraform plan output before applying changes
- Validate environment variables are set correctly
- Known issue: Some resources may appear to be destroyed/modified in plan due to missing optional env vars (sinch_sms_action, mailgun_email_provider) - this is expected

## Technical Stack & Dependencies
**Runtime:**
- Node.js 22.17.0+
- Terraform (latest recommended)

**Auth0 Integration:**
- auth0 npm package (^4.27.0)
- Auth0 Terraform Provider

**External Services:**
- RudderStack (@rudderstack/rudder-sdk-node ^2.1.5): Analytics and event tracking
- Sentry (@sentry/aws-serverless ^9.43.0): Error tracking and monitoring
- ConfigCat (configcat-node ^11.4.0): Feature flag management
- Mailgun: Email delivery service
- Sinch: SMS delivery service

**Testing & Development:**
- Vitest (^3.2.4): Test runner with fast execution
- ESLint (^9.32.0): Linting and code quality
- Prettier (^3.6.2): Code formatting (REQUIRED - CI will fail without proper formatting)
- Rollup (^4.46.2): Bundling for Auth0 Actions
- Nock (^14.0.7): HTTP mocking for tests

**Other Dependencies:**
- axios (^1.11.0): HTTP client
- libphonenumber-js (^1.12.33): Phone number validation
- @bufbuild/protobuf (^2.8.0): Protocol Buffers support
- @connectrpc/connect (^2.1.0): Connect RPC framework

## Service Contract
**Primary Service:** Auth0 Identity Platform Configuration and Custom Action Handlers

**Key Operations:**
- User authentication and authorization
- MFA enrollment and challenge with grace periods for new users
- Custom claims injection into JWT tokens
- Email and phone verification workflows
- Invitation workflow processing (Build, Account Center, legacy invites)
- Account provisioning and cross-boarding between Mailgun/Sinch products
- Role assignment and permissions management
- Security checkpoints and blocklist verification

**API Standards:**
- Auth0 Actions API (event-driven serverless functions triggered by authentication flows)
- Auth0 Management API for configuration and user management
- RESTful APIs for external service integrations

## Core Business Functions
**User Lifecycle:**
- User signup with email/phone verification
- Account provisioning with external systems integration
- Cross-boarding between different Mailgun/Sinch products
- Role assignment and permissions management
- User metadata management

**MFA Management:**
- Enrollment flow with OTP/TOTP/SMS options
- Challenge with primary factor (OTP, phone) and recovery codes
- Grace period for newly created users (2 hours) to prevent double MFA on first login
- "Remember browser" functionality for trusted devices
- Force MFA vs. optional MFA based on ACR values and user metadata

**Invitation Workflow:**
- Build invitation handling with role assignment
- Account Center invitation processing
- Legacy invitation support for backward compatibility
- Automatic role assignment upon invitation acceptance
- Invitation token validation and expiration

**Multi-tenant Support:**
- Organization-based access control
- SAML SSO for enterprise customers
- Custom domain support

**Security Features:**
- Blocklist verification during signup
- Email domain validation
- Suspicious enrollment detection
- Security checkpoints in authentication flows

## Configuration
**Environment Files:**
- [`env/dev.tfvars`](env/dev.tfvars): Development environment variables
- [`env/staging.tfvars`](env/staging.tfvars): Staging environment variables
- [`env/prod.tfvars`](env/prod.tfvars): Production environment variables

**Secrets Management:**
- Auth0 credentials via TF_VAR environment variables (never commit these)
- GitHub Secrets for CI/CD pipeline variables
- Google Cloud Secret Manager (via service accounts) for staging/prod
- Vault integration for sensitive configuration

**Configuration Files:**
- [`backend.tf`](backend.tf): Remote state configuration (GCS buckets)
- `backend_override.tf`: Local development override (not in git, created from template)
- [`sources/rollup.config.js`](sources/rollup.config.js): Build configuration
- [`sources/vitest.config.js`](sources/vitest.config.js): Test configuration
- [`sources/eslint.config.js`](sources/eslint.config.js): Linting rules

## Critical Commands
**Terraform Operations:**
```bash
terraform init                               # Initialize Terraform
terraform plan                               # Preview changes (ALWAYS review before apply)
terraform apply                              # Apply configuration (use with caution)
terraform plan --var-file=env/staging.tfvars # Plan for specific environment
terraform init --backend-config="bucket=..."  # Initialize with remote backend
```

**Auth0 Actions Development:**
```bash
cd sources/
npm install                   # Install dependencies
npm run build                # Build Actions with Rollup
npm test                     # Run tests
npm run ci:test              # Run tests (CI mode)
npm run lint                 # Lint code
npm run fix                  # Fix linting issues
npm run prettier:check       # Check formatting (CI requirement)
npm run prettier:write       # Format code (REQUIRED before commit)
```

**Git Operations:**
```bash
git status                   # Check current state
git add <files>             # Stage changes
git commit -m "message"     # Commit changes
git push origin <branch>    # Push to remote (triggers CI/CD)
```

**Pre-Commit Checklist:**
Before committing changes to Auth0 Actions:
1. `cd sources/`
2. `npm run prettier:write` - Format all code
3. `npm run lint` - Verify no linting errors
4. `npm test` - Verify all tests pass
5. Review changes carefully - auto-deploys on merge to main!

## Known Issues & Considerations

**Terraform Lifecycle:**
- Deleting a database connection will delete associated users - lifecycle `prevent_destroy` is configured to prevent accidental deletion
- To destroy an environment, you must target individual resources: https://developer.hashicorp.com/terraform/tutorials/state/resource-targeting
- Branding resource cannot be destroyed through Terraform

**Expected Plan Warnings:**
When running Terraform plan without optional environment variables, expect these resources to show changes (this is normal):
- `auth0_action.sinch_sms_action[0]` - appears to be destroyed
- `auth0_email_provider.mailgun_email_provider[0]` - appears to be destroyed
- `auth0_trigger_action.send_phone_message_sinch_sms_action[0]` - appears to be destroyed
- `auth0_client.auth0_clients[2]` web_origins will lose `https://*.ninowire.com` (cannot be terraformed yet)

**CI/CD Considerations:**
- Prettier formatting is STRICTLY enforced - CI will fail if code is not formatted
- All tests must pass before merge
- Terraform auto-deploys on merge to main - review changes thoroughly
- Use feature branches and pull requests for all changes

**Universal Login Limitations:**
- Limited customization options for Universal Login UI
- Some features cannot be fully managed through Terraform

# General Guidance

## Code Standards and Architecture
- Follow existing patterns in Auth0 Actions (see similar files for reference)
- Maintain separation between Action logic and shared utilities
- Use the coding style found in each file; avoid deviations from established patterns
- Preserve existing functions, methods, procedures, and unit tests unless explicitly required for the task
- Add comprehensive JSDoc comments for all new functions
- Include detailed console.log statements for debugging (viewable in Auth0 logs)

## Dependencies and Libraries
- Use current library versions from package.json
- Only use official, well-maintained, and highly-rated libraries for new solutions
- Maintain consistency with existing dependency patterns
- Avoid adding unnecessary dependencies - leverage existing utilities in `sources/src/shared/`

## Error Handling and Security
- Apply consistent error handling strategies as established in the codebase
- Use try-catch blocks for async operations
- Log errors with sufficient context for troubleshooting
- Return early on error conditions to prevent cascading failures
- Follow security best practices for authentication systems:
  - Never log sensitive data (passwords, tokens, API keys)
  - Validate all inputs
  - Use parameterized queries
  - Implement rate limiting where appropriate
- Implement performance-conscious solutions (Actions must execute quickly)

## Testing Requirements
- Create test file for every new Action (e.g., `myAction.js` → `myAction.test.js`)
- Use Vitest framework with mocked dependencies
- Test happy paths, edge cases, and error conditions
- Mock external API calls using Nock or vi.mock()
- Aim for comprehensive coverage of business logic
- Run tests locally before committing: `npm test`
- Verify CI tests pass before merging

## Formatting and Linting (CRITICAL)
- **ALWAYS run `npm run prettier:write` before committing** - CI will fail without proper formatting
- Run `npm run lint` to check for code quality issues
- Fix linting errors with `npm run fix` when possible
- Follow ESLint rules configured in `eslint.config.js`
- Maintain consistent code style with existing files

## Code Quality
- Write specific, accurate, and high-value comments; avoid unnecessary or low-value commentary
- Make changes only to files directly related to your assigned task
- Maintain code clarity and readability
- Use descriptive variable and function names
- Keep functions focused and single-purpose
- Avoid deeply nested conditionals - use early returns

## Auth0 Action Specific Guidance
- Actions execute in isolated environment with limited CPU/memory
- Keep execution time minimal (Actions timeout after a few seconds)
- Use api.multifactor.enable() and api.authentication.challengeWith() correctly
- Be aware of when Actions can redirect vs when they cannot
- Test Actions behavior thoroughly - errors in production affect all users
- Remember that event.user contains user profile data, event.authentication contains session data

## Secrets and Sensitive Information
- **NEVER commit secrets, passwords, API keys, or credentials to the repository**
- Use environment variables for all sensitive configuration
- Document required environment variables in assignment.md
- For secrets documentation, create AGENT-ASSIGNMENT-RESOURCES.MD and mark "DO NOT SHARE" at top
- Ensure AGENT-ASSIGNMENT-RESOURCES.MD is in .gitignore

## Communication Guidelines
- Ask questions when requirements or implementation details are unclear
- Notify when additional information is needed rather than making assumptions
- Do not create solutions without proper clarification when facing ambiguity
- Document architectural decisions in assignment.md
- Provide clear commit messages describing what changed and why

## Auth0 Actions Development Standards

### Module Syntax and Code Style
- **Use ES Module Syntax:** Always use ES module exports (`export function`, `export const`) instead of CommonJS pattern (`exports.foo = foo`). This aligns with modern JavaScript standards and the codebase patterns. Example: `export function isWithinGracePeriod(createdAt, gracePeriodHours = 2) { ... }` instead of `function isWithinGracePeriod(...) { ... }` followed by `exports.isWithinGracePeriod = isWithinGracePeriod`.

### File Organization for Actions
- **Event Trigger Handlers in Root of src/**: Auth0 event trigger handlers (e.g., `onExecutePostLogin`, `onExecutePreUserRegistration`) should be placed directly in the `/src` root folder in separate files. This makes them easily discoverable as main action entry points (examples: `enforceMFA.js`, `provisionAccount.js`, `customClaims.js`).
  
- **Helper Functions in /src/shared**: Utility functions and helper code that are not direct action handlers should be placed in `/src/shared` folder. This separates business logic and reusable utilities from action-specific code (examples: `request.js`, `vetEmail.js`, `maskEmail.js`, authentication/provisioning helpers).
  - Guideline: If a function is used by multiple actions or is a utility that doesn't constitute a handler itself, place it in `/src/shared`.

### Testing Patterns
- **Co-locate Tests with Source**: Test files should mirror the source structure in `/tests` directory with matching naming conventions (e.g., `src/enforceMFA.js` → `tests/enforceMFA.test.js`).
- **Export Helper Functions for Testing**: When helper functions need to be tested in isolation, export them using ES module syntax. The test file will import them naturally via `import { functionName } from '../src/file.js'`.

### Dependency Management
- **package-lock.json is Required**: Both `sources/package-lock.json` and `scripts/package-lock.json` are required and must be tracked in git. They ensure reproducible builds across environments.
- **Only Update on Verified Dependency Changes**: Do NOT update or commit package-lock.json unless there is a verified, intentional dependency change (e.g., `npm install new-package` or updating a package version). Avoid accidental modifications caused by different npm versions or environments.
- **Workflow**: 
  1. If a dependency needs updating, run `npm install` in the appropriate directory (`sources/` or `scripts/`)
  2. Verify the changes are intentional and necessary
  3. Commit the updated package-lock.json in a separate commit with a clear message describing the dependency change
  4. Never restore package-lock.json from a different branch as part of feature work unless the dependency change itself is being reverted

## Pre-Commit Checklist
Before committing any changes to Auth0 Actions:
1. [ ] Run `cd sources/ && npm run prettier:write`
2. [ ] Run `npm run lint` and fix any errors
3. [ ] Run `npm test` and ensure all tests pass
4. [ ] Review changes - no secrets or sensitive data committed
5. [ ] Update tests for any new or modified functionality
6. [ ] Verify package-lock.json changes are intentional (or not modified if no dependencies changed)
7. [ ] Add meaningful commit message
8. [ ] Consider impact - changes auto-deploy on merge to main!

*Last updated: 2026-01-14 - Created from ID-806 implementation learnings*


## Deployment

Deployments are triggered manually from GitHub Actions. Navigate to the repository's **Actions** tab, select the deploy workflow, and run it with the appropriate environment and tag parameters. No Slack bot integration — all deploys are initiated directly from GitHub.
