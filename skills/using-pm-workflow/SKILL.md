---
name: using-pm-workflow
description: Use when starting any PM workflow session. Establishes skill-checking behavior before ANY action including clarifying questions. Activates at session start, before responding to requests, when switching contexts, or when unsure which command to use. Lists all skills, commands, and trigger conditions for Product Managers building production code.
---

# PM Workflow Bootstrap

This skill establishes the foundational behavior for all PM workflow sessions.

<EXTREMELY_IMPORTANT>
BEFORE responding to ANY request—including asking clarifying questions—you MUST:
1. Review the applicable skills
2. Identify which skills apply to the current context
3. Load and follow those skills' guidance
4. THEN proceed with your response
</EXTREMELY_IMPORTANT>

## Workflow Overview

**Project-Level (run once):**
1. `/generate_service_inventory` - Catalog existing code
2. `/discovery` - Analyze patterns and architecture
3. `/epic-planning` - Create business-focused epics
4. `/planning` - Decompose into sub-tickets

**Ticket-Level (run per ticket):**
5. `/adaptation` - Implementation guide
6. `/implementation` - Write production code
7. `/testing` - Build test suite
8. `/documentation` - Generate docs
9. `/codereview` - Quality review
10. `/security_review` - OWASP assessment (closes tickets)

## Session Start Checklist

1. Identify current workflow phase (project or ticket level)
2. Determine which command applies
3. Load relevant skills for that phase
4. Proceed with skill guidance active

## Core Skills

| Skill | When It Activates |
|-------|-------------------|
| `production-code-standards` | Writing or reviewing code |
| `service-reuse` | Creating new services/APIs |
| `testing-philosophy` | Writing tests |
| `mvd-documentation` | Writing docs/comments |
| `security-patterns` | Auth, data handling, APIs |

See `references/command-reference.md` for complete command details, skill triggers, and red flags.

This bootstrap skill ensures you always work within the full PM workflow system, not around it.
