# AGENT-README: simpletexting-auth0-terraform

**Repository:** `git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/simpletexting-auth0-terraform.git`

**Last Updated:** 2026-02-24

---

## LLM Agent Preamble

### Mandatory Reading Requirements

Before making any changes to this repository, AI agents MUST:

1. **Read this entire AGENT-README document** - Understanding the localization build system and Terraform deployment workflow is critical
2. **Review the current email templates** in `liquid_template/*.html` to understand the base HTML structure
3. **Examine localization files** in `liquid_template/languages/*.json` to understand translation key-value structure
4. **Study the TypeScript localization scripts** in `scripts/localization/*.ts` to understand how Liquid conditionals are generated
5. **Review Terraform files** (`email_templates.tf`, `main.tf`) to understand Auth0 resource management
6. **Check test files** (`support/test.sh`, `scripts/quality-check.sh`) to understand validation requirements

### Focus Areas for Analysis

When working with this codebase, pay special attention to:

- **Multi-language support system** - TypeScript → Liquid template generation with conditional logic
- **Auth0 resource definitions** - Email templates, applications, and M2M clients
- **Build workflow** - npm scripts that must run before Terraform deployment
- **Template variable consistency** - Ensuring Auth0 variables are properly used across languages
- **Terraform state management** - Changes affect production Auth0 configuration

### Testing Requirements

Before proposing or implementing changes:

1. Run `npm run localize` to regenerate Liquid templates
2. Execute `support/test.sh` to validate all 40+ test cases
3. Run `scripts/quality-check.sh` for comprehensive validation (16 checks)
4. Test Terraform plan with `terraform plan` to verify no unintended changes
5. Validate generated templates contain proper Liquid syntax

---

## Prerequisites

### Required Software

- **Node.js**: 22.17.0 or higher
- **npm**: Latest version compatible with Node.js 22+
- **Terraform**: 1.0 or higher
- **jq**: JSON processor (for test scripts)
- **bash**: Shell interpreter (macOS/Linux)

### Required Credentials & Access

- **Auth0 Management API credentials**:
  - `AUTH0_DOMAIN`
  - `AUTH0_CLIENT_ID`
  - `AUTH0_CLIENT_SECRET`
- **GitLab access**: Read/write permissions for the repository
- **Terraform state access**: Backend configuration credentials (if using remote state)

### Environment Setup

```bash
export AUTH0_DOMAIN="your-tenant.auth0.com"
export AUTH0_CLIENT_ID="your-client-id"
export AUTH0_CLIENT_SECRET="your-client-secret"
```

---

## Project Overview

The `simpletexting-auth0-terraform` repository manages Auth0 configuration as Infrastructure as Code (IaC) with a sophisticated multi-language email template system.

### Purpose

- **Primary**: Deploy and manage Auth0 email templates with comprehensive 6-language support
- **Secondary**: Manage Auth0 applications and Machine-to-Machine (M2M) clients for SimpleTexting Services
- **Tertiary**: Provide repeatable, version-controlled Auth0 configuration deployment

### Key Capabilities

1. **Multi-language Email Templates**: Supports 6 languages (en-US, en-GB, es-ES, de-DE, fr-FR, pt-BR)
2. **Localization Build System**: TypeScript scripts generate Liquid template conditionals from JSON translation files
3. **Auth0 Resource Management**: Terraform manages email templates, applications, and M2M clients
4. **Comprehensive Testing**: 40+ automated tests validate template generation and quality

### Business Value

- Consistent user experience across multiple languages and regions
- Rapid deployment of email template updates
- Version-controlled infrastructure changes with audit trail
- Reduced manual configuration errors in Auth0

---

## Classification

**Category:** Infrastructure as Code (IaC) - Terraform

**Sub-categories:**
- Identity & Access Management (IAM)
- Email Template Management
- Multi-language Localization System
- Auth0 Configuration Management

