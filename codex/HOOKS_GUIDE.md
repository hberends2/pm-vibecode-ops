# Codex Hooks Guide

This guide explains how to replicate Claude Code hook behaviors manually in Codex.

---

## What Are Hooks?

In Claude Code, hooks are automated triggers that run at specific events during a session. They enforce quality standards without requiring manual intervention.

**Codex does not have a hook system.** However, you can replicate these behaviors manually by following the checklists in this guide.

---

## Claude Code Hooks Overview

The PM workflow defines three hook types in `hooks/hooks.json`:

| Hook Type | Trigger | Purpose |
|-----------|---------|---------|
| `SessionStart` | Session begins (startup, resume, clear, compact) | Inject workflow context and skill reminders |
| `PostToolUse` | After specific tool calls | Validate actions (e.g., epic closure prerequisites) |
| `Stop` | Before ending session | End-of-session summary with code review notes |

---

## SessionStart Hook

### Claude Code Behavior

When a Claude Code session starts (or resumes/clears/compacts), the hook runs `scripts/session-start.sh` which injects:

- List of all 10 available skills with descriptions
- Workflow command sequence (project-level and ticket-level)
- Critical reminders:
  - Users may be non-engineers - explain decisions clearly
  - All code must be production-ready
  - Skills BLOCK prohibited patterns
  - Only `/security-review` closes tickets

### Codex Alternative

**At the start of each Codex session**, review this context checklist:

#### Session Start Checklist

```markdown
## Session Context (Include at Session Start)

### Available Skills
- production-code-standards: Blocks workarounds, temporary code, fallbacks, TODO comments
- service-reuse: Check service inventory before creating ANY new service
- testing-philosophy: Fix broken tests BEFORE writing new tests
- mvd-documentation: Document "why" not "what"; no placeholder content
- security-patterns: Auth on every endpoint; parameterized queries; input validation
- model-aware-behavior: Read ALL files before proposing changes
- verify-implementation: Verify work before marking tasks complete
- divergent-exploration: Explore alternatives before converging
- epic-closure-validation: All sub-tickets must be Done/Cancelled before epic closure
- using-pm-workflow: Follow workflow phases correctly

### Workflow Commands (in sequence)

**Project-Level:**
1. generate-service-inventory - Catalog existing services
2. discovery - Analyze patterns and architecture
3. epic-planning - Create business-focused epics
4. planning - Decompose epics into technical tickets

**Ticket-Level (for each ticket):**
5. adaptation - Create implementation guide
6. implementation - Write production code
7. testing - Build comprehensive test suite
8. documentation - Generate API docs
9. codereview - Quality assurance review
10. security-review - OWASP assessment (ONLY this phase closes tickets)

### Critical Reminders
- Explain technical decisions clearly for non-engineers
- ALL code must be production-ready - no shortcuts
- Only security-review closes tickets
```

#### Practical Application

**Option 1**: Keep this checklist in a file and review it when starting Codex:

```bash
# Create session start reference
cat > ~/codex-session-start.md << 'EOF'
[Paste checklist above]
EOF

# Review before each session
cat ~/codex-session-start.md
```

**Option 2**: Include abbreviated context in your prompt:

```markdown
## Session Context
Following PM workflow. Current phase: [implementation/testing/etc.]
Key skills active: production-code-standards, service-reuse
Reminder: All code production-ready, no workarounds.

---

[Your actual prompt]
```

---

## PostToolUse Hook: Epic Closure Validation

### Claude Code Behavior

When `mcp__linear-server__update_issue` is called to mark an EPIC as Done, the hook validates:

1. **All sub-tickets are Done or Cancelled** - No incomplete work
2. **No workarounds shipped** - Production standards maintained
3. **Business value delivered** - Original success criteria met

If validation fails, the hook **BLOCKS** the epic closure and reports which tickets need attention.

### Codex Alternative

**Before closing any epic**, manually verify these requirements:

