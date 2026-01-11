# Service Inventory Template

A structured approach to cataloging existing services and preventing duplication.

## Service Inventory Structure

Create `service-inventory.yaml` in your project root:

```yaml
# service-inventory.yaml
# Last updated: 2024-01-15
# Maintainer: team-name

services:
  # Group by domain
  authentication:
    location: src/modules/auth/
    primary_file: auth.service.ts
    capabilities:
      - validateToken
      - refreshToken
      - generateToken
      - revokeToken
    mandate: ALL authentication MUST use this service
    dependencies: [jwt, redis]

  user_management:
    location: src/modules/users/
    primary_file: user.service.ts
    capabilities:
      - createUser
      - updateUser
      - deleteUser
      - getUserById
      - searchUsers
    mandate: ALL user operations MUST use this service
    related: [authentication, email]

utilities:
  validation:
    location: src/utils/validators/
    files:
      - email.validator.ts
      - phone.validator.ts
      - password.validator.ts
      - uuid.validator.ts
    mandate: ALL input validation MUST use existing validators

  formatting:
    location: src/utils/formatters/
    files:
      - currency.formatter.ts
      - date.formatter.ts
      - phone.formatter.ts
    mandate: ALL display formatting MUST use existing formatters

infrastructure:
  base_classes:
    repository: src/common/repositories/base.repository.ts
    service: src/common/services/base.service.ts
    controller: src/common/controllers/base.controller.ts

  middleware:
    auth: src/common/middleware/auth.middleware.ts
    logging: src/common/middleware/logging.middleware.ts
    rateLimit: src/common/middleware/rate-limit.middleware.ts

  guards:
    roles: src/common/guards/roles.guard.ts
    ownership: src/common/guards/ownership.guard.ts

events:
  # Document event-driven integrations
  order.created:
    triggers:
      - Send confirmation email
      - Update inventory
      - Notify fulfillment
    handler: src/modules/orders/order.events.ts

  user.registered:
    triggers:
      - Send welcome email
      - Create default preferences
      - Log analytics event
    handler: src/modules/users/user.events.ts
```

## How to Document Existing Services

### 1. Identify Service Boundaries
```bash
# Find all service files
find src -name "*.service.ts" -o -name "*Service.ts"

# Find all utility files
find src -name "*.util.ts" -o -name "*.helper.ts"

# Find all middleware
find src -name "*.middleware.ts"
```

### 2. Document Capabilities
For each service, list its public methods:
```bash
# Extract exported functions from a file
grep -E "export (async )?function|export class" src/modules/auth/auth.service.ts
```

### 3. Document Dependencies
Note what each service depends on for import decisions:
```bash
# Find imports in a service
head -30 src/modules/auth/auth.service.ts
```

## Search Patterns for Finding Existing Functionality

Before creating ANYTHING, run these searches:

### By Function Type
```bash
# Validation
grep -ri "validate\|isValid\|check.*valid" src/ --include="*.ts"

# Formatting
grep -ri "format\|toString\|display" src/ --include="*.ts"

# API calls
grep -ri "fetch\|axios\|httpClient" src/ --include="*.ts"

# Caching
grep -ri "cache\|redis\|memoize" src/ --include="*.ts"
```

### By Domain
```bash
# Email functionality
grep -ri "email\|sendmail\|smtp" src/ --include="*.ts"

# Payment processing
grep -ri "payment\|stripe\|charge\|invoice" src/ --include="*.ts"

# File handling
grep -ri "upload\|download\|s3\|storage" src/ --include="*.ts"
```

### By Pattern Name
```bash
# Repository pattern
grep -ri "repository\|Repository" src/ --include="*.ts"

# Factory pattern
grep -ri "factory\|Factory\|create.*Instance" src/ --include="*.ts"
```

## Decision Criteria: When Is It "Similar Enough"?

### REUSE if:
- Same input/output types (even if names differ)
- 80%+ functionality overlap
- Same external service (Stripe, Twilio, etc.)
- Same domain concept (user, order, payment)

### CREATE NEW if:
- Fundamentally different algorithm required
- Different external service integration
- New domain not yet represented
- Existing service would require breaking changes

### Examples

**Scenario: Need email validation for new signup form**
```
Search: grep -ri "email.*valid\|validate.*email" src/
Found: src/utils/validators/email.validator.ts

Decision: REUSE - exact functionality exists
Action: import { validateEmail } from '@/utils/validators'
```

**Scenario: Need SMS notifications (new feature)**
```
Search: grep -ri "sms\|twilio\|text.*message" src/
Found: Nothing

Decision: CREATE NEW - new integration required
Action: Create src/modules/notifications/sms.service.ts
Update: Add to service-inventory.yaml after creation
```

## Service Categories Reference

### Common Utilities (check first)
- Validators: email, phone, password, URL, UUID, date range
- Formatters: currency, dates, phone numbers, addresses
- Parsers: CSV, JSON, query strings, file types
- Generators: IDs, slugs, tokens, random strings

### Common Infrastructure (extend, don't recreate)
- Base Repository: CRUD operations, query building
- Base Service: Logging, error handling, transactions
- Base Controller: Response formatting, error responses

### Common Integrations (use existing)
- Auth: Token validation, session management
- Email: Transactional, marketing, templates
- Storage: File upload, signed URLs, CDN
- Payments: Charges, refunds, subscriptions

### Event Handlers (emit, don't couple)
- Use existing event bus instead of direct service calls
- Check existing event handlers before creating new ones
- Document new events in service inventory
