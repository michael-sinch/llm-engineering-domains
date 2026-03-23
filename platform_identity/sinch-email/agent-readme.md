**Repository:** git@github.com:mailgun/sinch-email.git  
**Last Updated:** 2026-02-24

# Sinch Email - AGENT README

---

## 1. LLM Agent Preamble

**Critical Setup Instructions for AI Agents:**

Before beginning any work in this repository, LLM agents must complete the following workflow:

1. **Repository Discovery**: Systematically explore the repository structure, identifying all applications, libraries, and key configuration files.
2. **Project Classification**: Confirm this is a Frontend/Hybrid monorepo with multiple TypeScript/React applications managed by NX.
3. **Architecture Mapping**: Understand the NX workspace structure, library scopes (browser/, shared/, portal/, etc.), and module boundaries enforced by ESLint.
4. **Data Source Validation**: Verify existing API integrations, service contracts, and third-party dependencies (Stripe, Split.io, RudderStack, Sentry).
5. **Documentation Review**: Read [AGENTS.md](AGENTS.md), [README.md](README.md), and relevant files in [docs/](docs/) directory.
6. **Test Organization**: Identify existing test patterns (Vitest for unit tests, Cypress for E2E) and quality gates before making changes.
7. **NX MCP Server**: Utilize the NX MCP server for enhanced workspace interactions, project graph exploration, and task execution.

**Key Principles:**
- This is a **multi-application monorepo** with 8 distinct applications
- React Compiler is enabled - generally avoid `useMemo` and `useCallback`
- Strict module boundaries are enforced by ESLint
- Always use `renderPortal` from `@sinch-email/browser/utils/component-testing` for component tests
- Follow Conventional Commits for all git operations
- Never run `git push` without explicit instruction

---

## 2. Prerequisites

**Required Tools & Versions:**
- **Node.js**: 22.14.0 (specified in [.nvmrc](.nvmrc), managed via nvm)
- **pnpm**: 10.18.1+ (specified in package.json `packageManager` field)
- **NX CLI**: Installed automatically via pnpm (v22.3.3)
- **Unix Environment**: bash or zsh (macOS/Linux)

**Optional but Recommended:**
- **NX Cloud Account**: For distributed caching and faster builds (use `@mailgun.com` or `@sinch.com` email)
- **NPM Authentication**: Required for private packages - configure `.npmrc` file with GitHub Packages token

**Setup Verification:**
```bash
node -v          # Should output: v22.14.0
pnpm -v          # Should output: 10.18.1 or higher
pnpm nx --version # Should output: 22.3.3
```

