# PM Workflow Command Reference

Complete reference for all workflow commands and their purposes.

## Project-Level Commands (Phases 1-4)

Run these in order when starting a new project or major initiative:

| Phase | Command | Purpose |
|-------|---------|---------|
| 1 | `/generate_service_inventory` | Catalog existing code to prevent duplication |
| 2 | `/discovery` | Analyze patterns, architecture, and integration points |
| 3 | `/epic-planning` | Create business-focused epics with duplicate detection |
| 4 | `/planning` | Decompose epics into actionable sub-tickets |

## Ticket-Level Commands (Phases 5-10)

Run these for EACH ticket, in order:

| Phase | Command | Purpose |
|-------|---------|---------|
| 5 | `/adaptation` | Create implementation guide for specific ticket |
| 6 | `/implementation` | Write production code |
| 7 | `/testing` | Build comprehensive test suite |
| 8 | `/documentation` | Generate API docs and user guides |
| 9 | `/codereview` | Quality assurance review |
| 10 | `/security_review` | OWASP vulnerability assessment |

**Critical**: Only `/security_review` closes tickets and merges code to main.

## Available Skills Reference

| Skill | Triggers When | What It Enforces |
|-------|---------------|------------------|
| `production-code-standards` | Writing ANY code, reviewing code, suggesting implementations | Zero workarounds, no temporary fixes, fail-fast patterns |
| `service-reuse` | Creating new services, APIs, or utilities | Must check service inventory first, prevent duplication |
| `testing-philosophy` | Writing tests, discussing test coverage | Fix broken tests before new tests, accuracy over coverage |
| `mvd-documentation` | Writing docs, comments, or explanations | Document "why" not "what", minimal viable documentation |
| `security-patterns` | Authentication, authorization, data handling, API design | OWASP compliance, security-first implementation |

## Red Flags: Stop and Check a Skill

If any of these thoughts cross your mind, STOP and consult the relevant skill:

| If You're Thinking... | Check This Skill |
|-----------------------|------------------|
| "Let me just add a quick workaround for now" | `production-code-standards` |
| "I'll create a new utility for this" | `service-reuse` |
| "Let me write some tests to get coverage up" | `testing-philosophy` |
| "I should document every function" | `mvd-documentation` |
| "I'll add authentication later" | `security-patterns` |
| "This fallback will handle edge cases" | `production-code-standards` |
| "The existing tests are flaky, I'll skip them" | `testing-philosophy` |
| "I need a helper function for this" | `service-reuse` |

## Working with Non-Engineers

Many users of this workflow are Product Managers and other non-engineers who are building production applications with AI assistance. This means:

- **Explain technical decisions** in plain language when relevant
- **Never assume familiarity** with programming concepts, patterns, or tooling
- **Provide context** for why certain approaches are required (security, maintainability, etc.)
- **Flag risks clearly** when suggesting approaches that require technical judgment

However, the production code standards apply equally regardless of who is building. The code that ships must be production-ready—no exceptions for "it's just a PM building this."
