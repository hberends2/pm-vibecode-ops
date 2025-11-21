# Git Worktree Integration Guide

## For Product Managers: Why This Matters

**You don't need to understand the technical details**—the workflow handles everything automatically. But here's what worktrees mean for you:

| Without Worktrees | With Worktrees |
|-------------------|----------------|
| Work on 1 ticket at a time | Work on multiple tickets simultaneously |
| Agents conflict when working in parallel | Zero conflicts between concurrent work |
| Must wait for one ticket to finish | Start new tickets anytime |
| Complex branch switching | Each ticket is fully isolated |

**What you experience**: Run `/adaptation TICKET-123` and the system creates an isolated workspace. When `/security_review` passes, changes merge automatically and cleanup happens invisibly.

**Bottom line**: Worktrees let you (or your team) work on 2, 3, or more tickets concurrently without any coordination overhead.

---

## Technical Overview

This workflow uses **git worktrees** to enable truly concurrent development without conflicts. Each ticket gets its own isolated working directory, allowing multiple AI agents or developers to work simultaneously without interfering with each other.

## What Are Git Worktrees?

Git worktrees allow you to have multiple working directories (worktrees) attached to the same repository. Each worktree can have a different branch checked out, enabling parallel development on multiple features.

**Traditional Workflow (without worktrees):**
```
repo/
├── .git/
├── src/
└── ...

Problem: Only one branch at a time, must stash/commit before switching
```

**Worktree Workflow:**
```
repo/                           # Main repository
├── .git/
├── src/
├── .worktrees/
│   ├── TICKET-101/            # Isolated worktree for ticket 101
│   │   ├── .git → ../../.git  # Links to main .git
│   │   ├── src/               # Independent working files
│   │   └── ...
│   ├── TICKET-102/            # Isolated worktree for ticket 102
│   │   ├── .git → ../../.git
│   │   ├── src/
│   │   └── ...
│   └── TICKET-103/            # Isolated worktree for ticket 103
│       └── ...
```

**Benefits:**
- ✅ Work on multiple tickets simultaneously
- ✅ No context switching, stashing, or branch conflicts
- ✅ Each ticket completely isolated
- ✅ Shared git objects (efficient disk usage)

---

## Workflow Commands and Worktree Lifecycle

### Phase 1: Adaptation - Worktree Creation

**Command**: `/adaptation TICKET-ID`

**What happens:**
1. Checks for existing worktree (asks user to reuse or recreate)
2. Creates `.worktrees/TICKET-ID` directory
3. Creates feature branch in worktree
4. Updates `.gitignore` to exclude `.worktrees/`
5. Documents worktree path in Linear ticket

**Example:**
```bash
/adaptation TICKET-123

# Output:
# ✅ Worktree created successfully!
#    Path: /path/to/repo/.worktrees/TICKET-123
#    Branch: feature/TICKET-123-user-authentication
```

**Linear Ticket Update:**
```markdown
### 🛠️ Development Environment
- **Worktree Path**: `/absolute/path/to/repo/.worktrees/TICKET-123`
- **Branch**: `feature/TICKET-123-user-authentication`
- **Worktree Status**: ✅ New worktree created and validated
```

### Phase 2-6: Development in Worktree

**Commands**:
- `/implementation TICKET-ID`
- `/testing TICKET-ID`
- `/documentation TICKET-ID`
- `/codereview TICKET-ID`

**What happens:**
1. Reads worktree path from Linear ticket
2. Validates worktree exists
3. Navigates to worktree automatically
4. Performs work within isolated directory
5. Commits to worktree's feature branch
6. Returns to original directory after completion

**User experience:**
```bash
# You run the command
/implementation TICKET-123

# Command automatically:
# 1. Loads worktree path from Linear
# 2. cd .worktrees/TICKET-123
# 3. Does implementation work
# 4. Commits changes
# 5. cd back to original directory

# You never need to manually navigate!
```

### Phase 7: Security Review - Merge & Cleanup

**Command**: `/security_review TICKET-ID`

**What happens when security review PASSES:**
1. Performs security assessment
2. Navigates to main repo
3. Checks out and updates main branch
4. Merges worktree branch to main (with --no-ff)
5. Pushes merged changes to origin/main
6. Deletes remote feature branch
7. Removes worktree directory
8. Prunes worktree metadata
9. Closes Linear ticket