**Common Issues:**
- Missing `.npmrc`: Follow [GitHub Packages configuration guide](https://mailgun.atlassian.net/wiki/spaces/ENG/pages/4277797073/)
- Wrong Node version: Run `nvm use` or `nvm install` to activate correct version
- Build failures: Ensure all prerequisites are properly configured

---

## 3. Project Overview

**Sinch Email Monorepo** is a TypeScript/React monorepo managed by NX and pnpm. It hosts frontend code for multiple Sinch Email applications including:

- **Mailgun Portal** (default application)
- **Admin Application**
- **Login/Signup Flows**
- **Public Viewer**
- **Sinch ID**
- **Portal Bot** (Slack automation)
- **Cockpit Legacy Proxy**

**Key Characteristics:**
- **Framework**: React 18.3.1 with TypeScript 5.9.3
- **Build System**: NX 22.3.3 with Rspack (Portal) and Vite (other apps)
- **Package Manager**: pnpm 10.18.1 with workspace support
- **Monorepo Strategy**: Shared libraries organized by scope and type with strict module boundaries
- **Code Organization**: Apps in [apps/](apps/), shared libraries in [libs/](libs/), custom tooling in [tools/](tools/)

**Development Philosophy:**
- Functional components with hooks (no class components)
- React Compiler enabled for automatic optimization
- Server state via TanStack Query v5
- Client state via Jotai
- Strict TypeScript with full type safety
- Test-driven development with Vitest and Cypress

---

## 4. Classification of Assignment

**Repository Details:**
- **Repository URL**: git@github.com:mailgun/sinch-email.git
- **Classification**: Frontend/Hybrid - Multi-application TypeScript/React Monorepo
- **Primary Language**: TypeScript (strict mode enabled)
- **Framework**: React 18 with hooks
- **Architecture Pattern**: NX Monorepo with Module Federation capabilities

**Application Portfolio:**
- **portal**: Mailgun Portal (main customer-facing application)
- **admin**: Administrative dashboard for internal operations
- **login**: Authentication flow
- **signup**: User registration flow
- **public-viewer**: Public-facing email preview and tracking
- **sinch-id**: Sinch Identity integration
- **portal-bot**: Slack bot for portal automation
- **cockpit**: Legacy proxy for API routing

**Technology Stack:**
- **UI Framework**: React 18.3.1
- **Language**: TypeScript 5.9.3
- **Build Tools**: Rspack (Portal), Vite (other apps)
- **State Management**: TanStack Query v5 + Jotai
- **Styling**: styled-components 5.3.6
- **Testing**: Vitest 4.0.9 + Cypress 15.8.1
- **Monorepo Tool**: NX 22.3.3

---

## 5. System Architecture

### NX Workspace Structure

The monorepo follows NX best practices with clear separation of concerns:

```
sinch-email/
├── apps/              # Deployable applications
│   ├── portal/        # Mailgun Portal (default, uses Rspack)
│   ├── admin/         # Admin application
│   ├── login/         # Authentication UI
│   ├── signup/        # Registration UI
│   ├── public-viewer/ # Public email viewer
│   ├── sinch-id/      # Sinch Identity integration
│   ├── portal-bot/    # Slack automation bot
│   └── cockpit/       # Legacy API proxy
├── libs/              # Shared libraries
│   ├── browser/       # Browser-only code
│   ├── shared/        # Runtime-agnostic code
│   ├── portal/        # Portal-specific code
│   ├── public-viewer/ # Public viewer code
│   ├── sinch-id/      # Sinch ID code
│   └── portal-bot/    # Bot-specific code
├── tools/             # Custom NX plugins and tooling
└── docs/              # Documentation
```

### Library Organization

**Scopes** (runtime context):
- **browser/**: Browser-only libraries (DOM APIs, window object)
- **node/**: Node.js-only libraries (server-side utilities)
- **shared/**: Runtime-agnostic libraries (pure functions, utilities)
- **portal/**: Portal-specific libraries
- **public-viewer/**: Public viewer-specific libraries
- **sinch-id/**: Sinch ID-specific libraries
- **portal-bot/**: Bot-specific libraries

**Types** (functional purpose):
- **feature/**: Stateful components with business logic
- **ui/**: Presentational components (pure UI)
- **data/**: React hooks and data access patterns
- **util/**: Pure utility functions and helpers
- **service/**: API interface definitions and client code

**Example Library Path**: `@sinch-email/browser/service/account`
- Scope: `browser` (browser-only)
- Type: `service` (API interface)
- Name: `account` (Account API)

### Module Boundaries

ESLint enforces strict module boundaries via `@nx/enforce-module-boundaries`:

**Allowed Dependencies:**
- `browser` libs can import from `browser` and `shared` libs
- `shared` libs can only import from other `shared` libs
- `node` libs can import from `node` and `shared` libs
- Feature libs can import from data, ui, util, and service libs
- UI libs should **not** import from feature libs

**Constraint Enforcement:**
- Violations cause build failures
- Dependency graph must remain acyclic
- Cross-scope imports are explicitly forbidden unless allowed

### Build Architecture

**Portal Application** (apps/portal/):
- Build tool: Rspack (high-performance webpack-compatible bundler)
- Configuration: [apps/portal/rspack/rspack.config.ts](apps/portal/rspack/rspack.config.ts)
- Assets: Cockpit assets, portal public files, template images
- Default project in nx.json

**Other Applications**:
- Build tool: Vite 7.1.7
- Faster builds, better DX for non-Portal apps
- Configuration per app: `vite.config.ts`

**Development Modes:**
- **Standalone** (default): Proxies API calls to staging Cockpit
- **Development**: Uses local Cockpit Legacy Proxy instance

---

## 6. Key Components

### Applications (8 total)

#### 1. Portal ([apps/portal/](apps/portal/))
- **Purpose**: Main Mailgun Portal customer interface
- **Build Tool**: Rspack
- **Default Project**: Yes (nx.json defaultProject)
- **Key Features**: Email management, domain configuration, analytics, billing
- **Start Command**: `pnpm start` (standalone) or `pnpm start-portal-legacy` (with local Cockpit)
- **URL**: http://localhost:4200

#### 2. Admin ([apps/admin/](apps/admin/))
- **Purpose**: Internal administrative dashboard
- **Build Tool**: Vite
- **Key Features**: System configuration, user management, admin operations
- **Start Command**: `pnpm start-admin`

#### 3. Login ([apps/login/](apps/login/))
- **Purpose**: User authentication flow
- **Build Tool**: Vite
- **Key Features**: SSO integration, session management
- **Documentation**: [docs/SSO Login.md](docs/SSO%20Login.md)

#### 4. Signup ([apps/signup/](apps/signup/))
- **Purpose**: New user registration flow
- **Build Tool**: Vite
- **Key Features**: Account creation, onboarding

#### 5. Public Viewer ([apps/public-viewer/](apps/public-viewer/))
- **Purpose**: Public-facing email preview and tracking
- **Build Tool**: Vite
- **Start Command**: `pnpm start-public-viewer`

#### 6. Sinch ID ([apps/sinch-id/](apps/sinch-id/))
- **Purpose**: Sinch Identity integration and SSO
- **Build Tool**: Vite
- **Start Command**: `pnpm start-sinch-id`

#### 7. Portal Bot ([apps/portal-bot/](apps/portal-bot/))
- **Purpose**: Slack bot for portal automation and alerts
- **Dependencies**: @slack/bolt, @slack/web-api
- **E2E Test**: `pnpm alert:e2e`

#### 8. Cockpit ([apps/cockpit/](apps/cockpit/))
- **Purpose**: Legacy proxy for routing API calls
- **Info**: Required for development mode with local API
- **Integration**: Portal references Cockpit assets in build

### Library Organization

**Browser Libraries** ([libs/browser/](libs/browser/)):
- **service/**: 30+ API service integrations (account, alerts, domains, etc.)
- **feature/**: Business verification UI, support UI (micro-frontends)
- **data/**: atoms (Jotai state), hooks, mocks (MSW)
- **ui/**: Shared presentational components
- **util/**: Browser-specific utilities

**Shared Libraries** ([libs/shared/](libs/shared/)):
- **data/**: Common data access patterns, hooks
- **ui/**: Design system components, FeatureGate
- **util/**: Pure utility functions
- **service/**: Shared API interfaces

**Application-Specific Libraries**:
- **portal/**: Portal-specific features and utilities
- **public-viewer/**: Public viewer features
- **sinch-id/**: Sinch ID integration code
- **portal-bot/**: Bot-specific features

---

## 7. Development Environment

### Initial Setup

**1. Install Node.js via nvm:**
```bash
# Install/use correct Node version
nvm use  # Uses version from .nvmrc (22.14.0)

# Or install if not present
nvm install
```

**2. Configure NPM Authentication:**

Create `.npmrc` file in project root for GitHub Packages access:
```bash
# Follow guide at:
# https://mailgun.atlassian.net/wiki/spaces/ENG/pages/4277797073/
```

**3. Install Dependencies:**
```bash
pnpm install
```

**4. (Optional) Login to NX Cloud:**
```bash
pnpm nx login
# Use @mailgun.com or @sinch.com email
# Provides distributed caching and faster builds
```

### Environment Configuration

**Environment Files:**
- [.env](.env): Shared environment variables (committed)
- `.env.local`: Local overrides (not committed, see [.env.local.example](.env.local.example))
- Application-specific env files in app directories

**Required Environment Variables:**
- API endpoints for backend services
- Feature flag configuration (Split.io)
- Sentry DSN for error tracking
- Analytics keys (RudderStack)
- Stripe publishable keys

**Generate Test Keys:**
```bash
pnpm generate-env-keys  # Generates .env with 24-hour test keys
```

### Development Modes

**Portal Standalone Mode (Default):**
```bash
pnpm start
# Proxies API calls to staging Cockpit
# No local backend required
# http://localhost:4200
```

**Portal with Local Cockpit:**
```bash
# Terminal 1: Start Cockpit Legacy Proxy
cd ../cockpit && docker-compose up

# Terminal 2: Start Portal in development mode
pnpm start-portal-legacy
# Proxies to local Cockpit instance
```

**Other Applications:**
```bash
pnpm start-admin          # Admin app
pnpm start-public-viewer  # Public viewer
pnpm start-sinch-id       # Sinch ID
```

### SSO Authentication

**Login to Test Account:**
```bash
# Login and open new browser tab
pnpm sso --email=testaccount@pathwire.com

# Login to staging
pnpm sso --email=test@example.com --configuration=staging

# Open in incognito
pnpm sso --email=test@example.com --incognito

# Print link instead of opening
pnpm sso --email=test@example.com --link
```

**Documentation**: [docs/SSO Login.md](docs/SSO%20Login.md)

### Containerization

**Available Dockerfiles:**
- [apps/portal/Dockerfile](apps/portal/Dockerfile)
- [apps/admin/Dockerfile](apps/admin/Dockerfile)
- [apps/public-viewer/Dockerfile](apps/public-viewer/Dockerfile)
- [apps/portal-bot/Dockerfile](apps/portal-bot/Dockerfile)

**Docker Build:**
```bash
pnpm nx build-image portal
# Configured in nx.json with GitHub Actions integration
```

---

## 8. Build & Deployment

### Development Servers

**Start Applications:**
```bash
pnpm start                   # Portal (standalone, default)
pnpm start-portal-legacy     # Portal with local Cockpit
pnpm start-admin             # Admin app
pnpm start-public-viewer     # Public viewer
pnpm start-sinch-id          # Sinch ID app

# Direct NX commands
pnpm nx serve portal -c standalone
pnpm nx serve portal -c development
pnpm nx serve admin
```

**Development Server Features:**
- Hot Module Replacement (HMR)
- Fast Refresh for React components
- Source maps for debugging
- Automatic port assignment (Portal defaults to 4200)

### Testing

**Unit Tests (Vitest):**
```bash
# Run tests for a specific project
pnpm nx test portal
pnpm nx test <project-name>

# Skip NX cache
pnpm nx test portal --skip-nx-cache

# Run specific test file
pnpm nx test portal --testFile=path/to/test.spec.ts

# Run tests matching pattern
pnpm nx test portal -- --testNamePattern="user login"

# Test affected projects only
pnpm nx affected -t test

# Watch mode
pnpm nx test portal --watch
```

**E2E Tests (Cypress):**
```bash
# Run E2E tests
pnpm nx e2e portal-e2e
pnpm nx e2e login-e2e

# Open Cypress interactive mode
pnpm nx open-cypress portal-e2e

# Test against staging
pnpm nx e2e portal-e2e --configuration=staging

# Test against production
pnpm nx e2e portal-e2e --configuration=production

# Run affected E2E tests
pnpm nx affected -t e2e
```

**Test Environments:**
- **Local**: http://localhost:4200 (default)
- **Staging**: https://app.ninowire.com (configured in nx.json)
- **Production**: https://app.mailgun.com (configured in nx.json)

**Documentation**: [docs/Cypress Testing.md](docs/Cypress%20Testing.md)

### Linting & Formatting

**Linting:**
```bash
# Lint specific project
pnpm nx lint portal

# Lint affected projects
pnpm nx affected -t lint

# Lint all projects
pnpm nx run-many -t lint

# Auto-fix issues
pnpm nx lint portal --fix
```

**Formatting:**
```bash
# Format specific files
pnpm nx format:write --files=src/app/component.tsx

# Format all files
pnpm nx format:write

# Check formatting without fixing
pnpm nx format:check
```

**Pre-commit Hooks:**
- Husky configured in [.husky/](.husky/)
- lint-staged runs on staged files ([lint-staged.config.js](lint-staged.config.js))
- Commitlint enforces Conventional Commits ([commitlint.config.ts](commitlint.config.ts))

### Building

**Development Builds:**
```bash
# Build specific project
pnpm nx build portal

# Build with development configuration
pnpm nx build portal -c development
```

**Production Builds:**
```bash
# Production build
pnpm nx build portal -c production

# Build affected projects
pnpm nx affected -t build

# Build multiple projects
pnpm nx run-many -t build -p portal admin login
```

**Build Outputs:**
- Location: `dist/apps/<app-name>/`
- Portal uses Rspack: optimized for production
- Other apps use Vite: fast builds with tree-shaking
- Source maps generated for debugging
- Assets bundled and optimized

### CI/CD Integration

**GitHub Actions:**
- Workflows in [.github/workflows/](.github/workflows/)
- Uses NX affected commands for optimization
- Distributed task execution via NX Cloud
- Docker image builds configured in nx.json

**NX Cloud:**
- Distributed caching across CI runs
- Remote task execution
- Build analytics and insights
- Login: `pnpm nx login`

**Deployment Targets:**
- **Staging**: https://app.ninowire.com
- **Production**: https://app.mailgun.com

---

## 9. Project Structure

### Top-Level Directory Layout

```
sinch-email/
├── apps/                    # Deployable applications
├── libs/                    # Shared libraries
├── tools/                   # Custom NX plugins and generators
├── docs/                    # Documentation and guides
├── @types/                  # Global TypeScript type definitions
├── .github/                 # GitHub Actions workflows
├── .husky/                  # Git hooks
├── .nx/                     # NX cache
├── dist/                    # Build output
├── node_modules/            # Dependencies
│
├── .editorconfig            # Editor configuration
├── .env                     # Shared environment variables
├── .env.local.example       # Example local env file
├── .gitignore               # Git ignore rules
├── .nvmrc                   # Node version specification
├── .prettierrc              # Prettier configuration
├── AGENTS.md                # AI agent guide (primary reference)
├── README.md                # Project README
├── babel.config.json        # Babel configuration
├── commitlint.config.ts     # Commitlint configuration
├── eslint.config.js         # ESLint configuration
├── lint-staged.config.js    # Lint-staged configuration
├── nx.json                  # NX workspace configuration
├── package.json             # Workspace dependencies
├── pnpm-lock.yaml           # Lockfile
├── pnpm-workspace.yaml      # Workspace definition
└── tsconfig.base.json       # Base TypeScript configuration
```

### Applications Directory ([apps/](apps/))

Each application has a similar structure:

```
apps/<app-name>/
├── src/                     # Source code
│   ├── app/                 # Application components
│   ├── assets/              # Static assets
│   ├── index.ts             # Entry point
│   └── test-setup.ts        # Test configuration
├── public/                  # Public assets
├── Dockerfile               # Docker configuration
├── project.json             # NX project configuration
├── tsconfig.app.json        # App-specific TS config
├── tsconfig.json            # Base TS config
├── tsconfig.spec.json       # Test TS config
├── vite.config.ts           # Vite configuration (or rspack)
└── eslint.config.js         # ESLint configuration
```

### Libraries Directory ([libs/](libs/))

Organized by scope and type:

```
libs/
├── browser/                 # Browser-only libraries
│   ├── data/                # Data access (atoms, hooks, mocks)
│   ├── feature/             # Stateful features
│   ├── service/             # API services (30+ integrations)
│   ├── ui/                  # UI components
│   └── util/                # Browser utilities
├── shared/                  # Runtime-agnostic libraries
│   ├── data/                # Shared data patterns
│   ├── ui/                  # Shared UI components
│   ├── util/                # Pure utilities
│   └── service/             # Shared service interfaces
├── portal/                  # Portal-specific libraries
├── public-viewer/           # Public viewer libraries
├── sinch-id/                # Sinch ID libraries
└── portal-bot/              # Bot libraries
```

**Library Naming Convention**: `<scope>-<type>-<name>`
- Example: `browser-service-account`
- Import path: `@sinch-email/browser/service/account`

### Tools Directory ([tools/](tools/))

Custom NX plugins and tooling:

- **eslint-rules**: Custom ESLint rules
- **mg-login**: SSO authentication tool
- **sinch-tools**: NX generators and executors
  - Library generator: `pnpm create-lib`
  - React app generator: `pnpm create-react-app`
  - Node app generator: `pnpm create-node-app`

### Documentation Directory ([docs/](docs/))

Comprehensive guides and references:

- **API Call Authorization.md**: API authentication patterns
- **API Request Context.md**: Request context management
- **Cypress Testing.md**: E2E testing guide
- **Feature Gating in Pathwire Apps.md**: Feature flag usage
- **Migration Guide.md**: Upgrade and migration procedures
- **Package Management.md**: Dependencies and package management
- **React Hook Form Usage.md**: Form handling guide
- **SSO Login.md**: Single sign-on implementation
- **Sentry.md**: Error tracking configuration
- **Using NX.md**: NX workspace usage
- **architectural-decision-records/**: ADRs
- **frontend-best-practices/**: Coding standards
- **libraries/**: Library-specific documentation
- **testing/**: Testing strategies and patterns

---

## 10. Key Files

### Root Configuration Files

**[package.json](package.json)**
- Workspace name: `@sinch-email/root`
- Package manager: pnpm 10.18.1
- Node version: 22.14.0 (see .nvmrc)
- Scripts for common operations
- Dependencies: React 18.3.1, TypeScript 5.9.3, NX 22.3.3
- Dev dependencies: Vitest, Cypress, ESLint, Prettier

**[nx.json](nx.json)**
- Default project: `portal`
- Target defaults for build, test, e2e
- Named inputs for caching
- Plugin configuration (ESLint, Cypress, Vitest)
- Staging URL: https://app.ninowire.com
- Production URL: https://app.mailgun.com

**[tsconfig.base.json](tsconfig.base.json)**
- Compiler options: strict mode, ES2022, moduleResolution: bundler
- Path mappings for all libraries (490+ lines)
- Base configuration for all TypeScript projects
- Import aliases for apps and libs

**[eslint.config.js](eslint.config.js)**
- ESLint v9 flat config format
- TypeScript ESLint parser and plugin
- React and React Hooks plugins
- Import sorting and organization rules
- Module boundary enforcement via @nx/enforce-module-boundaries
- Custom rules from tools/eslint-rules

**[.nvmrc](.nvmrc)**
- Specifies Node.js version: 22.14.0
- Used by nvm: `nvm use`

**[pnpm-workspace.yaml](pnpm-workspace.yaml)**
- Defines workspace packages
- Patterns: apps/*, libs/*, tools/*

### Important Documentation

**[AGENTS.md](AGENTS.md)** - **PRIMARY REFERENCE FOR AI AGENTS**
- Project overview and architecture
- MCP server usage (NX MCP server available)
- Build/test/lint commands
- Project structure and library organization
- Code style guidelines
- React component patterns
- Naming conventions
- TanStack Query patterns
- Testing guidelines
- Error handling
- Commit conventions
- Module boundaries

**[README.md](README.md)**
- Quick start guide
- NX Cloud setup
- Development server instructions
- SSO authentication
- Additional resources

**[.env.local.example](.env.local.example)**
- Template for local environment variables
- Required API endpoints
- Feature flag configuration
- Service credentials

### Configuration Files

**Quality & Formatting:**
- [.prettierrc](.prettierrc): Code formatting rules (single quotes, trailing commas)
- [commitlint.config.ts](commitlint.config.ts): Conventional Commits enforcement
- [lint-staged.config.js](lint-staged.config.js): Pre-commit hook configuration
- [.editorconfig](.editorconfig): Editor settings

**Build Configuration:**
- [babel.config.json](babel.config.json): Babel configuration with React Compiler
- [apps/portal/rspack/rspack.config.ts](apps/portal/rspack/rspack.config.ts): Portal build config
- Individual `vite.config.ts` files per application

**Type Definitions:**
- [@types/](@types/): Global augmentations and custom type definitions

---

## 11. Testing Strategy

### Philosophy

**"Write tests. Not too many. Mostly integration."**

In practice, the codebase contains primarily unit tests with valuable integration tests for component interactions. The goal is pragmatic test coverage focusing on business-critical paths.

### Unit Testing with Vitest

**Test Framework**: Vitest 4.0.9
- Fast, Vite-powered test runner
- Compatible with Jest API
- Built-in code coverage via v8
- Watch mode for development

**Component Testing:**
```typescript
import { renderPortal } from '@sinch-email/browser/utils/component-testing';
import { screen } from '@testing-library/react';

test('renders domain selector', () => {
  renderPortal(<DomainSelector domains={mockDomains} />);
  
  // Prefer queries by role/label
  expect(screen.getByRole('combobox', { name: /select domain/i }))
    .toBeInTheDocument();
});
```

**Key Testing Utilities:**
- `renderPortal`: Custom render function with providers
- `@testing-library/react` 16.3.0: Component testing utilities
- `@testing-library/react-hooks` 8.0.1: Hook testing
- `@testing-library/jest-dom` 6.6.3: Custom matchers

**Query Priority** (from best to worst):
1. `getByRole` - Most accessible and robust
2. `getByLabelText` - Good for form inputs
3. `getByPlaceholderText` - Fallback for inputs
4. `getByText` - For non-interactive content
5. `getByTestId` - Last resort, avoid when possible

**API Mocking with MSW:**
```typescript
import { http, HttpResponse } from 'msw';
import { server } from '@sinch-email/browser/data/mocks';

// Setup in test file
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// Mock specific endpoint
test('handles API error', async () => {
  server.use(
    http.get('/api/accounts/me', () => {
      return HttpResponse.json({ error: 'Not found' }, { status: 404 });
    })
  );
  
  renderPortal(<AccountPage />);
  expect(await screen.findByText(/error loading account/i))
    .toBeInTheDocument();
});
```

**Running Unit Tests:**
```bash
# Run all tests for a project
pnpm nx test portal

# Skip cache
pnpm nx test portal --skip-nx-cache

# Run specific file
pnpm nx test portal --testFile=src/app/components/DomainSelector.spec.tsx

# Run tests matching pattern
pnpm nx test portal -- --testNamePattern="domain selector"

# Watch mode
pnpm nx test portal --watch

# Coverage
pnpm nx test portal --coverage

# Test affected
pnpm nx affected -t test
```

### E2E Testing with Cypress

**Test Framework**: Cypress 15.8.1
- End-to-end browser automation
- Real browser testing (Chrome, Firefox, Edge)
- Time-travel debugging
- Automatic waiting and retries

**Available E2E Projects:**
- portal-e2e
- login-e2e
- signup-e2e
- portal-bot-feature-e2e-alert

**Test Structure:**
```typescript
describe('Domain Management', () => {
  beforeEach(() => {
    cy.login('testaccount@pathwire.com');
    cy.visit('/domains');
  });

  it('should add a new domain', () => {
    cy.get('[data-cy=add-domain-button]').click();
    cy.get('[data-cy=domain-input]').type('example.com');
    cy.get('[data-cy=submit-button]').click();
    
    cy.contains('Domain added successfully').should('be.visible');
    cy.contains('example.com').should('exist');
  });
});
```

**Running E2E Tests:**
```bash
# Run E2E tests headlessly
pnpm nx e2e portal-e2e

# Open Cypress interactive mode
pnpm nx open-cypress portal-e2e

# Test against staging
pnpm nx e2e portal-e2e --configuration=staging

# Test against production (read-only tests only)
pnpm nx e2e portal-e2e --configuration=production

# Run affected E2E tests
pnpm nx affected -t e2e
```

**Test Environments:**
- **Local**: Default, tests against http://localhost:4200
- **Staging**: https://app.ninowire.com (baseUrl in nx.json)
- **Production**: https://app.mailgun.com (baseUrl in nx.json)

**Documentation**: [docs/Cypress Testing.md](docs/Cypress%20Testing.md)

### Test Organization

**Test File Naming:**
- Unit tests: `*.spec.ts`, `*.spec.tsx`
- E2E tests: `*.cy.ts`
- Located alongside source files or in `__tests__` directories

**Test Setup Files:**
- `src/test-setup.ts` in each project
- Configures Testing Library, jest-dom matchers
- Sets up MSW server if needed

**Coverage Goals:**
- Focus on business-critical paths
- Avoid testing implementation details
- Test user-facing behavior
- Aim for meaningful coverage, not percentage

### Testing Best Practices

**From [AGENTS.md](AGENTS.md):**
1. Use `renderPortal` for all component tests
2. Prefer `getByRole` over `getByTestId`
3. Use MSW for API mocking, not hand-rolled mocks
4. Test user behavior, not implementation
5. Keep tests simple and readable
6. Avoid testing third-party library internals
7. Use `useSuspenseQuery` with `<SuspenseBoundary>` for loading states

---

## 12. Security & Compliance

### Authentication & Authorization

**Single Sign-On (SSO):**
- Integration with Sinch ID for unified authentication
- OAuth 2.0 / OpenID Connect flows
- Session management via secure cookies
- Documentation: [docs/SSO Login.md](docs/SSO%20Login.md)

**API Authorization:**
- API calls authorized via Bearer tokens
- Tokens obtained through authentication flow
- Request context includes user and account information
- Documentation: [docs/API Call Authorization.md](docs/API%20Call%20Authorization.md)

**Role-Based Access Control (RBAC):**
- User roles: admin, basic, billing, developer, sending, support
- Capabilities enforced by Cerberus service
- Feature gating based on user capabilities
- Capability checking via `useFeature` hook

**Feature Gating:**
- **Capability Gates**: Based on user role and permissions
- **Plan Gates**: Based on account billing plan
- **Aspect Gates**: Based on user actor aspects (Theater service)
- **Dev Gates**: Environment-based feature flags
- Documentation: [docs/Feature Gating in Pathwire Apps.md](docs/Feature%20Gating%20in%20Pathwire%20Apps.md)

### Error Tracking & Monitoring

**Sentry Integration:**
- Error tracking in production and staging
- React error boundary integration
- Source maps for debugging production errors
- Performance monitoring
- Release tracking
- Documentation: [docs/Sentry.md](docs/Sentry.md)

**Sentry Setup:**
```typescript
import * as Sentry from '@sentry/react';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  integrations: [/* ... */],
  tracesSampleRate: 1.0,
});
```

**Configuration:**
- Package: @sentry/react 10.17.0
- Build plugin: @sentry/vite-plugin 4.3.0
- Source map upload in production builds

### Code Quality

**ESLint Configuration:**
- TypeScript strict checking enabled
- React and React Hooks rules
- Import ordering and organization
- Module boundary enforcement
- Custom rules in tools/eslint-rules

**Prettier Formatting:**
- Single quotes for strings
- Trailing commas everywhere
- 2-space indentation
- Line length: 100 characters
- Auto-formatting on save (recommended)

**Pre-commit Hooks (Husky):**
- Lint staged files
- Run Prettier formatting
- Type checking
- Conventional Commit validation

**Conventional Commits:**
- Enforced via commitlint
- Format: `<type>(<scope>): <description>`
- Types: feat, fix, docs, test, refactor, style, build, ci, chore, perf
- Breaking changes: `!` after type/scope

### Dependency Management

**Private Package Access:**
- GitHub Packages for internal npm packages
- `.npmrc` configuration required
- Authentication via GitHub token

**Dependency Updates:**
- Regular security updates
- Renovate bot for automated PRs
- Manual review required for major versions

**License Compliance:**
- MIT license for workspace
- Third-party license checking in builds
- Extract licenses in production builds

---

## 13. Monitoring & Operations

### Development Tools

**NX Console:**
- VS Code extension for NX operations
- Visual task runner
- Project graph visualization
- Generator UI for scaffolding

**React DevTools:**
- Component hierarchy inspection
- Props and state debugging
- Performance profiling

**TanStack Query DevTools:**
- Query cache inspection
- Mutation tracking
- Automatic refetch debugging
- Available in development builds

**Browser DevTools:**
- Source maps for debugging production builds
- Network inspection for API calls
- Performance profiling
- React Profiler integration

### Error Handling

**React Error Boundaries:**
- Catch rendering errors
- Fallback UI display
- Error reporting to Sentry
- Component: `react-error-boundary` 6.0.0

**TanStack Query Error Handling:**
- Automatic error states in queries
- Global error handling via mutation meta
- Toast notifications for user-facing errors
- Retry logic for transient failures

**Zod Runtime Validation:**
- Schema validation for API responses
- Type-safe parsing with error messages
- Version 4.0.15 with enhanced features

**Global Error Handling:**
```typescript
// Mutation with error handling
export const accountsMutations = {
  updateAccount: mutationOptions({
    meta: {
      errorMessage: 'Unable to update account.',
      successMessage: 'Successfully updated account.',
      invalidates: [accountsQueries.baseKeys.account()],
    },
    mutationFn: updateAccount,
  }),
};
```

### Logging

**Development Logging:**
- Console logs for debugging
- Redux DevTools for state inspection (if applicable)
- Network logs in browser DevTools

**Production Logging:**
- Structured error logging to Sentry
- Performance metrics
- User interaction tracking (RudderStack)
- Minimal console output

### Analytics & Telemetry

**RudderStack:**
- User behavior analytics
- Event tracking
- Conversion funnel analysis
- Package: @rudderstack/analytics-js 3.5.2

**Split.io Feature Flags:**
- A/B testing
- Feature rollouts
- Targeting based on user attributes
- Package: @splitsoftware/splitio-react 1.11.1

**NX Cloud Analytics:**
- Build performance metrics
- Cache hit rates
- Task execution times
- Distributed task insights

---

## 14. Technical Stack & Dependencies

### Core Framework

**React 18.3.1:**
- Functional components with hooks
- Concurrent features (Suspense, Transitions)
- Automatic batching
- React Compiler enabled for optimization

**TypeScript 5.9.3:**
- Strict mode enabled
- Full type safety
- Path mapping for imports
- Shared base configuration

### State Management

**TanStack Query 5.90.2:**
- Server state management
- Automatic caching and refetching
- Optimistic updates
- Suspense integration
- DevTools: @tanstack/react-query-devtools

**Jotai 2.6.5:**
- Atomic state management
- Primitive and flexible
- TypeScript-first
- No boilerplate
- Used for client-side UI state

**React Context:**
- Scoped state for component trees
- Authentication context
- Theme context

### Routing

**React Router:**
- react-router-dom 5.3.4 (legacy apps)
- react-router-dom-v5-compat 6.21.3 (migration path)
- Nested routes
- Protected routes with authentication

### UI & Styling

**styled-components 5.3.6:**
- CSS-in-JS solution
- Theme support
- Dynamic styling
- SSR compatible (if needed)
- Babel plugin for optimization

**Nectary Design System:**
- @nectary/components 5.15.4
- @nectary/assets 3.4.2
- @nectary/theme-base 1.8.0
- @nectary/theme-cpaas-base 1.2.0
- @nectary/theme-cpaas-dashboard 1.3.0
- @nectary/theme-dark 1.2.1

**Mailgun Components:**
- @mailgun/mailjet-react-components 8.0.0

**Fonts:**
- @fontsource/dm-mono 5.0.19
- @fontsource/dm-sans 5.1.0
- @fontsource/roboto 4.5.8

### Forms & Validation

**react-hook-form 7.62.0:**
- Performant form handling
- Minimal re-renders
- TypeScript support
- Documentation: [docs/React Hook Form Usage.md](docs/React%20Hook%20Form%20Usage.md)

**Zod 4.0.15:**
- Schema validation
- Type inference
- Runtime type safety
- Integration: @hookform/resolvers 5.2.1

**Form Utilities:**
- @hookform/error-message 2.0.1
- Field-level validation
- Async validation support

### Data Handling

**axios 1.10.0:**
- HTTP client
- Interceptors for auth
- Request/response transforms
- Error handling

**date-fns 4.1.0:**
- Modern date utility library
- Tree-shakeable
- TypeScript support
- Timezone support: date-fns-tz 3.2.0

**lodash 4.17.21:**
- Utility functions
- Prefer native JS when possible
- Use specific imports for tree-shaking

**ts-deepmerge 6.0.2:**
- Type-safe deep merging

**ts-pattern 5.1.2:**
- Pattern matching for TypeScript
- Exhaustive checking

### Charting & Visualization

**recharts 2.15.0:**
- React charting library
- Declarative API
- Responsive charts
- Composable components

**codemirror 5.65.0:**
- Code editor component
- Syntax highlighting
- React integration: react-codemirror2 7.2.1

### Third-Party Integrations

**Stripe:**
- @stripe/react-stripe-js 2.4.0
- @stripe/stripe-js 2.2.0
- Payment processing
- Subscription management

**Split.io:**
- @splitsoftware/splitio-react 1.11.1
- Feature flags
- A/B testing
- Targeting

**RudderStack:**
- @rudderstack/analytics-js 3.5.2
- Event tracking
- User analytics
- Data pipeline

**Slack (Portal Bot):**
- @slack/bolt 3.22.0
- @slack/web-api 6.13.0
- Bot framework
- API integration

**Vimeo:**
- @vimeo/player 2.21.0
- Video player integration

**Sentry:**
- @sentry/react 10.17.0
- Error tracking
- Performance monitoring

### Build & Development Tools

**NX 22.3.3:**
- Monorepo management
- Task orchestration
- Caching and optimization
- Affected commands
- Cloud: nx-cloud 19.1.0

**Build Tools:**
- Rspack (Portal): Webpack-compatible, high-performance
- Vite 7.1.7 (other apps): Fast build tool
- esbuild 0.19.5: Fast JavaScript bundler

**Compilers:**
- @swc/core 1.5.7: Fast TypeScript/JavaScript compiler
- babel-plugin-react-compiler 1.0.0: React Compiler plugin

**Testing:**
- vitest 4.0.9: Unit test framework
- @vitest/coverage-v8 4.0.9: Coverage tool
- @vitest/ui 4.0.9: Test UI
- cypress 15.8.1: E2E testing
- @testing-library/react 16.3.0: Component testing
- @testing-library/jest-dom 6.6.3: Custom matchers
- @testing-library/dom 10.4.0: DOM utilities
- msw 2.7.5: API mocking

**Code Quality:**
- eslint 9.39.1: Linting
- typescript-eslint 8.46.4: TypeScript ESLint
- prettier 3.6.2: Code formatting
- husky 9.1.7: Git hooks
- lint-staged 15.4.3: Staged file linting

---

## 15. Service Contract

### Overview

As a **frontend monorepo**, Sinch Email does not expose service contracts but rather **consumes** API contracts from backend services. All API integrations are typed and documented in the `libs/browser/service/` and `libs/shared/service/` directories.

### Backend Service Integrations

The repository integrates with 30+ backend services, including:

**Core Services:**
- **Accounts**: User accounts and billing
- **Cerberus**: Authentication and RBAC
- **Theater**: User actors and aspects
- **Cockpit**: Legacy API proxy and routing

**Email Services:**
- **Domains**: Domain management and verification
- **Analytics**: Email analytics and reporting
- **Alerts**: Alert configuration and management
- **Webhooks**: Webhook management
- **Routes**: Routing rules
- **Mailboxes**: Mailbox management

**Infrastructure Services:**
- **Argus**: Monitoring
- **Augur**: Predictions
- **Bandersnatch**: ???
- **Blackbook**: Contact management
- **Cashier**: Payment processing
- **Censor**: Content filtering
- **Vulcand**: Load balancing

**Additional Services:**
- Bounce Classification
- DBL Monitor (Domain Blocklist)
- DMARC Management
- Email Preview
- Entri Integration (domain verification)
- Exporter
- Suppression Management
- IP Management
- Templates
- And many more...

### Service Client Pattern

**Query Pattern (TanStack Query):**
```typescript
// libs/browser/service/account/src/lib/queries.ts
export const accountsQueries = {
  baseKeys: {
    all: () => ['accounts'],
    account: () => [...accountsQueries.baseKeys.all(), 'account'],
    subaccounts: () => [...accountsQueries.baseKeys.all(), 'subaccounts'],
  },
  getAccount: (...params: Parameters<typeof getAccountMe>) =>
    queryOptions({
      queryFn: () => getAccountMe(...params),
      queryKey: [...accountsQueries.baseKeys.account(), 
                  ...getRequestContextQueryKeyParams(params)],
    }),
} as const;
```

**Mutation Pattern:**
```typescript
// libs/browser/service/account/src/lib/mutations.ts
export const accountsMutations = {
  updateAccount: mutationOptions({
    meta: {
      errorMessage: 'Unable to update account.',
      successMessage: 'Successfully updated account.',
      invalidates: [accountsQueries.baseKeys.account()],
    },
    mutationFn: updateAccount,
  }),
};
```

**Key Principles:**
- Use `baseKeys` for invalidation, not `*.queryKey`
- Include request context in query keys
- Provide user-facing error/success messages in meta
- Type-safe with full TypeScript inference

**Documentation**: [AGENTS.md](AGENTS.md) - TanStack Query Patterns section

### API Request Context

**Request Context Pattern:**
- All API calls include request context (user, account, session)
- Context passed as part of params object
- Documented in [docs/API Request Context.md](docs/API%20Request%20Context.md)

### Type Safety

**API Response Typing:**
- Zod schemas for runtime validation
- TypeScript interfaces for compile-time checking
- Automatic type inference from schemas
- Validation errors surfaced to users

---

## 16. Core Business Functions

### Mailgun Portal (Primary Application)

**Email Campaign Management:**
- Send transactional and marketing emails
- Template management with variable substitution
- Batch sending with rate limiting
- Scheduling and automation

**Domain Management:**
- Add and verify domains
- DNS configuration assistance
- Entri integration for automatic verification
- SPF, DKIM, DMARC setup
- Domain reputation monitoring

**Analytics & Reporting:**
- Email delivery statistics
- Open and click tracking
- Bounce and complaint analysis
- Real-time dashboards with Recharts
- Historical data analysis
- Export capabilities

**Recipient Management:**
- Mailing list management
- Suppression lists (bounces, complaints, unsubscribes)
- Contact validation
- Bulk operations

**Webhooks & Integrations:**
- Configure webhooks for email events
- Custom routing rules
- API credential management
- Rate limiting configuration

**Billing & Subscriptions:**
- Stripe integration for payments
- Plan management and upgrades
- Usage tracking and invoicing
- Billing history

**Team Management:**
- User invitations
- Role-based permissions (admin, developer, billing, etc.)
- Capability management via Cerberus
- Audit logging

### Admin Application

**System Administration:**
- Internal operations dashboard
- Account management
- System configuration
- User support tools

**Account Operations:**
- Account status management
- Usage monitoring
- Compliance tools
- Support ticket integration

### Authentication Suite

**Login Application:**
- SSO via Sinch ID
- OAuth 2.0 / OpenID Connect
- Session management
- Password recovery
- Multi-factor authentication (if enabled)

**Signup Application:**
- New user registration
- Email verification
- Onboarding flow
- Initial account setup
- Plan selection

**Sinch ID Integration:**
- Unified identity across Sinch products
- Cross-application SSO
- Centralized user management

### Public Viewer

**Email Preview:**
- Public-facing email view
- Tracking pixel support
- Click tracking
- Responsive email rendering

### Portal Bot (Slack Integration)

**Slack Automation:**
- Alert notifications in Slack
- Bot commands for common operations
- Integration with portal APIs
- E2E testing: `pnpm alert:e2e`

### Feature Gating System

**Capability-Based Access:**
- Role-based feature visibility
- Account capability checking
- Dynamic UI adaptation

**Plan-Based Features:**
- Billing plan feature enforcement
- Custom plan features
- Feature overrides by support

**Dev Feature Flags:**
- Environment-based features (staging vs production)
- Gradual rollouts
- A/B testing via Split.io

**Usage:**
```typescript
import { FeatureGate } from '@sinch-email/shared/ui/components';

