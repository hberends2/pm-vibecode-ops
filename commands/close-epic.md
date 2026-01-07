---
allowed-tools: Bash(git diff:*), Bash(git status:*), Bash(git log:*), Bash(git show:*), Read, Glob, Grep, LS, Task, mcp__linear-server__get_issue, mcp__linear-server__update_issue, mcp__linear-server__create_comment, mcp__linear-server__list_comments, mcp__linear-server__list_issues, mcp__linear-server__list_projects
description: Close completed epic with retrofit analysis, downstream impact propagation, and CLAUDE.md updates
argument-hint: <epic-id> [--skip-retrofit] [--skip-downstream] (e.g., /close-epic EPIC-123)
workflow-phase: epic-closure
closes-epic: true
workflow-sequence: "/security-review (closes tickets) -> **/close-epic** (FINAL EPIC GATE)"
---

## MANDATORY: Agent Invocation Required

**You MUST use the Task tool to invoke the `epic-closure-agent` for this phase.**

### Step 1: Pre-Agent Context Gathering (YOU do this BEFORE invoking agent)

**As the orchestrator, YOU must gather ALL context before spawning the agent:**

1. **Fetch epic details**: Use `mcp__linear-server__get_issue` with epic ID
2. **Fetch all sub-tickets**: Use `mcp__linear-server__list_issues` with parent filter to get all child tickets
3. **Verify completion status**: Check that ALL sub-tickets are Done or Cancelled
4. **Fetch all comments**: Use `mcp__linear-server__list_comments` to get epic history
5. **Extract context for agent prompt:**
   - Epic ID, title, and full description
   - List of all sub-tickets with their final status
   - Phase reports from sub-tickets (implementation summaries, testing results, security findings)
   - Original business goals and success criteria from epic description
6. **Identify related epics**: Use `mcp__linear-server__list_issues` to find dependent/related epics

**IMPORTANT**: The agent does NOT have access to Linear. You must include ALL relevant context in the prompt.

### Step 2: Completion Verification (BLOCKING)

**Before invoking the agent, verify ALL sub-tickets are complete:**

```bash
# The orchestrator MUST verify:
# 1. All sub-tickets are in Done or Cancelled state
# 2. No sub-tickets are In Progress, Todo, or Blocked

# If ANY sub-ticket is incomplete:
# - DO NOT proceed with epic closure
# - Report which tickets need completion
# - Exit with clear guidance on next steps
```

**BLOCKING GATE**: If any sub-ticket is not Done/Cancelled, the epic CANNOT be closed.

### Step 3: Agent Invocation (Provide Full Context)

Use the Task tool to invoke the `epic-closure-agent` with ALL context embedded:

**Your prompt to the agent MUST include:**
- The epic ID for reference
- The full epic title and description (copy the text)
- Complete list of sub-tickets with their final status
- Implementation summaries from each sub-ticket
- Testing and security findings
- Original success criteria
- List of related/dependent epics (for downstream analysis)

**Example prompt structure:**
```
## Epic Context
**ID**: [epic-id]
**Title**: [title from get_issue]
**Description**:
[full description text from get_issue]

## Original Success Criteria
[extract from epic description]

## Sub-Ticket Summary
| Ticket ID | Title | Status | Key Outcomes |
|-----------|-------|--------|--------------|
| TICKET-1  | ...   | Done   | ...          |
| TICKET-2  | ...   | Done   | ...          |

## Implementation Summary
[aggregated from sub-ticket implementation reports]

## Testing Summary
[aggregated from sub-ticket testing reports]

## Security Summary
[aggregated from sub-ticket security reviews]

## Related Epics (for Downstream Analysis)
- EPIC-456: [title] - depends on this epic
- EPIC-789: [title] - related capability

## User Options
--skip-retrofit: [true/false based on user input]
--skip-downstream: [true/false based on user input]

## Your Task
Perform epic closure analysis following the six-phase workflow:
1. Verify completion (already done by orchestrator)
2. Retrofit analysis (unless --skip-retrofit)
3. Downstream impact (unless --skip-downstream)
4. Documentation audit
5. CLAUDE.md updates
6. Closure summary

Return a structured epic closure report when complete.
```

**CRITICAL**: Do NOT tell the agent to "fetch the epic" - the agent cannot access Linear.

### Step 4: Post-Agent Completion (YOU Write to Linear and Close Epic)

After the agent returns its report:

1. **Parse the agent's report** - Extract retrofit actions, downstream updates, CLAUDE.md changes
2. **Write the closure comment** - Use `mcp__linear-server__create_comment` with the structured closure report
3. **Update related epics** (if downstream analysis was performed):
   - Use `mcp__linear-server__create_comment` to add guidance to dependent epics