**Merge commit format:**
```
Merge TICKET-123: Complete workflow

✅ All quality gates passed:
- Adaptation
- Implementation
- Testing
- Documentation
- Code Review
- Security Review (APPROVED)

Closes TICKET-123

Co-Authored-By: Claude <noreply@anthropic.com>
```

---

## Concurrent Development Scenarios

### Scenario 1: Two Agents, Two Features

```bash
# Terminal 1: Agent Alice works on authentication
cd /path/to/my-app
claude code

> /adaptation TICKET-101
> /implementation TICKET-101
# Works in .worktrees/TICKET-101/

# Terminal 2: Agent Bob works on payments (SIMULTANEOUSLY!)
cd /path/to/my-app
claude code

> /adaptation TICKET-102
> /implementation TICKET-102
# Works in .worktrees/TICKET-102/

# Both agents work at the same time with ZERO conflicts!
```

### Scenario 2: Pausing and Resuming Work

```bash
# Start work on TICKET-201
/adaptation TICKET-201
/implementation TICKET-201

# Need to switch to urgent TICKET-202
# NO NEED TO COMMIT OR STASH!
# Just leave TICKET-201 worktree as-is

# Start new ticket
/adaptation TICKET-202
/implementation TICKET-202

# Later, resume TICKET-201
# Worktree still exists with uncommitted changes
/adaptation TICKET-201
# System detects existing worktree
# User chooses "Reuse existing worktree"
# Continue where you left off!
```

### Scenario 3: Manual Inspection During Development

```bash
# While agent works on TICKET-303, you want to inspect code

# List active worktrees
git worktree list
# Output:
# /path/to/repo                  abc1234 [main]
# /path/to/repo/.worktrees/TICKET-303  def5678 [feature/TICKET-303-api]

# Navigate to worktree
cd .worktrees/TICKET-303

# Inspect files
ls -la
git status
git log

# View changes
git diff main

# Return to main repo
cd ../..

# Agent continues working - no interference!
```

---

## Troubleshooting

### Problem: "Worktree not found" error

**Symptoms:**
```
❌ ERROR: Worktree not found at /path/to/repo/.worktrees/TICKET-123
```

**Causes:**
1. Adaptation phase wasn't run for this ticket
2. Worktree was manually deleted
3. Wrong ticket ID

**Solution:**
```bash
# Check if worktree exists
ls -la .worktrees/

# List all active worktrees
git worktree list

# If worktree doesn't exist, run adaptation
/adaptation TICKET-123
```

### Problem: Worktree exists but corrupted

**Symptoms:**
```
❌ ERROR: Directory exists but is not a valid git worktree
```

**Solution:**
```bash
# Remove corrupted worktree forcefully
git worktree remove --force .worktrees/TICKET-123

# Recreate worktree
/adaptation TICKET-123
# Choose "Remove and recreate"
```

### Problem: Merge conflict during security review

**Symptoms:**
```
❌ ERROR: Merge conflict detected

Conflicts in files:
  src/services/auth.service.ts
  src/types/user.types.ts
```

**Solution:**
```bash
# Security review will abort the merge automatically
# Manual resolution required:

cd /path/to/repo
git checkout main
git merge feature/TICKET-123-description

# Resolve conflicts manually
# Edit conflicted files
vim src/services/auth.service.ts

# Stage resolved files
git add src/services/auth.service.ts src/types/user.types.ts

# Complete merge
git commit

# Push changes
git push origin main

# Manually remove worktree
git worktree remove .worktrees/TICKET-123

# Close Linear ticket manually
```

### Problem: Disk space running low

**Symptoms:**
Many old worktrees accumulating

**Solution:**
```bash
# List all worktrees
git worktree list

# Remove specific old worktree
git worktree remove .worktrees/OLD-TICKET-456

# Or force remove if needed
git worktree remove --force .worktrees/OLD-TICKET-456

# Clean up stale metadata
git worktree prune

# Find large worktrees
du -sh .worktrees/*
```

### Problem: "Branch already exists" during adaptation

**Symptoms:**
```
❌ ERROR: Failed to create worktree
Branch 'feature/TICKET-789-name' already exists
```

**Solution:**
```bash
# List branches
git branch -a | grep TICKET-789

# If branch exists but worktree doesn't, delete branch
git branch -D feature/TICKET-789-name

# Recreate worktree
/adaptation TICKET-789
```

---

## Best Practices

### 1. Always Use Commands (Don't Manual Manage)