<FeatureGate features={{ dev: 'MyFeature' }}>
  <NewFeatureComponent />
</FeatureGate>
```

**Documentation**: [docs/Feature Gating in Pathwire Apps.md](docs/Feature%20Gating%20in%20Pathwire%20Apps.md)

---

## 17. Configuration

### Environment Variables

**Shared Environment** ([.env](.env)):
- Committed to repository
- Shared across all applications
- Non-sensitive configuration

**Local Environment** (`.env.local`):
- Not committed (in .gitignore)
- Local overrides and secrets
- Template: [.env.local.example](.env.local.example)

**Application-Specific:**
- Environment files in app directories
- Override shared configuration
- Application-specific settings

**Key Variables:**
```bash
# API Endpoints
API_BASE_URL=https://api.mailgun.net

# Feature Flags
SPLIT_IO_KEY=your_key_here

# Error Tracking
SENTRY_DSN=your_dsn_here

# Analytics
RUDDERSTACK_WRITE_KEY=your_key_here

# Payments
STRIPE_PUBLISHABLE_KEY=pk_test_...
```

**Generate Test Keys:**
```bash
pnpm generate-env-keys
# Generates .env with 24-hour test keys
# Required for E2E testing
```

### NPM Configuration

**Private Package Access** (`.npmrc`):
- Required for GitHub Packages
- Authentication via GitHub token
- Not committed to repository
- Documentation: [GitHub Packages Setup](https://mailgun.atlassian.net/wiki/spaces/ENG/pages/4277797073/)

**Example `.npmrc`:**
```
@mailgun:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=YOUR_GITHUB_TOKEN
```

### NX Configuration

**[nx.json](nx.json):**
- Default project: portal
- Target defaults (build, test, e2e)
- Caching strategy
- E2E environment URLs:
  - Staging: https://app.ninowire.com
  - Production: https://app.mailgun.com

**NX Cloud:**
- Login: `pnpm nx login`
- Use @mailgun.com or @sinch.com email
- Distributed caching
- Remote task execution

### TypeScript Configuration

**Base Configuration** ([tsconfig.base.json](tsconfig.base.json)):
- Strict mode enabled
- ES2022 target
- Module resolution: bundler
- Path mappings for all libraries

**Project-Specific:**
- `tsconfig.app.json`: Application code
- `tsconfig.spec.json`: Test files
- Extends base configuration

### ESLint Configuration

**[eslint.config.js](eslint.config.js):**
- Flat config format (ESLint v9)
- TypeScript strict checking
- React and React Hooks rules
- Import organization
- Module boundary enforcement
- Custom rules in tools/eslint-rules

**Module Boundaries:**
- Enforced via @nx/enforce-module-boundaries
- Scope-based constraints (browser, shared, node)
- Type-based constraints (feature, ui, data, util, service)

### Editor Configuration

**[.editorconfig](.editorconfig):**
- Consistent coding style across editors
- Indentation: 2 spaces
- Charset: UTF-8
- Line endings: LF

**VS Code Settings** ([.vscode/](.vscode/)):
- Workspace-specific settings
- Extension recommendations
- Debug configurations

### Git Configuration

**Hooks** ([.husky/](.husky/)):
- Pre-commit: lint-staged
- Commit-msg: commitlint

**Commitlint** ([commitlint.config.ts](commitlint.config.ts)):
- Conventional Commits format
- Type validation
- Scope validation

**Lint-staged** ([lint-staged.config.js](lint-staged.config.js)):
- Lint modified files
- Format with Prettier
- Type check

---

## 18. Critical Commands

### Development Commands

**Start Applications:**
```bash
# Portal (standalone mode - default)
pnpm start
# Same as: pnpm nx serve portal -c standalone

