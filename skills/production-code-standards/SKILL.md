---
name: production-code-standards
description: |
  This skill enforces production-grade code quality standards. Activate when "implementing a feature",
  "fixing a bug", "creating a service", "writing production code", "reviewing a PR", or for code in
  src/, lib/, or app/ directories. Blocks workarounds, fallbacks, TODO/FIXME/HACK comments, and empty
  catch blocks.
---

# Production Code Standards

All production code must be permanent, complete, and production-grade.

## Enforcement Workflow

1. **Before writing**: Plan the complete, permanent solution
2. **While writing**: Check against prohibited patterns
3. **When blocked**: STOP, document, create ticket - never workaround
4. **Before committing**: Verify no prohibited patterns exist

## Prohibited Patterns

| Pattern | Why Blocked |
|---------|-------------|
| `value \|\| default` | Hides errors silently |
| `TODO/FIXME/HACK` | Temporary code in production |
| `catch (e) { }` | Swallows errors |
| Mock services in src/ | Test artifacts in production |
| `setTimeout` for race conditions | Workaround, not fix |

## Required Patterns

| Pattern | Purpose |
|---------|---------|
| Fail-fast validation | Errors caught at entry |
| Typed custom errors | Debuggable, catchable |
| Error propagation | Let errors bubble to handlers |
| Repository pattern | Data access abstraction |

## When Blocked

If proper implementation is blocked:

1. **STOP** - Do not create a workaround
2. **DOCUMENT** - State what's blocking
3. **CREATE TICKET** - File ticket for the blocker
4. **WAIT** - Blocker must be fixed first

**If you cannot implement without a workaround, do not implement. Communicate the blocker clearly.**

See `references/anti-patterns.md` for detailed code examples of prohibited and required patterns.