**❌ Don't:**
```bash
# Manual worktree management
git worktree add .worktrees/TICKET-999 -b feature/manual
cd .worktrees/TICKET-999
# Work...
# Forget to document in Linear
```

**✅ Do:**
```bash
# Use workflow commands
/adaptation TICKET-999
# Automatically creates, documents, and tracks worktree
```

### 2. Trust the Workflow for Cleanup

**❌ Don't:**
```bash
# Manually remove worktrees
rm -rf .worktrees/TICKET-123
# This breaks git worktree tracking!
```

**✅ Do:**
```bash
# Let security review handle cleanup
/security_review TICKET-123
# Automatically merges and removes worktree

# Or if needed, use git command
git worktree remove .worktrees/TICKET-123
```

### 3. Monitor Active Worktrees

```bash
# Weekly cleanup check
git worktree list

# Expected output: Only active tickets
# If you see old completed tickets, clean up:
git worktree remove .worktrees/OLD-TICKET
```

### 4. Understand Disk Usage

**How much space do worktrees use?**
- Worktrees share `.git` objects (efficient!)
- Each worktree: ~size of working files only
- Example: 500MB repo + 3 worktrees = ~2GB total (not 2GB each!)

```bash
# Check disk usage
du -sh . # Main repo
du -sh .worktrees/* # Each worktree
```

### 5. Concurrent Session Limits

**Practical limits:**
- **1-5 concurrent worktrees**: Ideal for most teams
- **5-10 concurrent worktrees**: Fine for larger teams
- **10+ concurrent worktrees**: Consider cleanup schedule

---

## FAQ

### Q: Can I work in the main repo while worktrees exist?

**A:** Yes! The main repo continues to work normally. Worktrees are independent.

```bash
# In main repo
cd /path/to/repo
git status # Shows main branch

# In worktree
cd .worktrees/TICKET-123
git status # Shows feature/TICKET-123-name branch
```

### Q: What happens if I delete `.worktrees/` manually?

**A:** Don't! Use `git worktree remove` instead. Manual deletion breaks git's worktree tracking.

**Recovery if you did:**
```bash
# Git thinks worktrees still exist but they don't
git worktree prune # Cleans up metadata
```

### Q: Can I push from within a worktree?

**A:** Yes! Commands do this automatically. You can also do it manually:

```bash
cd .worktrees/TICKET-123
git push origin HEAD
```

### Q: Do worktrees affect remote collaborators?

**A:** No! Worktrees are local only. Remote sees only branches and commits.

### Q: Can I use worktrees with existing branches?

**A:** Yes, but the workflow creates new branches automatically in adaptation phase.

```bash
# If you have an existing branch
git worktree add .worktrees/TICKET-456 existing-branch-name
```

### Q: What if security review fails?

**A:** Worktree remains intact for fixes:

```bash
# Security review fails
# Fix issues in worktree
cd .worktrees/TICKET-123
# Make fixes
git commit -m "fix: security issue"
git push origin HEAD

# Re-run security review
cd /path/to/repo
/security_review TICKET-123
# If it passes, worktree is merged and removed
```

---

## Advanced Usage

### Inspecting Multiple Worktrees Simultaneously

```bash
# Open multiple terminals/tabs
# Terminal 1
cd .worktrees/TICKET-101
code . # Open in VSCode

# Terminal 2
cd .worktrees/TICKET-102
code . # Open in another VSCode window

# Compare implementations side-by-side!
```

### Sharing Worktrees (Don't!)

Worktrees are local development constructs. To share work:

```bash
# ❌ Don't share worktree directories
# ✅ Push branch and share PR

cd .worktrees/TICKET-123
git push origin HEAD
# Share PR link with team
```

### Worktree State Inspection

```bash
# Check worktree integrity
git worktree list --porcelain

# Output:
# worktree /path/to/repo
# HEAD abc123...
# branch refs/heads/main
#
# worktree /path/to/repo/.worktrees/TICKET-123
# HEAD def456...
# branch refs/heads/feature/TICKET-123-name
```

---

## Summary

**Worktrees transform concurrent development from:**
```
⚠️  Stash → Switch → Unstash → Work → Stash → Switch...
```

**To:**
```
✅ Just work! Each ticket has its own space.
```

**Key Takeaways:**
1. Worktrees created automatically in `/adaptation`
2. All development commands work within worktrees transparently
3. Security review merges and cleans up automatically
4. Multiple agents can work concurrently without conflicts
5. User never needs to manually manage worktrees

**The workflow handles all worktree complexity for you!**