# Portal with local Cockpit
pnpm start-portal-legacy
# Same as: pnpm nx serve portal -c development

# Other applications
pnpm start-admin
pnpm start-public-viewer
pnpm start-sinch-id

# Direct NX command
pnpm nx serve <app-name>
pnpm nx serve <app-name> -c <configuration>
```

**SSO Authentication:**
```bash
# Login to test account
pnpm sso --email=testaccount@pathwire.com

# Login to staging
pnpm sso --email=test@example.com --configuration=staging

# Open in incognito
pnpm sso --email=test@example.com --incognito

# Print link instead of opening
pnpm sso --email=test@example.com --link

# Use ?key parameter instead of /sessions/sso
pnpm sso --email=test@example.com --key
```

### Testing Commands

**Unit Tests (Vitest):**
```bash
# Run all tests for a project
pnpm nx test <project-name>

# Skip NX cache
pnpm nx test <project-name> --skip-nx-cache

# Run specific test file
pnpm nx test <project-name> --testFile=path/to/test.spec.ts

# Run tests matching pattern
pnpm nx test <project-name> -- --testNamePattern="<pattern>"

# Watch mode
pnpm nx test <project-name> --watch

# Coverage
pnpm nx test <project-name> --coverage

# Test only affected projects
pnpm nx affected -t test
```

**E2E Tests (Cypress):**
```bash
# Run E2E tests
pnpm nx e2e <project-name>-e2e