#### Epic Closure Checklist

```markdown
## Epic Closure Validation

Before marking epic [EPIC-XXX] as Done:

### 1. Sub-Ticket Status Check
- [ ] List all sub-tickets: `[List ticket IDs]`
- [ ] Verify each ticket is Done or Cancelled
- [ ] Document any incomplete tickets and their blockers

**If any tickets are incomplete: STOP. Do not close the epic.**

### 2. Production Standards Check
- [ ] Review each completed ticket for workarounds
- [ ] Verify no TODO/FIXME comments were shipped
- [ ] Confirm no temporary solutions in production code
- [ ] Check that no "quick fixes" were left unresolved

**If workarounds exist: Create follow-up tickets before closing.**

### 3. Business Value Verification
- [ ] Review original epic description and success criteria
- [ ] Verify all success criteria have been met
- [ ] Document any scope changes and their justification

**If business value not delivered: Do not close until resolved.**

### Validation Result
- [ ] PASS - All checks complete, epic can be closed
- [ ] FAIL - Issues found: [describe blockers]
```

#### Practical Application

Create a template file for epic closure:

```bash
# Create epic closure checklist template
cat > ~/epic-closure-checklist.md << 'EOF'
# Epic Closure: [EPIC-ID]

## Sub-Ticket Status
| Ticket | Status | Notes |
|--------|--------|-------|
| | | |

All Done/Cancelled? [ ] Yes [ ] No

## Workaround Check
Reviewed for TODO/FIXME/HACK: [ ] Yes
Temporary code found: [ ] Yes [ ] No
If yes, follow-up tickets created: [ ] Yes [ ] N/A

## Business Value
Original success criteria met: [ ] Yes [ ] No
Scope changes documented: [ ] Yes [ ] N/A

## Decision
[ ] APPROVED - Close epic
[ ] BLOCKED - Reason: _______________
EOF
```

---

## Stop Hook: End-of-Session Summary

### Claude Code Behavior

Before stopping, Claude Code provides a summary with two parts:

1. **Code Review Notes**: Checks `git diff --name-only` for modified production files (src/, lib/, app/, services/, modules/, controllers/). For code files (.ts, .js, .tsx, .jsx), notes any:
   - TODO/FIXME comments
   - Empty catch blocks
   - console.log statements

   Skips: scripts/, tests/, config files, documentation, non-code changes.

2. **Linear Ticket Status**: If workflow commands were used on a Linear ticket, reminds to verify ticket status is updated.

### Codex Alternative

**Before ending your Codex session**, perform this manual review:

#### End-of-Session Checklist

```markdown
## End-of-Session Review

### Part 1: Code Review Notes

**Check modified files:**
```bash
git diff --name-only
```

**For production code files (src/, lib/, app/, services/):**
- [ ] Search for TODO/FIXME: `grep -rn "TODO\|FIXME" [modified-files]`
- [ ] Search for empty catches: Review catch blocks in diffs
- [ ] Search for console.log: `grep -rn "console.log" [modified-files]`

**Findings:**
- TODO/FIXME comments: [ ] None [ ] Found: ___
- Empty catch blocks: [ ] None [ ] Found: ___
- console.log statements: [ ] None [ ] Found: ___

**Skip if changes only in:** scripts/, tests/, __tests__/, *.test.*, *.spec.*, config, docs

### Part 2: Linear Ticket Status

**Did this session use workflow commands on a ticket?**
- [ ] No - Skip this section
- [ ] Yes - Ticket ID: ___

**Ticket status verification:**
- [ ] Ticket status reflects current progress
- [ ] Comments added for significant work
- [ ] Labels updated if phase completed
```

#### Practical Application

**Quick command-line check before ending session:**

