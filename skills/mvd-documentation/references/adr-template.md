# Architecture Decision Record (ADR) Template

ADRs capture the WHY behind significant architectural decisions. Use them when the decision will affect the codebase long-term and future developers need context.

## ADR Template

```markdown
# ADR-[NUMBER]: [TITLE]

## Status
[Proposed | Accepted | Deprecated | Superseded by ADR-XXX]

## Date
[YYYY-MM-DD]

## Context
[What is the issue we're facing? What forces are at play?]
- Business pressure: [deadlines, stakeholder requirements]
- Technical constraints: [existing architecture, performance needs]
- Team considerations: [expertise, maintenance capacity]

## Decision
We will [decision statement].

## Rationale
We chose this approach because:
1. [Primary reason]
2. [Secondary reason]
3. [Supporting evidence/data]

### Alternatives Considered
- **[Alternative A]**: Rejected because [reason]
- **[Alternative B]**: Rejected because [reason]

## Consequences

### Positive
- [Benefit 1]
- [Benefit 2]

### Negative
- [Trade-off 1]
- [Trade-off 2]

### Risks
- [Risk with mitigation strategy]
```

## Real ADR Example

```markdown
# ADR-007: PostgreSQL for Primary Database

## Status
Accepted

## Date
2024-01-10

## Context
Our application requires a primary database. We're building a SaaS platform with:
- Complex relational data (users, organizations, subscriptions)
- Transaction requirements (payment processing)
- Full-text search needs (content discovery)
- Team has mixed SQL experience (2 senior, 3 junior devs)

## Decision
We will use PostgreSQL 15+ as our primary database.

## Rationale
We chose PostgreSQL because:
1. ACID compliance required for payment transactions
2. Built-in full-text search avoids separate Elasticsearch dependency
3. JSON support allows flexible schema for user preferences
4. Strong ecosystem (Prisma, TypeORM, pg libraries)

### Alternatives Considered
- **MongoDB**: Rejected because transaction support is complex and our data is highly relational
- **MySQL**: Rejected because PostgreSQL's JSON and full-text search are superior for our needs
- **CockroachDB**: Rejected because operational complexity exceeds our team capacity

## Consequences

### Positive
- Single database handles relational, JSON, and search
- Mature tooling and extensive documentation
- Team can leverage existing SQL knowledge

### Negative
- Requires more schema planning than document databases
- Horizontal scaling requires more effort than MongoDB

### Risks
- If we outgrow single-node, will need read replicas or sharding
  - Mitigation: Design with partitioning in mind from start
```

## When to Write an ADR vs Inline Comment

### Write an ADR When:
- Decision affects multiple files or modules
- Alternative approaches were seriously considered
- Future developers might question "why not X?"
- Decision involves significant trade-offs
- Reversing the decision would be expensive

**ADR-worthy decisions:**
- Database technology choice
- Authentication strategy
- API versioning approach
- Deployment architecture
- Major dependency selections
- Data modeling patterns

### Use Inline Comment When:
- Decision is local to a single function or file
- Context is clear from surrounding code
- No significant alternatives were considered
- Explanation is under 3 sentences

**Inline comment examples:**
```typescript
// Using 30/360 day count convention per Stripe's pro-rata method
const dailyRate = monthlyAmount / 30;

// Intentionally catching all errors - upstream retries handle specific cases
} catch (e) { return null; }

// setTimeout avoids React state update on unmounted component
useEffect(() => { const timeout = setTimeout(...) }, []);
```

## Good vs Bad Documentation Examples

### Bad: Duplicates Code
```typescript
/**
 * @param userId - The user's ID
 * @param email - The new email address
 * @returns Promise<User> - The updated user
 */
async function updateUserEmail(userId: string, email: string): Promise<User>
```
TypeScript already tells us this. Zero value added.

### Good: Explains Business Logic
```typescript
/**
 * Email changes trigger immediate session invalidation because:
 * - Security: Compromised email shouldn't retain access
 * - Compliance: SOC2 requires credential change = re-authentication
 * - UX: User expects "secure" behavior for sensitive changes
 */
async function updateUserEmail(userId: string, email: string): Promise<User>
```

### Bad: Placeholder
```typescript
// TODO: Add proper documentation
// TBD: Figure out what this does
// FIXME: Document this later
```
Either document now or delete the comment. Placeholders become permanent.

### Good: Security Warning
```typescript
/**
 * WARNING: Handles PII - Date of Birth
 *
 * @security
 * - Never log the raw value
 * - Mask in error messages: XXXX-XX-XX
 * - Access logged per HIPAA requirement
 */
```

### Bad: Changelog in Code
```typescript
// v1.0 - Initial implementation
// v1.1 - Added retry logic (2024-01-05)
// v1.2 - Fixed timeout issue (2024-01-08)
```
Use git history for this. Comments become stale.

### Good: Algorithm Rationale
```typescript
/**
 * Uses Levenshtein distance with threshold of 3 for typo detection.
 *
 * Threshold chosen empirically:
 * - 1-2: Too strict, misses "recieve" -> "receive"
 * - 4+: Too loose, suggests unrelated words
 * - 3: Catches 94% of common typos in our test set
 */
```

## ADR File Naming and Organization

```
docs/
  architecture/
    decisions/
      ADR-001-use-typescript.md
      ADR-002-monorepo-structure.md
      ADR-003-authentication-strategy.md
      ADR-004-api-versioning.md
      ...
      README.md  # Index of all ADRs
```

### ADR Index (README.md)
```markdown
# Architecture Decision Records

| # | Title | Status | Date |
|---|-------|--------|------|
| 001 | Use TypeScript | Accepted | 2024-01-01 |
| 002 | Monorepo Structure | Accepted | 2024-01-02 |
| 003 | JWT Authentication | Superseded by 008 | 2024-01-05 |
| 004 | API Versioning via URL | Accepted | 2024-01-10 |
```