# Open Cypress interactive mode
pnpm nx open-cypress <project-name>-e2e

# Test against staging
pnpm nx e2e <project-name>-e2e --configuration=staging

# Test against production
pnpm nx e2e <project-name>-e2e --configuration=production

# Run affected E2E tests
pnpm nx affected -t e2e

# Portal bot alert test
pnpm alert:e2e
```

**Generate Test Keys:**
```bash
# Generate .env with test keys (valid 24 hours)
pnpm generate-env-keys
```

### Quality Commands

**Linting:**
```bash
# Lint specific project
pnpm nx lint <project-name>

# Auto-fix issues
pnpm nx lint <project-name> --fix

# Lint affected projects
pnpm nx affected -t lint

# Lint all projects
pnpm nx run-many -t lint
```

**Formatting:**
```bash
# Format specific files
pnpm nx format:write --files=path/to/file.ts

# Format all files
pnpm nx format:write

# Check formatting without fixing
pnpm nx format:check
```

**Type Checking:**
```bash
# Type check specific project
pnpm nx typecheck <project-name>

# Type check affected
pnpm nx affected -t typecheck
```

### Build Commands

**Development Builds:**
```bash
# Build project
pnpm nx build <project-name>

# Build with development configuration
pnpm nx build <project-name> -c development
```

**Production Builds:**
```bash
# Production build
pnpm nx build <project-name> -c production