```bash
# Check for common issues in your changes
echo "=== Modified Files ==="
git diff --name-only

echo ""
echo "=== TODO/FIXME in Changes ==="
git diff | grep -n "TODO\|FIXME" || echo "None found"

echo ""
echo "=== console.log in Changes ==="
git diff | grep -n "console.log" || echo "None found"

echo ""
echo "=== Empty catch blocks (manual review needed) ==="
git diff | grep -A2 "catch" | head -20
```

Save this as a script:

```bash
# Create end-of-session check script
cat > ~/codex-session-end.sh << 'EOF'
#!/bin/bash
echo "=== End-of-Session Code Review ==="
echo ""
echo "Modified files:"
git diff --name-only
echo ""
echo "Checking for issues..."
echo ""
echo "TODO/FIXME comments:"
git diff | grep -n "TODO\|FIXME" || echo "  None found"
echo ""
echo "console.log statements:"
git diff | grep -n "console.log" || echo "  None found"
echo ""
echo "Review complete. Check empty catch blocks manually if needed."
EOF

chmod +x ~/codex-session-end.sh

# Run before ending session
~/codex-session-end.sh
```

---

## Complete Manual Workflow Checklist

Combine all hook behaviors into a single workflow:

### At Session Start

1. Review available skills and workflow phases
2. Identify which phase you're working on
3. Include relevant skill content in your prompt
4. Note any active Linear tickets

### During Session

1. Follow skill enforcement (production-code-standards, etc.)
2. Verify work before claiming completion (verify-implementation)
3. If closing an epic, complete the Epic Closure Checklist

### Before Ending Session

1. Run the end-of-session code review
2. Address any TODO/FIXME/console.log findings
3. Verify Linear ticket status is updated
4. Commit clean code or note remaining work

---

## Automation Options for Codex

While Codex doesn't have native hooks, you can create shell automation:

### Pre-Session Script

```bash
#!/bin/bash
# codex-pre-session.sh

echo "=== PM Workflow Session Start ==="
echo ""
echo "Available skills: production-code-standards, service-reuse, testing-philosophy,"
echo "  mvd-documentation, security-patterns, model-aware-behavior, verify-implementation,"
echo "  divergent-exploration, epic-closure-validation, using-pm-workflow"
echo ""
echo "What phase are you working on?"
echo "  Project: discovery, epic-planning, planning"
echo "  Ticket:  adaptation, implementation, testing, documentation, codereview, security-review"
echo "  Epic:    close-epic"
echo ""
echo "Remember: Only security-review closes tickets!"
```

### Post-Session Script

```bash
#!/bin/bash
# codex-post-session.sh

echo "=== PM Workflow Session End ==="
echo ""

# Check for issues
if git diff --name-only | grep -qE '^(src|lib|app|services)/.*\.(ts|js|tsx|jsx)$'; then
    echo "Production code modified. Checking for issues..."
    echo ""

    issues=$(git diff | grep -c "TODO\|FIXME\|console\.log")
    if [ "$issues" -gt 0 ]; then
        echo "WARNING: Found $issues potential issues in your changes:"
        git diff | grep -n "TODO\|FIXME\|console\.log"
        echo ""
        echo "Consider addressing these before committing."
    else
        echo "No TODO/FIXME/console.log found. Code looks clean."
    fi
else
    echo "No production code changes detected."
fi

echo ""
echo "Did you update your Linear ticket status? (y/n)"
```

### Alias Setup

```bash
# Add to ~/.bashrc or ~/.zshrc
alias codex-start='~/codex-pre-session.sh && codex'
alias codex-end='~/codex-post-session.sh'
```

---

## Summary

| Hook | Claude Code | Codex Alternative |
|------|-------------|-------------------|
| SessionStart | Auto-injects context | Review session start checklist |
| PostToolUse (Epic) | Blocks invalid epic closure | Complete epic closure checklist |
| Stop | Auto-reviews code changes | Run end-of-session review script |

The key difference: **You are responsible for enforcement in Codex.** The checklists and scripts in this guide help you maintain the same quality standards, but require manual discipline to apply consistently.