4. **Close the epic**:
   - Use `mcp__linear-server__update_issue` to mark epic as "Done"
   - Add appropriate labels (e.g., "epic-completed", "retrofit-complete")
5. **Apply CLAUDE.md updates** - Use Edit tool to update project CLAUDE.md
6. **Verify success** - Confirm the comment was added and epic is closed
7. **Report to user** - Summarize closure actions, retrofit findings, downstream propagation

**YOU are responsible for the Linear comment, epic closure, and CLAUDE.md updates, not the agent.**

DO NOT attempt to perform epic closure analysis directly. The specialized epic-closure-agent handles the analysis.

---

## Required Skills
- **epic-closure-validation** - Validates all tickets complete before closure
- **production-code-standards** - Ensures no temporary workarounds were accepted
- **verify-implementation** - Verify actual completion, not assumed

## Usage Examples

```bash
# Basic epic closure
/close-epic EPIC-123

# Skip retrofit analysis (faster closure)
/close-epic EPIC-123 --skip-retrofit

# Skip downstream impact propagation
/close-epic EPIC-123 --skip-downstream

# Minimal closure (skip both optional phases)
/close-epic EPIC-123 --skip-retrofit --skip-downstream
```

You are acting as an **Epic Closure Coordinator** responsible for formally closing completed epics, extracting lessons learned, and ensuring knowledge propagation across the project.

# Epic Closure: Final Gate for Epic Completion

**Epic closure is the final step after ALL sub-tickets have passed security review and been marked Done.**

- Prerequisites: Every sub-ticket under this epic is Done or Cancelled
- This command analyzes the completed work, propagates learnings, and formally closes the epic
- Unlike ticket closure (done by /security-review), epic closure requires cross-cutting analysis

**Workflow Position:** `All sub-tickets Done -> **Epic Closure** (YOU ARE HERE - FINAL EPIC GATE)`

---

## Pre-flight Checks
Before running:
- [ ] Linear MCP connected
- [ ] All sub-tickets are Done or Cancelled
- [ ] Epic is currently in "In Progress" state
- [ ] Project CLAUDE.md is accessible (if documentation updates needed)

## IMPORTANT: Linear MCP Integration (Orchestrator Responsibility)

**The orchestrator (YOU) handles ALL Linear MCP operations. The agent does NOT have access to Linear.**

**Tools you will use:**
- **Fetch epic**: `mcp__linear-server__get_issue` - YOU fetch before agent invocation
- **List sub-tickets**: `mcp__linear-server__list_issues` - YOU fetch before agent invocation
- **Fetch comments**: `mcp__linear-server__list_comments` - YOU fetch before agent invocation
- **Add comments**: `mcp__linear-server__create_comment` - YOU write after agent returns
- **Update status**: `mcp__linear-server__update_issue` - YOU update after agent returns (CLOSE EPIC!)

You are closing epic **$ARGUMENTS** (or **$1** if single argument provided).

## Six-Phase Epic Closure Workflow

### Phase 1: Completion Verification (BLOCKING)

**Orchestrator performs this BEFORE invoking the agent.**

```bash
# Verify all sub-tickets are complete
# Use mcp__linear-server__list_issues with parent filter

# Check criteria:
# - All sub-tickets in Done or Cancelled state
# - No tickets in Todo, In Progress, or Blocked
# - Security review passed on all implementation tickets
```

**If ANY ticket is incomplete:**
- DO NOT proceed
- List incomplete tickets
- Provide guidance on next steps

### Phase 2: Retrofit Analysis (Skippable with --skip-retrofit)

**Agent analyzes patterns that should propagate BACKWARD to existing code.**

The agent identifies:
- New patterns established during this epic that improve on existing code
- Architectural decisions that existing code should adopt
- Security patterns that existing related code should implement
- Test patterns that existing test suites should follow

**Output:** List of retrofit recommendations with:
- What pattern/approach to propagate
- Which existing files/components should adopt it
- Priority (P0-P3)
- Estimated effort

### Phase 3: Downstream Impact (Skippable with --skip-downstream)

**Agent identifies impacts on FUTURE work (dependent epics).**

The agent analyzes:
- Epics that depend on this epic's completion
- Epics that share architectural patterns
- Epics that will interact with newly created services/APIs

**Output:** Guidance comments to add to dependent epics:
- New services/APIs available for reuse
- Patterns established that should be followed
- Integration points and how to use them
- Lessons learned that apply to their scope

### Phase 4: Documentation Audit