# Build affected projects
pnpm nx affected -t build

# Build multiple projects
pnpm nx run-many -t build -p portal admin login

# Build all projects
pnpm nx run-many -t build
```

**Docker Image Builds:**
```bash
# Build container image
pnpm nx build-image <project-name>
```

### NX Tools Commands

**Dependency Graph:**
```bash
# View project dependency graph
pnpm nx dep-graph

# View affected project graph
pnpm nx affected:graph
```

**NX Cloud:**
```bash
# Login to NX Cloud
pnpm nx login

# View task history
pnpm nx view-logs
```

**Generators:**
```bash
# Generate new library
pnpm create-lib <library-name>
# Same as: pnpm nx generate @sinch-email/sinch-tools:library

# Generate React application
pnpm create-react-app <app-name>

# Generate Node application
pnpm create-node-app <app-name>

# List available generators
pnpm nx list @sinch-email/sinch-tools
```

**Project Information:**
```bash
# Show project details
pnpm nx show project <project-name>

# List all projects
pnpm nx show projects

# Show task details
pnpm nx show project <project-name> --web
```

### Run Many Commands

```bash
# Run target for multiple projects
pnpm nx run-many -t <target>

# Examples:
pnpm nx run-many -t build
pnpm nx run-many -t test
pnpm nx run-many -t lint