**Technology Profile:**
- **Infrastructure**: Terraform-managed Auth0 resources
- **Build System**: TypeScript/Node.js
- **Template Engine**: Liquid (Auth0-supported)
- **Version Control**: GitLab

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│  Developer Workflow                                         │
│                                                             │
│  1. Edit JSON translations (liquid_template/languages/)    │
│  2. Edit HTML templates (liquid_template/*.html)           │
│  3. Run npm run localize                                   │
│                                                             │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│  TypeScript Localization Build System                      │
│                                                             │
│  • localizeMessage() function                              │
│  • Generates Liquid {{% if user.user_metadata.locale %}}   │
│  • Creates conditional blocks for each language            │
│  • Outputs to scripts/tmp/gen/ (gitignored)                │
│                                                             │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│  Generated Liquid Templates                                 │
│                                                             │
│  • verify_email.html (with 6-language conditionals)        │
│  • reset_email.html                                        │
│  • blocked_account.html                                    │
│  • stolen_credentials.html                                 │
│                                                             │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│  Terraform Deployment                                       │
│                                                             │
│  • email_templates.tf references generated files           │
│  • auth0 provider ~> 1.36                                  │
│  • Applies to Auth0 tenant                                 │
│                                                             │
└────────────────┬────────────────────────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────────────────────────┐
│  Auth0 Production                                           │
│                                                             │
│  • Email templates live in Auth0                           │
│  • User receives email based on user_metadata.locale       │
│  • Liquid engine evaluates conditionals at runtime         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Architecture Principles

1. **Source of Truth**: JSON translation files and HTML templates in version control
2. **Build-Time Generation**: Liquid conditionals generated during build, not written manually
3. **Infrastructure as Code**: All Auth0 resources defined in Terraform
4. **Runtime Evaluation**: Auth0's Liquid engine evaluates locale at email send time

---

## Key Components

### 1. Email Templates (4 Templates)

| Template | Purpose | User Action |
|----------|---------|-------------|
| `verify_email` | Email verification for new accounts | User signs up |
| `reset_email` | Password reset requests | User requests password reset |
| `blocked_account` | Account blocked notifications | Security event triggers block |
| `stolen_credentials` | Compromised credential alerts | Credentials found in breach |

### 2. Supported Languages (6 Languages)

- **en-US**: English (United States)
- **en-GB**: English (United Kingdom)
- **es-ES**: Spanish (Spain)
- **de-DE**: German (Germany)
- **fr-FR**: French (France)
- **pt-BR**: Portuguese (Brazil)

### 3. Localization Build System

**Core Components:**
- `scripts/localization/*.ts`: TypeScript scripts with `localizeMessage()` function
- `liquid_template/languages/*.json`: Translation key-value pairs for each language
- `npm run localize`: Command that generates Liquid templates

**Process:**
1. Reads JSON translation files
2. Creates Liquid conditional blocks: `{% if user.user_metadata.locale == "es-ES" %}`
3. Inserts localized text for each language
4. Provides fallback to en-US for unsupported locales
5. Outputs to `scripts/tmp/gen/` directory

### 4. Auth0 Resources

**Managed via Terraform:**
- Email templates (4 templates × 6 languages)
- Auth0 applications for SimpleTexting Services
- Machine-to-Machine (M2M) clients
- Application settings and metadata

---

## Development Environment

### Initial Setup

```bash
# Clone repository
git clone git@gitlab.com:sinch/sinch-projects/enterprise-and-messaging/beehive/teams/sinch-identity/simpletexting-auth0-terraform.git
cd simpletexting-auth0-terraform

# Install Node.js dependencies
npm install

# Set up Auth0 credentials
export AUTH0_DOMAIN="your-tenant.auth0.com"
export AUTH0_CLIENT_ID="your-client-id"
export AUTH0_CLIENT_SECRET="your-client-secret"

# Initialize Terraform
terraform init

# Generate localized templates
npm run localize

# Verify setup
support/test.sh
```

### Local Development Workflow

1. **Make changes** to translation files (`liquid_template/languages/*.json`) or templates (`liquid_template/*.html`)
2. **Regenerate templates**: `npm run localize`
3. **Run tests**: `support/test.sh` and `scripts/quality-check.sh`
4. **Preview changes**: `terraform plan`
5. **Commit changes**: Git commit + push to GitLab
6. **Deploy**: `terraform apply` (after code review/approval)

### Development Tools

- **IDE**: VS Code with Terraform and TypeScript extensions recommended
- **Linting**: TypeScript compiler for localization scripts
- **Validation**: Bash test scripts for comprehensive validation
- **Version Control**: Git with GitLab workflows

---

## Build & Deployment

### Build Process

#### Step 1: Install Dependencies

```bash
npm install
```

#### Step 2: Generate Localized Templates

```bash
npm run localize
```

This command:
- Executes TypeScript localization scripts
- Reads translation JSON files
- Generates Liquid templates with conditional logic
- Outputs to `scripts/tmp/gen/` (gitignored)

#### Step 3: Validate Generated Templates

```bash
support/test.sh
scripts/quality-check.sh
```

### Terraform Deployment Workflow

#### Plan Changes

```bash
terraform plan
```

Review the plan carefully to ensure only intended changes are shown.

#### Apply Changes

```bash
terraform apply
```

Applies Auth0 configuration changes. **CAUTION**: This affects production Auth0 tenant.

#### Deployment Checklist

- [ ] All tests passing (`support/test.sh`)
- [ ] Quality checks passing (`scripts/quality-check.sh`)
- [ ] `terraform plan` reviewed and approved
- [ ] Code review completed
- [ ] Backup of current Auth0 configuration (if critical change)
- [ ] Rollback plan documented

### CI/CD Considerations

While not explicitly documented in the repository, typical pipeline stages would include:

1. **Build**: `npm install && npm run localize`
2. **Test**: `support/test.sh && scripts/quality-check.sh`
3. **Plan**: `terraform plan` (auto-run on merge requests)
4. **Apply**: `terraform apply` (manual approval after merge to main)

---

## Project Structure

```
simpletexting-auth0-terraform/
├── liquid_template/              # Source email templates
│   ├── blocked_account.html      # Account blocked template
│   ├── reset_email.html          # Password reset template
│   ├── stolen_credentials.html   # Compromised credentials template
│   ├── verify_email.html         # Email verification template
│   └── languages/                # Translation files
│       ├── en-US.json            # English (US) translations
│       ├── en-GB.json            # English (UK) translations
│       ├── es-ES.json            # Spanish translations
│       ├── de-DE.json            # German translations
│       ├── fr-FR.json            # French translations
│       └── pt-BR.json            # Portuguese translations
│
├── scripts/
│   ├── localization/             # TypeScript localization build system
│   │   └── *.ts                  # Localization scripts
│   ├── tmp/gen/                  # Generated Liquid templates (gitignored)
│   └── quality-check.sh          # Quality validation script (16 checks)
│
├── support/
│   └── test.sh                   # Main test suite (40+ tests)
│
├── *.tf                          # Terraform configuration files
│   ├── email_templates.tf        # Email template resources
│   ├── main.tf                   # Applications and M2M clients
│   ├── variables.tf              # Terraform variables
│   └── providers.tf              # Provider configuration
│
├── package.json                  # Node.js dependencies and scripts
├── tsconfig.json                 # TypeScript configuration
└── README.md                     # Project documentation
```

---

## Key Files

### Critical Files (Do Not Modify Without Review)

| File | Purpose | Criticality |
|------|---------|-------------|
| `email_templates.tf` | Defines Auth0 email template resources | **CRITICAL** |
| `main.tf` | Defines Auth0 applications and M2M clients | **CRITICAL** |
| `scripts/localization/*.ts` | TypeScript localization build system | **HIGH** |
| `liquid_template/*.html` | Base HTML email templates | **HIGH** |

### Configuration Files

| File | Purpose |
|------|---------|
| `liquid_template/languages/*.json` | Translation key-value pairs for each language |
| `package.json` | npm scripts and dependencies |
| `tsconfig.json` | TypeScript compiler configuration |
| `variables.tf` | Terraform input variables |
| `providers.tf` | Auth0 provider configuration |

### Testing Files

| File | Purpose |
|------|---------|
| `support/test.sh` | Main test suite with 40+ test cases |
| `scripts/quality-check.sh` | Quality validation with 16 checks |

### Generated Files (Gitignored)

| File/Directory | Purpose |
|----------------|---------|
| `scripts/tmp/gen/*.html` | Generated Liquid templates (output of `npm run localize`) |
| `node_modules/` | Node.js dependencies |
| `.terraform/` | Terraform plugins and modules |

---

## Testing Strategy

### Test Suite Overview

The repository includes comprehensive testing with two primary test scripts:

#### 1. Main Test Suite: `support/test.sh`

**Coverage**: 40+ test cases

**Test Categories**:
- Liquid template syntax validation
- Language conditional logic verification
- Variable substitution testing
- HTML structure validation
- Encoding and escaping checks
- Translation key coverage verification
- Fallback behavior to en-US

**Execution**:
```bash
./support/test.sh
```

**Expected Output**: All tests should pass with clear success/failure indicators.

#### 2. Quality Checks: `scripts/quality-check.sh`

**Coverage**: 16 quality checks

**Check Categories**:
- Generated file existence verification
- Liquid syntax correctness
- Translation completeness
- HTML validity
- Link integrity
- Auth0 variable usage consistency

**Execution**:
```bash
./scripts/quality-check.sh
```

### Testing Workflow

1. **Before Changes**:
   - Run baseline tests to ensure clean starting state
   - Document current test results

2. **During Development**:
   - Run `npm run localize` after each change
   - Execute tests incrementally to catch issues early

3. **Before Commit**:
   - Run full test suite: `support/test.sh`
   - Run quality checks: `scripts/quality-check.sh`
   - Both must pass before committing

4. **Before Deployment**:
   - Execute `terraform plan` and review changes
   - Verify only intended resources are modified
   - Confirm Auth0 provider credentials are correct

### Manual Validation

In addition to automated tests, manually verify:

- Visual rendering of HTML templates in browser
- Proper rendering of Auth0 variables (use test email sends)
- Mobile responsiveness of email templates
- Spam score of generated emails (use mail-tester.com)

---

## Security & Compliance

### Secrets Management

**Required Secrets**:
- `AUTH0_DOMAIN`: Auth0 tenant domain
- `AUTH0_CLIENT_ID`: Management API client ID
- `AUTH0_CLIENT_SECRET`: Management API client secret

**Best Practices**:
- Never commit secrets to version control
- Use environment variables or secure secret management systems
- Rotate credentials periodically
- Use least-privilege access for Auth0 Management API

### Auth0 Permissions

Required Auth0 Management API scopes:
- `create:email_templates`
- `read:email_templates`
- `update:email_templates`
- `delete:email_templates`
- `create:clients`
- `read:clients`
- `update:clients`

### Compliance Considerations

- **GDPR**: Email templates must support EU languages (de-DE, fr-FR, es-ES)
- **Accessibility**: HTML templates should follow WCAG guidelines
- **Data Privacy**: No PII should be logged in build/deployment processes
- **Audit Trail**: All changes tracked in Git with commit messages

### Security Best Practices

1. **Template Security**:
   - Avoid unsafe Liquid filters
   - Sanitize user-provided metadata if used in templates
   - Use Auth0-provided variables only

2. **Terraform State**:
   - Use remote state backend with encryption
   - Restrict access to state files
   - Enable state locking to prevent concurrent modifications

3. **Deployment Security**:
   - Require code review before merging
   - Use protected branches in GitLab
   - Implement approval workflows for `terraform apply`

---

## Monitoring & Operations

### Health Checks

**Build System Health**:
- Verify `npm run localize` completes successfully
- Check generated files exist in `scripts/tmp/gen/`
- Validate Liquid syntax in generated templates

**Terraform State Health**:
- Monitor Terraform state for drift: `terraform plan` should show no changes
- Verify Auth0 provider connectivity
- Check for unauthorized manual changes in Auth0 console

### Logging

**Build Logs**:
- npm script output during `npm run localize`
- TypeScript compilation errors
- Test script output

**Deployment Logs**:
- `terraform plan` output
- `terraform apply` output
- Auth0 API responses

### Operational Procedures

#### Rollback Procedure

If a deployment causes issues:

1. **Identify Last Good Commit**:
   ```bash
   git log --oneline
   ```

2. **Revert to Previous Version**:
   ```bash
   git revert <commit-hash>
   npm run localize
   terraform plan
   terraform apply
   ```

3. **Alternative - Terraform State Rollback**:
   ```bash
   terraform state pull > backup.tfstate
   terraform apply -lock=false -auto-approve <previous-version>
   ```

#### Emergency Fixes

For urgent email template fixes:

1. Make minimal change to translation JSON or HTML template
2. Run `npm run localize`
3. Run `support/test.sh` to validate
4. Create merge request with "URGENT" label
5. Fast-track review and approval
6. Deploy with `terraform apply`

#### Terraform State Recovery

If Terraform state is corrupted:

1. Recover from remote backend backup
2. Or import existing Auth0 resources:
   ```bash
   terraform import auth0_email_template.verify_email verify_email
   ```

---

## Technical Stack & Dependencies

### Core Technologies

| Technology | Version | Purpose |
|------------|---------|---------|
| **Terraform** | ~> 1.0 | Infrastructure as Code framework |
| **Auth0 Provider** | ~> 1.36 | Terraform provider for Auth0 |
| **Node.js** | 22.17.0+ | JavaScript runtime for build system |
| **TypeScript** | Latest | Type-safe localization scripts |
| **npm** | Latest | Package management |
| **Liquid** | Auth0 version | Template engine for dynamic email content |

### npm Dependencies

Key dependencies from `package.json`:

- **TypeScript compiler**: Compiles localization scripts
- **Type definitions**: @types/node for Node.js APIs
- Additional dependencies for file I/O, JSON parsing, etc.

### Terraform Provider Configuration

```hcl
terraform {
  required_providers {
    auth0 = {
      source  = "auth0/auth0"
      version = "~> 1.36"
    }
  }
  required_version = "~> 1.0"
}
```

### External Dependencies

- **Auth0 Platform**: SaaS identity platform
- **Auth0 Management API**: API for programmatic Auth0 configuration
- **GitLab**: Version control and CI/CD platform

---

## Service Contract

### Email Template Types

The repository manages four distinct email template types used by Auth0:

#### 1. Verify Email (`verify_email`)

**Purpose**: Email verification for new user accounts

**Trigger**: User signs up with email/password

**Required Variables**:
- `user.email`: Recipient email address
- `url`: Verification link with token
- `user.user_metadata.locale`: User's preferred language

**Expected Behavior**:
- User clicks verification link
- Auth0 validates token
- Account becomes verified

#### 2. Reset Email (`reset_email`)

**Purpose**: Password reset requests

**Trigger**: User initiates password reset

**Required Variables**:
- `user.email`: Recipient email address
- `url`: Password reset link with token
- `user.user_metadata.locale`: User's preferred language

**Expected Behavior**:
- User clicks reset link
- Redirected to password reset page
- Enters new password

#### 3. Blocked Account (`blocked_account`)

**Purpose**: Notification when account is blocked due to suspicious activity

**Trigger**: Auth0 anomaly detection or manual block

**Required Variables**:
- `user.email`: Recipient email address
- `user.user_metadata.locale`: User's preferred language

**Expected Behavior**:
- User is informed of block
- Provided contact information for support

#### 4. Stolen Credentials (`stolen_credentials`)

**Purpose**: Alert when user credentials are found in a data breach

**Trigger**: Auth0 detects credentials in breach database

**Required Variables**:
- `user.email`: Recipient email address
- `user.user_metadata.locale`: User's preferred language

**Expected Behavior**:
- User is warned of compromised credentials
- Encouraged to change password immediately

### Language Support Contract

**Supported Locales**: `en-US`, `en-GB`, `es-ES`, `de-DE`, `fr-FR`, `pt-BR`

**Locale Detection**:
- Reads from `user.user_metadata.locale`
- Falls back to `en-US` if locale is unsupported or missing

**Conditional Logic Structure**:
```liquid
{% if user.user_metadata.locale == "es-ES" %}
  [Spanish content]
{% elsif user.user_metadata.locale == "de-DE" %}
  [German content]
{% elsif user.user_metadata.locale == "fr-FR" %}
  [French content]
{% elsif user.user_metadata.locale == "pt-BR" %}
  [Portuguese content]
{% elsif user.user_metadata.locale == "en-GB" %}
  [British English content]
{% else %}
  [US English content - default]
{% endif %}
```

### SLA & Performance

- **Deployment Time**: ~2-5 minutes for `terraform apply`
- **Build Time**: ~10-30 seconds for `npm run localize`
- **Test Execution**: ~5-15 seconds for all tests
- **Auth0 Email Delivery**: Per Auth0 SLA (typically <1 minute)

---

## Core Business Functions

### 1. Email Template Localization

**Function**: Generate multi-language email templates for Auth0 user communications

**Business Impact**:
- Improves user experience for international customers
- Reduces bounce rates from non-English speakers
- Supports global expansion to 6 language markets
- Ensures consistent branding across languages

**Key Operations**:
- Read translation JSON files for each supported language
- Generate Liquid conditional blocks for runtime locale detection
- Maintain HTML structure consistency across translations
- Provide en-US fallback for unsupported locales

### 2. Auth0 Application Management

**Function**: Manage Auth0 applications for SimpleTexting Services via Terraform

**Business Impact**:
- Ensures consistent Auth0 configuration across environments
- Enables repeatable application deployment
- Provides audit trail for application changes
- Reduces manual configuration errors

**Key Operations**:
- Define application settings, URLs, and metadata
- Configure authentication flows and grants
- Manage application secrets and credentials
- Update application branding and appearance

### 3. Machine-to-Machine Client Management

**Function**: Manage M2M clients for service-to-service authentication

**Business Impact**:
- Enables secure backend service communication
- Supports microservices architecture
- Provides centralized credential management
- Facilitates API access control

**Key Operations**:
- Create M2M client applications
- Configure client credentials and grants
- Assign API permissions and scopes
- Manage client metadata and settings

---

## Configuration

### Environment Variables

Required environment variables for deployment:

```bash
# Auth0 Configuration
export AUTH0_DOMAIN="your-tenant.auth0.com"
export AUTH0_CLIENT_ID="your-management-api-client-id"
export AUTH0_CLIENT_SECRET="your-management-api-client-secret"

# Optional: Terraform Backend Configuration
export TF_BACKEND_BUCKET="your-state-bucket"
export TF_BACKEND_KEY="terraform.tfstate"
```

### Terraform Variables

Key Terraform variables (defined in `variables.tf`):

- **Auth0 domain**: Target Auth0 tenant
- **Application names**: Names for managed applications
- **Client settings**: Configuration for M2M clients
- **Template settings**: Email template-specific configuration

### npm Scripts

Available npm scripts in `package.json`:

```json
{
  "scripts": {
    "localize": "Run TypeScript localization build system",
    "build": "Compile TypeScript scripts",
    "test": "Run test suite (if integrated)"
  }
}
```

### Localization Configuration

Translation files structure (`liquid_template/languages/*.json`):

```json
{
  "key_name": "Localized value",
  "email_subject": "Subject in target language",
  "email_body": "Body text in target language",
  "cta_button": "Button text in target language"
}
```

**Translation Key Conventions**:
- Use snake_case for key names
- Keep keys consistent across all language files
- Include context comments for translators
- Maintain alphabetical order for maintainability

---

## Critical Commands

### Build Commands

```bash
# Install dependencies
npm install

# Generate localized Liquid templates
npm run localize

# Compile TypeScript (if separate script)
npm run build
```

### Testing Commands

```bash
# Run main test suite (40+ tests)
./support/test.sh

# Run quality checks (16 checks)
./scripts/quality-check.sh

# Run both test suites
./support/test.sh && ./scripts/quality-check.sh
```

### Terraform Commands

```bash
# Initialize Terraform (first time or after provider changes)
terraform init

# Format Terraform files
terraform fmt

# Validate Terraform configuration
terraform validate

# Plan changes (dry run)
terraform plan

# Apply changes to Auth0
terraform apply

# Show current state
terraform show

# Check for drift
terraform plan -detailed-exitcode

# Import existing Auth0 resource
terraform import auth0_email_template.verify_email verify_email
```

### Development Commands

```bash
# View generated template (after npm run localize)
cat scripts/tmp/gen/verify_email.html

# Check for translation key consistency
grep -r "key_name" liquid_template/languages/

# Count lines in generated templates
wc -l scripts/tmp/gen/*.html

# Search for Auth0 variables in templates
grep -E '\{\{\s*\w+\.\w+\s*\}\}' liquid_template/*.html
```

### Debugging Commands

```bash
# Check TypeScript compilation errors
npx tsc --noEmit

# Validate JSON syntax in translation files
for file in liquid_template/languages/*.json; do
  echo "Validating $file"
  jq empty "$file"
done

# Show Terraform provider version
terraform version

# Debug Terraform plan with detailed logging
TF_LOG=DEBUG terraform plan
```

---

## Localization System Details

### Architecture Overview

The localization system transforms human-editable JSON translation files into Liquid templates with conditional logic for runtime locale evaluation.

### Key Function: `localizeMessage()`

**Location**: `scripts/localization/*.ts`

**Purpose**: Generate Liquid conditional blocks for multi-language support

**Input**:
- Translation key (e.g., "email_subject")
- Optional default text for fallback

**Output**: Liquid conditional string

**Example Output**:
```liquid
{% if user.user_metadata.locale == "es-ES" %}
  Verifica tu dirección de correo electrónico
{% elsif user.user_metadata.locale == "de-DE" %}
  Bestätigen Sie Ihre E-Mail-Adresse
{% elsif user.user_metadata.locale == "fr-FR" %}
  Vérifiez votre adresse e-mail
{% elsif user.user_metadata.locale == "pt-BR" %}
  Verifique seu endereço de e-mail
{% elsif user.user_metadata.locale == "en-GB" %}
  Verify your email address
{% else %}
  Verify your email address
{% endif %}
```

### Build Process Flow

1. **Read Source Templates**:
   - Load HTML templates from `liquid_template/*.html`
   - Identify translation markers (special syntax in TypeScript)

2. **Load Translation Data**:
   - Read JSON files from `liquid_template/languages/*.json`
   - Parse and validate JSON structure
   - Verify key consistency across languages

3. **Generate Liquid Conditionals**:
   - For each translation key, call `localizeMessage()`
   - Create `{% if %}` / `{% elsif %}` / `{% else %}` blocks
   - Insert translated text for each supported locale
   - Ensure en-US fallback in `{% else %}` block

4. **Output Templates**:
   - Write generated Liquid templates to `scripts/tmp/gen/`
   - Preserve HTML structure and formatting
   - Maintain Auth0 variable placeholders (e.g., `{{ url }}`)

5. **Validation**:
   - Check Liquid syntax correctness
   - Verify all translation keys are covered
   - Ensure Auth0 variables are preserved

### Translation File Structure

**File Naming Convention**: `liquid_template/languages/{locale}.json`

**Example (en-US.json)**:
```json
{
  "email_subject_verify": "Verify Your Email Address",
  "email_greeting": "Hello",
  "verify_button_text": "Verify Email",
  "footer_text": "If you didn't request this email, you can safely ignore it."
}
```

**Example (es-ES.json)**:
```json
{
  "email_subject_verify": "Verifica tu dirección de correo electrónico",
  "email_greeting": "Hola",
  "verify_button_text": "Verificar correo",
  "footer_text": "Si no solicitaste este correo, puedes ignorarlo de forma segura."
}
```

### Adding a New Language

To add support for a new language:

1. **Create Translation File**:
   ```bash
   cp liquid_template/languages/en-US.json liquid_template/languages/ja-JP.json
   ```

2. **Translate Content**:
   - Edit `ja-JP.json` with Japanese translations
   - Maintain same keys as other language files

3. **Update Localization Script**:
   - Add `ja-JP` to supported locales array in TypeScript
   - Update `localizeMessage()` to include new locale

4. **Regenerate Templates**:
   ```bash
   npm run localize
   ```

5. **Test**:
   ```bash
   ./support/test.sh
   ./scripts/quality-check.sh
   ```

6. **Deploy**:
   ```bash
   terraform plan
   terraform apply
   ```

### Liquid Conditional Evaluation

At runtime (when Auth0 sends an email):

1. Auth0 reads `user.user_metadata.locale` from user profile
2. Liquid engine evaluates `{% if %}` conditionals top-to-bottom
3. First matching condition's content is rendered
4. If no match, `{% else %}` block (en-US) is rendered
5. Final HTML email is sent to user

### Best Practices

- **Keep translations concise**: Email clients have limited display width
- **Maintain consistent tone**: Professional, friendly tone across languages
- **Test with real emails**: Use Auth0's test email functionality
- **Validate HTML rendering**: Check in multiple email clients (Gmail, Outlook, Apple Mail)
- **Update all languages together**: Avoid partial translations
- **Document translation context**: Add comments for ambiguous phrases

---

## General Guidance for AI Agents

### Understanding the Codebase

This repository is unique because:

1. **It's a hybrid system**: Combines TypeScript build scripts with Terraform IaC
2. **Generated files are critical**: The `scripts/tmp/gen/` directory contains deployment artifacts
3. **Don't edit generated files**: Always edit source templates and translations, then regenerate
4. **Liquid syntax matters**: Auth0 evaluates Liquid at runtime, so syntax must be perfect

### Common Pitfalls to Avoid

1. **Editing generated templates directly**: Always edit source files and run `npm run localize`
2. **Skipping tests**: Both test scripts must pass before deployment
3. **Modifying Liquid conditionals manually**: Use the TypeScript build system
4. **Ignoring translation key consistency**: All language files must have identical keys
5. **Deploying without `terraform plan`**: Always review plan output before applying

### Making Changes Safely

#### To Update Email Content:

1. Edit `liquid_template/languages/*.json` with new translation values
2. Run `npm run localize` to regenerate templates
3. Run tests: `./support/test.sh && ./scripts/quality-check.sh`
4. Review changes: `git diff`
5. Commit and deploy

#### To Update HTML Structure:

1. Edit base template in `liquid_template/*.html`
2. Ensure Auth0 variables are preserved (e.g., `{{ url }}`)
3. Run `npm run localize`
4. Test thoroughly - HTML changes affect all languages
5. Review generated output in `scripts/tmp/gen/`
6. Deploy with caution

#### To Add New Auth0 Resources:

1. Add resource definitions to appropriate `.tf` file
2. Run `terraform plan` to preview changes
3. Verify no existing resources are modified unintentionally
4. Run `terraform apply`
5. Document new resources in this README

### When to Seek Human Review

- Changes affecting all email templates
- Modifications to Terraform provider version
- Updates to Auth0 Management API scopes
- Adding/removing supported languages
- Changes to `localizeMessage()` function logic
- Any Terraform plan showing unexpected resource deletions

### Debugging Strategies

**Problem**: Generated templates have syntax errors

**Solution**:
1. Check TypeScript compilation: `npx tsc --noEmit`
2. Validate JSON files: `jq empty liquid_template/languages/*.json`
3. Review `localizeMessage()` logic in TypeScript
4. Check for unescaped special characters in translations

**Problem**: Terraform plan shows unwanted changes

**Solution**:
1. Check for manual Auth0 console modifications
2. Verify `scripts/tmp/gen/` is up-to-date: `npm run localize`
3. Review recent Git changes: `git log --oneline`
4. Compare state with code: `terraform show`

**Problem**: Tests failing after changes

**Solution**:
1. Read test output carefully - tests are descriptive
2. Verify generated files exist: `ls -la scripts/tmp/gen/`
3. Check Liquid syntax in generated templates
4. Ensure all translation keys are present in all language files

### Resources for Learning

- **Liquid Documentation**: https://shopify.github.io/liquid/
- **Auth0 Email Templates**: https://auth0.com/docs/customize/email
- **Terraform Auth0 Provider**: https://registry.terraform.io/providers/auth0/auth0/latest/docs
- **TypeScript Handbook**: https://www.typescriptlang.org/docs/

---

**End of AGENT-README**

## Deployment

> **Deployment details not yet documented.** Update this section with the deployment process for this repo.
