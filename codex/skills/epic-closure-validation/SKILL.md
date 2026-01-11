---
name: epic-closure-validation
description: |
  Validates epic completion before closure. Use when:
  - Closing epics: "close epic", "finish epic", "complete epic", "epic done", "mark epic complete"
  - Checking status: "are all tickets done", "check epic status", "verify epic completion"
  - Discussing epic IDs with closure intent (EPIC-*, EPc-*)
  - Epic closure workflows

  Blocks closing epics with incomplete sub-tickets. Requires all sub-tickets Done/Cancelled
  before epic can be closed. Prevents premature closure and ensures business value delivered.
---

# Epic Closure Validation

Validates that ALL sub-tickets are complete before an epic can be closed.

## Enforcement Workflow

1. **Before closure**: Verify all sub-tickets are Done or Cancelled
2. **Block if incomplete**: List incomplete tickets and required actions
3. **Check for workarounds**: Verify no temporary solutions shipped
4. **Validate business value**: Ensure original goals were met

## Blocking Conditions

| Condition | Action |
|-----------|--------|
| Sub-ticket In Progress | BLOCK - Must complete or cancel |
| Sub-ticket Todo | BLOCK - Must complete or cancel |
| Sub-ticket Blocked | BLOCK - Must resolve blocker |
| Workaround shipped | BLOCK - Must fix before closure |
| Security issue unresolved | BLOCK - Must address finding |

## Required Before Closure

| Check | Requirement |
|-------|-------------|
| Sub-tickets | ALL must be Done or Cancelled |
| Security reviews | ALL implementation tickets passed |
| Workarounds | NONE in production code |
| Business value | Original goals must be met |

## When Closure is Blocked

If epic closure is blocked:

1. **LIST** all incomplete tickets with their current status
2. **IDENTIFY** what action is needed for each
3. **DO NOT** proceed with closure analysis
4. **REPORT** clear guidance on next steps

**Block epic closure when work is incomplete. Report all blocking items with clear guidance on required next steps.**

## Valid Closure Scenarios

| Scenario | Valid? | Notes |
|----------|--------|-------|
| All tickets Done | Yes | Standard closure |
| Mix of Done + Cancelled | Yes | If business value delivered |
| Some tickets In Progress | NO | Must complete first |
| All tickets Cancelled | Maybe | Only if epic descoped entirely |

## Retrofit Analysis

When closure IS valid, include:

1. **What worked well** - Patterns to replicate
2. **What could improve** - Process adjustments
3. **Downstream impacts** - Other epics/systems affected
4. **Follow-up tickets** - New work identified during epic

## Creating Retrofit Tickets

Each significant finding from retrofit analysis becomes a ticket:

```markdown
## Retrofit: [Finding Title]

**Context**: [Which epic, what was learned]
**Recommendation**: [Specific action to take]
**Impact**: [What improves if addressed]
**Effort**: [T-shirt size estimate]
**Priority**: [Suggested priority]
```

---

## How to Use This Skill in Codex

Include this skill's content in your Codex prompt when:
- About to close an epic
- Reviewing epic status for completion
- Performing retrofit analysis
- Creating follow-up work from completed epics

Copy the blocking conditions checklist to ensure no incomplete epics are closed prematurely.