# Specify projects
pnpm nx run-many -t build -p portal admin login

# Parallel execution (default)
pnpm nx run-many -t test --parallel=3

# Skip cache
pnpm nx run-many -t build --skip-nx-cache
```

### Affected Commands

```bash
# Run target on affected projects
pnpm nx affected -t <target>

# Examples:
pnpm nx affected -t build
pnpm nx affected -t test
pnpm nx affected -t lint
pnpm nx affected -t e2e

# See what's affected
pnpm nx affected:graph

# Base comparison (default: main)
pnpm nx affected -t test --base=origin/main

# Compare specific commits
pnpm nx affected -t test --base=HEAD~1 --head=HEAD
```

### Maintenance Commands

**Dependencies:**
```bash
# Install dependencies
pnpm install

# Update dependencies
pnpm update

# Clean install
rm -rf node_modules && pnpm install
```

**Cache Management:**
```bash
# Clear NX cache
pnpm nx reset

# Clear all caches
rm -rf node_modules .nx dist && pnpm install
```

---

## 19. React/TypeScript Specific Best Practices

### Component Patterns

**Functional Components (Required):**
```typescript
// ✅ Correct: const arrow function
export const UserProfile = ({ userId }: Props) => {
  const [user, setUser] = useState<User | null>(null);
  
  useEffect(() => {
    // Effect logic
  }, [userId]);
  
  return <div>{/* JSX */}</div>;
};

// ❌ Incorrect: class component
export class UserProfile extends React.Component {
  // No class components
}
```

**React Compiler Enabled:**
```typescript
// ✅ React Compiler is enabled
// useMemo and useCallback are generally NOT needed
export const ExpensiveComponent = ({ data }: Props) => {
  // No need for useMemo - React Compiler optimizes automatically
  const processedData = expensiveOperation(data);
  
  // No need for useCallback - React Compiler optimizes automatically
  const handleClick = () => {
    console.log('clicked');
  };
  
  return <Button onClick={handleClick}>{processedData}</Button>;
};

// ⚠️ Only use manual memoization if profiling shows it's needed
```

**Props Handling:**
```typescript
// ✅ Correct: explicit props
<Button onClick={handleClick} disabled={isLoading} />

// ❌ Incorrect: prop spreading
<Button {...props} />
```

**Component Organization:**
```typescript
// One component per file (multiple small helpers allowed)
export const DomainSelector = ({ domains }: Props) => {
  // 1. Hooks at the top
  const [selected, setSelected] = useState<string>();
  const { data: account } = useSuspenseQuery(accountsQueries.getAccount());
  
  // 2. Effects after hooks
  useEffect(() => {
    // Effect logic
  }, [selected]);
  
  // 3. Event handlers
  const handleSelect = (domain: string) => {
    setSelected(domain);
  };
  
  // 4. Helper functions (or extract to file if reused)
  const filterDomains = (search: string) => {
    return domains.filter(d => d.name.includes(search));
  };
  
  // 5. Early returns
  if (!domains.length) {
    return <EmptyState />;
  }
  
  // 6. Main render
  return <div>{/* JSX */}</div>;
};
```

### Suspense and Data Fetching

**Use Suspense Boundaries:**
```typescript
import { SuspenseBoundary } from '@sinch-email/shared/ui/components';

// ✅ Wrap data-fetching components in SuspenseBoundary
export const AccountPage = () => {
  return (
    <SuspenseBoundary>
      <AccountDetails />
    </SuspenseBoundary>
  );
};

// Child component uses useSuspenseQuery
const AccountDetails = () => {
  const { data: account } = useSuspenseQuery(
    accountsQueries.getAccount()
  );
  
  return <div>{account.name}</div>;
};
```

**Prefer useSuspenseQuery over useQuery:**
```typescript
// ✅ Preferred: useSuspenseQuery with SuspenseBoundary
const { data } = useSuspenseQuery(accountsQueries.getAccount());
// No need for loading state, Suspense handles it

// ⚠️ Use useQuery only when you need fine-grained loading control
const { data, isLoading } = useQuery(accountsQueries.getAccount());
```

### TypeScript Patterns

**Type Imports:**
```typescript
// ✅ Correct: inline type imports
import { type User, type Account } from '@sinch-email/shared/types';

// ❌ Incorrect: separate import type
import type { User } from '@sinch-email/shared/types';
```

**Const Assertions:**
```typescript
// ✅ Use const assertion for immutable objects
export const ROUTES = {
  home: '/',
  domains: '/domains',
  settings: '/settings',
} as const;
```

**Generic Components:**
```typescript
// ✅ Type-safe generic components
interface TableProps<T> {
  data: T[];
  renderRow: (item: T) => ReactNode;
}

export const Table = <T,>({ data, renderRow }: TableProps<T>) => {
  return (
    <table>
      {data.map(renderRow)}
    </table>
  );
};
```

### State Management Patterns

**TanStack Query for Server State:**
```typescript
// queries.ts
export const domainsQueries = {
  baseKeys: {
    all: () => ['domains'],
    list: () => [...domainsQueries.baseKeys.all(), 'list'],
    detail: () => [...domainsQueries.baseKeys.all(), 'detail'],
  },
  getDomains: (...params: Parameters<typeof getDomains>) =>
    queryOptions({
      queryFn: () => getDomains(...params),
      queryKey: [...domainsQueries.baseKeys.list(), ...params],
    }),
} as const;

// mutations.ts
export const domainsMutations = {
  createDomain: mutationOptions({
    meta: {
      errorMessage: 'Unable to create domain.',
      successMessage: 'Domain created successfully.',
      invalidates: [domainsQueries.baseKeys.list()],
    },
    mutationFn: createDomain,
  }),
};

// Component usage
const { data: domains } = useSuspenseQuery(domainsQueries.getDomains());
const createMutation = useMutation(domainsMutations.createDomain);
```

**Jotai for Client State:**
```typescript
// atoms.ts
import { atom } from 'jotai';

export const sidebarOpenAtom = atom(false);
export const selectedDomainAtom = atom<string | null>(null);

// Component usage
import { useAtom, useSetAtom } from 'jotai';

const [isOpen, setIsOpen] = useAtom(sidebarOpenAtom);
const setSelectedDomain = useSetAtom(selectedDomainAtom);
```

### Form Handling

**React Hook Form with Zod:**
```typescript
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

// Schema separate from defaultValues
const schema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
  sendingEnabled: z.boolean(),
});

type FormData = z.infer<typeof schema>;

// Zero values for defaultValues (no undefined)
const DEFAULT_VALUES: FormData = {
  email: '',
  name: '',
  sendingEnabled: false,
};