**Agent maps implemented features against documentation needs.**

The agent checks:
- Do implemented features have adequate CLAUDE.md coverage?
- Are new services/patterns documented for future AI sessions?
- Are architectural decisions captured?
- Are integration points documented?

**Output:** Documentation gaps requiring CLAUDE.md updates.

### Phase 5: CLAUDE.md Updates

**Agent proposes specific CLAUDE.md changes based on audit.**

Update categories:
- **New Services**: Add to service inventory section
- **New Patterns**: Add to architectural patterns section
- **New APIs**: Add to integration points section
- **Lessons Learned**: Add to best practices/guidelines section

**Output:** Specific edit instructions for CLAUDE.md.

### Phase 6: Epic Closure

**Orchestrator creates closure summary and marks epic Done.**

Closure comment includes:
- Summary of completed work
- Business value delivered
- Retrofit recommendations (if generated)
- Downstream guidance propagated (if performed)
- CLAUDE.md updates applied
- Lessons learned

---

## Epic Closure Report Format

After completing the analysis, the orchestrator adds the following comment to the epic:

```markdown
## Epic Closure Report

### Status: COMPLETE

### Business Value Delivered
[Summary of what was accomplished vs. original goals]

### Sub-Ticket Summary
| Ticket | Title | Status | Key Outcomes |
|--------|-------|--------|--------------|
| TICK-1 | ... | Done | ... |
| TICK-2 | ... | Done | ... |

### Retrofit Analysis
**Patterns to Propagate Backward:**
1. **[Pattern Name]** - [Description]
   - Files to update: `path/to/existing/code`
   - Priority: P[0-3]
   - Estimated effort: [hours]

### Downstream Impact
**Guidance Added to Dependent Epics:**
- EPIC-456: Added integration guidance for [new service]
- EPIC-789: Added pattern guidance for [architectural decision]

### CLAUDE.md Updates Applied
- Added [new service] to service inventory
- Documented [pattern] in architectural patterns section
- Added [integration point] to API documentation

### Lessons Learned
1. [Lesson with actionable guidance]
2. [Lesson with actionable guidance]

### Metrics
- Tickets Completed: X/Y
- Total Implementation Time: ~X hours
- Security Issues Found/Fixed: X/X
- Test Coverage Achieved: X%

**Epic Closed**: [Date/Time]
**Closure Coordinator**: AI Epic Closure Agent
```

---

## CLAUDE.md Update Process

When the agent identifies CLAUDE.md updates:

1. **Read current CLAUDE.md**: Use Read tool to get current content
2. **Identify insertion points**: Find appropriate sections for each update
3. **Apply edits**: Use Edit tool to make targeted updates
4. **Verify changes**: Confirm edits were applied correctly

**CLAUDE.md Update Categories:**

```markdown
## Service Inventory
### [New Service Name]
- **Purpose**: [what it does]
- **Location**: `path/to/service`
- **API**: [key methods/endpoints]
- **Dependencies**: [what it requires]

## Architectural Patterns
### [Pattern Name]
- **When to use**: [conditions]
- **Implementation**: [how to apply]
- **Example**: [reference implementation]

## Integration Points
### [Integration Name]
- **Endpoint/Interface**: [how to connect]
- **Authentication**: [auth requirements]
- **Usage**: [typical use cases]
```

---

## Handling Edge Cases

### Partial Completion
If some tickets are Done but others are Cancelled:
- Epic can still be closed if business value was delivered
- Document what was descoped in closure report
- Note any follow-up epics created for descoped work

### Retrofit Declined
If user skips retrofit with `--skip-retrofit`:
- Note in closure report that retrofit analysis was skipped
- Document that existing code may benefit from patterns established

### Downstream Declined
If user skips downstream with `--skip-downstream`:
- Note in closure report that downstream propagation was skipped
- Dependent epics may need manual review for new integration points

### No CLAUDE.md Updates Needed
If documentation audit finds no gaps:
- Note "Documentation Complete" in closure report
- Skip Phase 5

---

## Critical: Epic Closure Finality

**After epic closure:**
- Epic status is "Done" and should not be reopened
- All retrofit recommendations are documented (not necessarily implemented)
- Downstream epics have been updated with relevant guidance
- CLAUDE.md reflects new patterns/services from this epic
- Lessons learned are captured for future reference

**Do NOT close the epic if:**
- Any sub-ticket is still In Progress, Todo, or Blocked
- Critical security issues remain unresolved
- Business value was not delivered (descope to new epic instead)

Your final reply must contain the epic closure report, confirmation of Linear updates, and summary of any CLAUDE.md changes applied.
