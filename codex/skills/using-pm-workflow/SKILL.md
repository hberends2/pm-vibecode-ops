---
name: using-pm-workflow
description: |
  Guides users through PM workflow phases and command sequence. Use when:
  - Asking for guidance: "what command", "which phase", "where do I start", "what's next", "workflow"
  - Starting work: "new project", "new session", "help me plan", "kick off", "get started"
  - Learning the workflow: "how do I use this", "what are the commands", "workflow overview"
  - Discussing phases: discovery, planning, adaptation, implementation, testing, codereview
  - Context switch between project-level and ticket-level work

  Provides workflow sequence guidance (discovery -> epic-planning -> planning -> adaptation ->
  implementation -> testing -> documentation -> codereview -> security-review).
---

# PM Workflow Bootstrap

This skill establishes the foundational behavior for all PM workflow sessions.

**IMPORTANT**: Before responding to ANY request:
1. Review the applicable skills
2. Identify which skills apply to the current context
3. Follow those skills' guidance
4. THEN proceed with the response

## Workflow Overview

**Project-Level (run once):**
1. **Generate Service Inventory** - Catalog existing code
2. **Discovery** - Analyze patterns and architecture
3. **Epic Planning** - Create business-focused epics
4. **Planning** - Decompose into sub-tickets

**Ticket-Level (run per ticket):**
5. **Adaptation** - Implementation guide
6. **Implementation** - Write production code
7. **Testing** - Build test suite
8. **Documentation** - Generate docs
9. **Code Review** - Quality review
10. **Security Review** - OWASP assessment (closes tickets)

**Epic-Level (after all tickets complete):**
11. **Close Epic** - Close epic with retrofit analysis

## Session Start Checklist

1. Identify current workflow phase (project or ticket level)
2. Determine which step applies
3. Load relevant skills for that phase
4. Proceed with skill guidance active

**Quick Decision Tree:**
```
Starting new project?
  -> Discovery -> Epic Planning -> Planning

Working on a ticket?
  -> Adaptation -> Implementation -> Testing
  -> Documentation -> Code Review -> Security Review

All tickets in epic done?
  -> Close Epic (retrofit analysis, creates follow-up tickets)

"Where do I start?"
  -> Check if service inventory exists
  -> No: Generate Service Inventory first
  -> Yes: Discovery
```

## Core Skills by Phase

| Skill | When It Activates |
|-------|-------------------|
| `production-code-standards` | Writing or reviewing code |
| `service-reuse` | Creating new services/APIs |
| `testing-philosophy` | Writing tests |
| `mvd-documentation` | Writing docs/comments |
| `security-patterns` | Auth, data handling, APIs |
| `verify-implementation` | Claiming work is complete |
| `divergent-exploration` | Design decisions, architecture |

## Ticket Closure Rules

- **Only Security Review closes tickets** - marks ticket as "Done" when no critical/high issues found
- Documentation, code review, and other phases do NOT close tickets
- Ticket remains "In Progress" until security review completes successfully

## Epic Closure Rules

- **Only Close Epic closes epics** - marks epic as "Done" when all sub-tickets are Done/Cancelled
- Includes retrofit analysis, downstream impact propagation
- Creates retrofit tickets for lessons learned
- Epic remains "In Progress" until all sub-tickets pass security review

---

## How to Use This Skill in Codex

Include this skill's content in your Codex prompt when:
- Starting a new project or session
- Unsure which phase you're in
- Need guidance on workflow sequence
- Switching between project-level and ticket-level work

Copy the workflow overview and decision tree to maintain proper workflow sequence.