export const UserForm = () => {
  const { control, handleSubmit, reset } = useForm<FormData>({
    resolver: zodResolver(schema),
    defaultValues: DEFAULT_VALUES,
  });
  
  const onSubmit = (data: FormData) => {
    // Handle submission
  };
  
  const handleReset  = () => {
    reset(DEFAULT_VALUES); // Re-pass defaultValues
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* Form fields */}
    </form>
  );
};
```

**Documentation**: [docs/React Hook Form Usage.md](docs/React%20Hook%20Form%20Usage.md)

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Components | PascalCase | `DomainSelector`, `UserProfile` |
| Files (components) | PascalCase | `DomainSelector.tsx` |
| Files (utilities) | camelCase | `dateUtils.ts`, `validators.ts` |
| Functions/variables | camelCase | `getUserData`, `selectedDomain` |
| Constants | UPPER_CASE | `API_ROUTES`, `DEFAULT_TIMEOUT` |
| Types/Interfaces | PascalCase | `User`, `DomainConfig` |
| Hooks | `use` prefix | `useActiveDomain`, `useAuth` |
| Query objects | `<entity>Queries` | `accountsQueries`, `domainsQueries` |
| Mutation objects | `<entity>Mutations` | `accountsMutations` |
| Atoms (Jotai) | `<name>Atom` | `sidebarOpenAtom` |

### Import Organization

**Automatic Sorting (Prettier):**
```typescript
// 1. External packages (alphabetized)
import { useEffect, useState } from 'react';
import { useSuspenseQuery } from '@tanstack/react-query';
import { z } from 'zod';

// 2. Internal packages (@sinch-email/*)
import { accountsQueries } from '@sinch-email/browser/service/account';
import { Button } from '@sinch-email/shared/ui/components';

// 3. Relative imports
import { type FormData } from './types';
import { validateInput } from './validators';
```

### Error Handling

**Zod for Runtime Validation:**
```typescript
const responseSchema = z.object({
  id: z.string(),
  name: z.string(),
  createdAt: z.string().datetime(),
});

try {
  const data = responseSchema.parse(apiResponse);
} catch (error) {
  // Handle validation error
  if (error instanceof z.ZodError) {
    console.error(error.issues);
  }
}
```

**Error Boundaries:**
```typescript
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary
  fallback={<ErrorFallback />}
  onError={(error) => {
    // Log to Sentry
  }}
>
  <App />
</ErrorBoundary>
```

### Testing Patterns

**Component Tests:**
```typescript
import { renderPortal } from '@sinch-email/browser/utils/component-testing';
import { screen } from '@testing-library/react';
import { userEvent } from '@testing-library/user-event';

test('submits form with valid data', async () => {
  const user = userEvent.setup();
  const onSubmit = vi.fn();
  
  renderPortal(<DomainForm onSubmit={onSubmit} />);
  
  await user.type(
    screen.getByRole('textbox', { name: /domain name/i }),
    'example.com'
  );
  
  await user.click(
    screen.getByRole('button', { name: /add domain/i })
  );
  
  expect(onSubmit).toHaveBeenCalledWith({ name: 'example.com' });
});
```

---

## 20. General Guidance for LLM Agents

### Git Workflow

**Critical Rules:**
1. **Always ask before making commits** - Never commit without explicit user confirmation
2. **Never run `git push`** unless explicitly instructed by the user
3. **Follow Conventional Commits format**: `<type>(<scope>): <description>`

**Commit Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation only
- `test`: Adding or updating tests
- `refactor`: Code refactoring without behavior change
- `style`: Formatting, whitespace (no code change)
- `build`: Build system or dependency changes
- `ci`: CI configuration changes
- `chore`: Miscellaneous (no production code change)
- `perf`: Performance improvement

**Commit Examples:**
```bash
git commit -m "feat(portal): add domain verification UI"
git commit -m "fix(admin): resolve account filtering bug"
git commit -m "docs(readme): update setup instructions"
git commit -m "test(domains): add unit tests for DomainSelector"
```

**Breaking Changes:**
```bash
git commit -m "feat(api)!: change authentication flow"
# or
git commit -m "feat(api): change authentication flow

BREAKING CHANGE: API now requires OAuth 2.0"
```

### NX MCP Server Usage

**The NX MCP Server is available** for enhanced workspace interactions:

**Capabilities:**
- Project graph exploration
- Task execution
- Generator invocation
- Dependency analysis
- Affected project detection

**When to Use:**
- Exploring project structure
- Understanding dependencies
- Discovering available tasks
- Generating new code
- Running NX commands programmatically

**Example Usage Scenarios:**
- "Show me all projects that depend on @sinch-email/browser/service/account"
- "Generate a new feature library for domain management"
- "Run tests for affected projects"
- "View the dependency graph"

### Code Generation Strategy

**Before Generating Code:**
1. Read existing patterns in similar code
2. Check [AGENTS.md](AGENTS.md) for guidelines
3. Review relevant documentation in [docs/](docs/)
4. Understand the module boundary constraints
5. Identify the correct library scope and type

**Code Generation Principles:**
1. Follow existing patterns consistently
2. Use TypeScript strict mode
3. Prefer functional components with hooks
4. Use TanStack Query for server state
5. Use Jotai for client state
6. Include proper error handling
7. Write tests alongside code
8. Update documentation if needed

### Testing Guidance

**Always Run Tests:**
```bash
# After making changes
pnpm nx test <affected-project>

# Before committing
pnpm nx affected -t test
```

**Test Coverage Expectations:**
- Business logic: High coverage
- UI components: Key user flows
- Integration points: Critical paths
- Edge cases: Error handling

### Module Boundaries

**Respect NX Module Boundaries:**
- `browser` cannot import from `node`
- `shared` can be imported by all scopes
- Feature libs can import from ui, data, util, service
- UI libs should not import from feature libs

**If Boundary Violated:**
1. Understand why the boundary exists
2. Consider refactoring to respect it
3. If necessary, discuss with team before breaking

### Performance Considerations

**React Compiler is Enabled:**
- Generally avoid `useMemo` and `useCallback`
- Trust the compiler to optimize
- Only manually optimize if profiling shows it's needed

**Code Splitting:**
- Use dynamic imports for large features
- Lazy load routes when possible
- Bundle size awareness

### Documentation Updates

**When to Update Documentation:**
- Adding new features
- Changing architecture
- Adding new patterns
- Deprecating functionality
- Updating dependencies (major versions)

**Where to Document:**
- [AGENTS.md](AGENTS.md): AI agent guidance
- [README.md](README.md): Quick start and overview
- [docs/](docs/): Detailed guides
- Code comments: Complex logic only
- JSDoc: Public APIs and utilities

### Communication with User

**Provide Clear Updates:**
- Explain what you're doing and why
- Highlight any decisions made
- Note any assumptions
- Flag potential issues proactively
- Ask for clarification when uncertain

**Before Major Changes:**
- Explain the proposed approach
- Outline alternatives considered
- Estimate impact and scope
- Wait for confirmation

### Error Recovery

**If Problems Occur:**
1. Check error messages carefully
2. Verify prerequisites are met
3. Clear caches if needed: `pnpm nx reset`
4. Review recent changes
5. Consult documentation
6. Ask user for guidance if stuck

**Common Issues:**
- Missing `.npmrc`: Setup GitHub Packages auth
- Wrong Node version: Run `nvm use`
- Build failures: Clear cache and reinstall
- TypeScript errors: Check imports and types
- ESLint errors: Fix or disable specific rules

### Quality Standards

**Code Quality:**
- Pass all linters (ESLint)
- Pass formatting checks (Prettier)
- Pass type checking (TypeScript)
- Pass all tests (Vitest, Cypress)
- No console warnings in production

**Review Checklist:**
- [ ] Code follows existing patterns
- [ ] Tests written and passing
- [ ] TypeScript types are correct
- [ ] No ESLint errors
- [ ] Formatted with Prettier
- [ ] Module boundaries respected
- [ ] Documentation updated if needed
- [ ] Commit message follows conventions

---

**End of AGENT README**

This document serves as the comprehensive guide for AI agents working in the Sinch Email monorepo. For additional context, refer to [AGENTS.md](AGENTS.md), [README.md](README.md), and documentation in [docs/](docs/).
```


## Deployment

> **Deployment details not yet documented.** Update this section with the deployment process for this repo.
