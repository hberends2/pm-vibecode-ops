# Git Worktree Migration Guide

**Purpose**: Migrate git worktree functionality from pm-vibecode-ops to ~/coachingai/.claude commands

**Target**: Add concurrent development capabilities using isolated worktrees per ticket

---

## Executive Summary

This migration adds **git worktree** support to enable true parallel development. Each ticket gets its own isolated working directory (`.worktrees/[ticket-id]`), allowing multiple AI agents to work concurrently without conflicts.

**Key Benefits:**
- ✅ Complete isolation - multiple tickets can be worked on simultaneously
- ✅ Zero conflicts - each worktree is independent
- ✅ Automatic management - commands handle worktree lifecycle
- ✅ Clean history - isolated commits until final merge
- ✅ Easy debugging - inspect any ticket's worktree independently

**Migration Scope:**
- 1 new file: `_worktree_helpers.md`
- 6 modified commands: `adaptation.md`, `implementation.md`, `testing.md`, `documentation.md`, `codereview.md`, `security_review.md`
- 0 modified agents: Agents work through commands, no changes needed

---

## Migration Overview

### Worktree Lifecycle in the Workflow

```
Adaptation → Implementation → Testing → Documentation → Code Review → Security Review
    ↓             ↓             ↓            ↓               ↓               ↓
  CREATE      Navigate      Navigate     Navigate        Navigate     MERGE + REMOVE
 .worktrees/  to worktree   to worktree  to worktree     to worktree  worktree → merge target
 TICKET-ID/                                                            (main or feature branch)
```

**Configurable Merge Target**: The merge target is determined by adaptation phase:
- If run from `main` branch → merges back to `main`
- If run from `feature/X` branch → merges back to `feature/X`
- Documented in Linear ticket for downstream commands to use

### File Changes Required

| File | Change Type | Description |
|------|-------------|-------------|
| `commands/_worktree_helpers.md` | **NEW** | Reusable bash functions for worktree operations |
| `commands/adaptation.md` | **MODIFY** | Add worktree creation section (lines 367-514) |
| `commands/implementation.md` | **MODIFY** | Add worktree navigation section (lines 56-196) |
| `commands/testing.md` | **MODIFY** | Add worktree context setup (lines 203-260) |
| `commands/documentation.md` | **MODIFY** | Add worktree context setup (lines 51-107) |
| `commands/codereview.md` | **MODIFY** | Add worktree context setup (lines 52-162) |
| `commands/security_review.md` | **MODIFY** | Add worktree merge and cleanup (lines 347-748) |

---

## Step-by-Step Migration

### Phase 1: Create Helper Functions File

**Action**: Create new file `~/coachingai/.claude/commands/_worktree_helpers.md`

**Source**: Copy from `pm-vibecode-ops/claude/commands/_worktree_helpers.md`

**Contents**: This file contains reusable bash functions:
- `get_worktree_path_from_linear()` - Extract worktree path from Linear comments
- `validate_worktree_exists()` - Validate worktree state
- `check_existing_worktree()` - Check for existing worktree and prompt user
- `create_worktree_for_ticket()` - Create new worktree with error handling
- `remove_worktree()` - Safely remove worktree
- `merge_worktree_to_main()` - Merge worktree branch to main
- `execute_in_worktree()` - Execute commands within worktree context
- `list_all_worktrees()` - List active worktrees
- `get_repo_root()` - Find repository root
- **NEW**: `get_merge_target_from_linear()` - Extract merge target from Linear ticket comments
- **NEW**: `detect_parent_branch()` - Detect if current branch is a feature branch for merge target

**Implementation**:
```bash
# Copy the entire _worktree_helpers.md file
cp /Users/brian/pm-vibecode-ops/claude/commands/_worktree_helpers.md \
   ~/coachingai/.claude/commands/_worktree_helpers.md
```

**Important**: This file is NOT executed directly - functions are referenced by other commands.

---

### Phase 2: Modify adaptation.md

**Target File**: `~/coachingai/.claude/commands/adaptation.md`

**Changes Required**: Replace the "Branch Creation Strategy" section

**Location in coachingai file**: Around line 445 (search for "## Branch Creation Strategy")

**Location in pm-vibecode-ops**: Lines 367-514 (section titled "## Worktree and Branch Creation Strategy")

**What to Replace**:
1. Remove existing "Branch Creation Strategy" section (approximately lines 445-520 in coachingai)
2. Insert new "Worktree and Branch Creation Strategy" section from pm-vibecode-ops

**Key Changes**:
- ❌ **Remove**: Traditional branch creation with `git branch` and `git checkout`
- ✅ **Add**: **Parent branch detection** - automatically detects if running from feature branch
- ✅ **Add**: **Configurable merge target** - worktrees merge to parent feature branch OR main
- ✅ **Add**: Worktree creation logic with `git worktree add` (from detected parent branch)
- ✅ **Add**: Worktree validation and error handling
- ✅ **Add**: `.gitignore` updates for `.worktrees/` directory
- ✅ **Add**: User prompting for existing worktrees (reuse vs recreate)

**New Section Title**: "## Worktree and Branch Creation Strategy"

**New Subsections**:
- Overview: Git Worktrees for Concurrent Development
- Branch Decision Based on Ticket Complexity
- Git Worktree and Branch Management Commands
- Worktree Management Guidelines

**Critical Addition - Worktree Path and Merge Target Documentation**:
The Linear adaptation report format must include worktree path AND merge target:
```markdown
### 🛠️ Development Environment
- **Worktree Path**: `[absolute-path-to-worktree]` ⚠️ **CRITICAL: All downstream commands read this path**
- **Branch**: `[branch-name]` (created in worktree)
- **Parent Branch**: `[parent-branch-name]` (what this branch was created from)
- **Merge Target**: `[merge-target-branch]` ⚠️ **CRITICAL: Security review will merge to this branch**
  - [If feature branch]: `feature/large-feature` (merges to parent feature branch, NOT main)
  - [If main]: `main` (merges directly to main)
- **Worktree Status**:
  - [If new worktree created]: ✅ New worktree created and validated
  - [If reusing worktree]: ♻️ Reusing existing worktree (user confirmed)

**Merge Flow**: When security review passes, the worktree branch will be merged to **`[merge-target-branch]`**, then the worktree will be removed. If merging to a feature branch, that feature branch must eventually be merged to main separately.
```

**Implementation**:
```bash
# 1. Open coachingai adaptation.md
# 2. Find "## Branch Creation Strategy" section
# 3. Delete from "## Branch Creation Strategy" to the next "##" heading
# 4. Copy lines 367-514 from pm-vibecode-ops/claude/commands/adaptation.md
# 5. Paste into coachingai/adaptation.md at the same location
```

---

### Phase 3: Modify implementation.md

**Target File**: `~/coachingai/.claude/commands/implementation.md`

**Changes Required**: Replace "Branch Management" section with "Worktree Context Loading"

**Location in coachingai file**: Around line 46 (search for "## Branch Management")

**Location in pm-vibecode-ops**: Lines 56-196 (sections "Worktree Context Loading" through "Worktree Cleanup and Exit")

**What to Replace**:
1. Remove existing "Branch Management" section (approximately lines 46-140 in coachingai)
2. Insert new worktree sections from pm-vibecode-ops

**Key Changes**:
- ❌ **Remove**: `git checkout` to switch branches
- ✅ **Add**: Extract worktree path from Linear ticket comments
- ✅ **Add**: Validate worktree exists and is healthy
- ✅ **Add**: Navigate to worktree with `cd "$WORKTREE_PATH"`
- ✅ **Add**: Safety checks to prevent commits to main repo
- ✅ **Add**: Return to original directory after work completes

**New Sections to Add**:
1. "## Worktree Context Loading" (replaces "Branch Management")
   - Worktree-Based Development overview
   - Loading Worktree Context from Linear
   - Validating Worktree Exists
   - Navigating to Worktree
2. "## Worktree Safety Check" (before commit process)
3. "## Worktree Cleanup and Exit" (at end of command)

**Critical Pattern**:
```bash
# Standard worktree setup pattern used in ALL ticket-level commands
TICKET_ID="$1"
REPO_ROOT=$(git rev-parse --show-toplevel)
WORKTREE_PATH="${REPO_ROOT}/.worktrees/${TICKET_ID}"

# Validate and navigate
cd "$WORKTREE_PATH"

# ... do work ...

# Return to original directory
cd "$ORIGINAL_DIR"
```

**Implementation**:
```bash
# 1. Open coachingai implementation.md
# 2. Find "## Branch Management" section
# 3. Delete from "## Branch Management" through "## Branch Safety Check"
# 4. Copy lines 56-196 from pm-vibecode-ops/claude/commands/implementation.md
# 5. Paste into coachingai/implementation.md, replacing deleted sections
# 6. Ensure "Worktree Cleanup and Exit" is added at end of workflow
```

---

### Phase 4: Modify testing.md

**Target File**: `~/coachingai/.claude/commands/testing.md`

**Changes Required**: Add "Worktree Context Setup" section before testing workflow

**Location in coachingai file**: Before "## Testing Workflow" (approximately line 203)

**Location in pm-vibecode-ops**: Lines 203-260 (section "Worktree Context Setup")

**What to Add**:
Insert new section "## Worktree Context Setup" with subsections for:
- Loading worktree context
- Validating worktree
- Navigating to worktree for tests

**Key Changes**:
- ✅ **Add**: Worktree validation before running tests
- ✅ **Add**: Navigation to worktree directory
- ✅ **Add**: Verification that we're in correct worktree
- ✅ **Modify**: Testing workflow to reference worktree context loading

**Testing Workflow Integration**:
Update step 2 in "## Testing Workflow" from:
```
2. **Branch Verification**: Confirm on feature branch (NOT main) using `git branch --show-current`
```

To:
```
2. **Worktree Context Loading**: Navigate to ticket's isolated worktree (see above section)
```

**Implementation**:
```bash
# 1. Open coachingai testing.md
# 2. Find "## Testing Workflow" section (around line 203)
# 3. Insert new section BEFORE "## Testing Workflow"
# 4. Copy lines 203-260 from pm-vibecode-ops/claude/commands/testing.md
# 5. Update step 2 in Testing Workflow to reference worktree context
```

---

### Phase 5: Modify documentation.md

**Target File**: `~/coachingai/.claude/commands/documentation.md`

**Changes Required**: Add "Worktree Context Setup" section before documentation workflow

**Location in coachingai file**: Before "## Documentation Workflow" (approximately line 51)

**Location in pm-vibecode-ops**: Lines 51-107 (section "Worktree Context Setup")

**What to Add**:
Insert new section "## Worktree Context Setup" with:
- Loading worktree context for documentation
- Validating worktree exists
- Navigating to worktree

**Key Changes**:
- ✅ **Add**: Worktree validation before generating docs
- ✅ **Add**: Navigation to worktree directory
- ✅ **Modify**: Documentation workflow to reference worktree context

**Documentation Workflow Integration**:
Update step 2 in workflow from "Branch Verification" to:
```
2. **Worktree Context Loading**: Navigate to ticket's isolated worktree (see above section)
```

**Implementation**:
```bash
# 1. Open coachingai documentation.md
# 2. Find the workflow section (around line 51)
# 3. Insert new section BEFORE the workflow
# 4. Copy lines 51-107 from pm-vibecode-ops/claude/commands/documentation.md
# 5. Update workflow steps to reference worktree context
```

---

### Phase 6: Modify codereview.md

**Target File**: `~/coachingai/.claude/commands/codereview.md`

**Changes Required**: Add "Worktree Context Setup" section before code review workflow

**Location in coachingai file**: Before code review workflow (approximately line 52)

**Location in pm-vibecode-ops**: Lines 52-162 (section "Worktree Context Setup")

**What to Add**:
Insert new section "## Worktree Context Setup" with:
- Loading worktree context for review
- Validating worktree exists
- Navigating to worktree

**Key Changes**:
- ✅ **Add**: Worktree validation before code review
- ✅ **Add**: Navigation to worktree directory
- ✅ **Modify**: Code review workflow to reference worktree context

**Implementation**:
```bash
# 1. Open coachingai codereview.md
# 2. Find the workflow section (around line 52)
# 3. Insert new section BEFORE the workflow
# 4. Copy lines 52-162 from pm-vibecode-ops/claude/commands/codereview.md
# 5. Update workflow steps to reference worktree context
```

---

### Phase 7: Modify security_review.md (MOST CRITICAL)

**Target File**: `~/coachingai/.claude/commands/security_review.md`

**Changes Required**: Add worktree merge and cleanup logic

**Location in coachingai file**: Multiple locations

**Location in pm-vibecode-ops**: Lines 347-748

**What to Add**:
1. "## Worktree Context Setup" section (before analysis - lines 347-379)
2. "## Merge Worktree and Cleanup" section (after security review passes - lines 640-748)

**Key Changes**:
- ✅ **Add**: Worktree context loading before security review
- ✅ **Add**: **Merge target extraction** from Linear ticket comments
- ✅ **Add**: Worktree merge to **configurable target** (main OR feature branch) after security passes
- ✅ **Add**: Worktree cleanup and removal
- ✅ **Add**: Remote branch deletion
- ✅ **Add**: Conflict resolution guidance (using merge target, not hardcoded main)
- ✅ **Add**: Warning message when merging to feature branch (reminder to merge feature→main later)
- ✅ **Critical**: This is the ONLY command that removes worktrees

**This is THE MOST IMPORTANT change** because security_review is the final gate that:
1. **Extracts merge target** from Linear ticket (feature branch OR main)
2. Merges worktree branch to **merge target** (configurable, not hardcoded main)
3. Pushes to **origin/[merge-target]**
4. Deletes remote branch
5. Removes worktree
6. Closes Linear ticket

**Merge Logic Flow**:
```bash
if security_review_passes_with_no_critical_issues; then
  1. Navigate to main repo (exit worktree)
  2. **Extract merge target from Linear ticket comments** (NEW)
  3. Checkout and update **merge target branch** (was: hardcoded main)
  4. Merge worktree branch to **merge target** with --no-ff
  5. Push to **origin/[merge-target]** (was: origin/main)
  6. **If merge target is feature branch, show reminder to merge to main** (NEW)
  7. Delete remote branch
  8. Remove worktree with git worktree remove
  9. Close Linear ticket
else
  Keep worktree for fixes
  Keep ticket open
fi
```

**Implementation**:
```bash
# 1. Open coachingai security_review.md
# 2. Add "Worktree Context Setup" section at line 347
# 3. Find "## CRITICAL: Linear Ticket Closure (FINAL GATE)" section
# 4. Replace/add merge logic from lines 640-748 of pm-vibecode-ops
# 5. Ensure merge only happens when security review PASSES
```

---

## Verification Checklist

After completing migration, verify each change:

### File Creation
- [ ] `_worktree_helpers.md` exists in `~/coachingai/.claude/commands/`
- [ ] File contains all 9 helper functions
- [ ] File includes usage examples

### adaptation.md
- [ ] "Worktree and Branch Creation Strategy" section present
- [ ] `.gitignore` update logic included
- [ ] User prompting for existing worktrees included
- [ ] Linear report format includes worktree path

### implementation.md
- [ ] "Worktree Context Loading" section replaces "Branch Management"
- [ ] "Worktree Safety Check" section before commits
- [ ] "Worktree Cleanup and Exit" section at end
- [ ] No `git checkout` commands remain

### testing.md
- [ ] "Worktree Context Setup" section before workflow
- [ ] Testing workflow step 2 references worktree context
- [ ] Tests run within worktree directory

### documentation.md
- [ ] "Worktree Context Setup" section before workflow
- [ ] Documentation workflow references worktree context
- [ ] Docs generated within worktree

### codereview.md
- [ ] "Worktree Context Setup" section before workflow
- [ ] Code review workflow references worktree context
- [ ] Review happens within worktree

### security_review.md (CRITICAL)
- [ ] "Worktree Context Setup" at beginning
- [ ] "Merge Worktree and Cleanup" section added
- [ ] Merge only happens when security passes
- [ ] Worktree removal logic included
- [ ] Remote branch deletion included
- [ ] Conflict resolution guidance included

---

## Testing the Migration

### Manual Test Plan

**Test 1: Adaptation Creates Worktree**
```bash
# Run adaptation for a test ticket
/adaptation TEST-123

# Verify worktree was created
ls -la .worktrees/TEST-123
git worktree list | grep TEST-123

# Check Linear comment includes worktree path
```

**Test 2: Implementation Uses Worktree**
```bash
# Run implementation for the same ticket
/implementation TEST-123

# Verify implementation happened in worktree
cd .worktrees/TEST-123
git log -1  # Should show implementation commit

# Verify main repo is unchanged
cd ../../
git status  # Should be clean
```

**Test 3: Security Review Merges and Cleans Up**
```bash
# Run security review (assuming it passes)
/security_review TEST-123

# Verify worktree was removed
ls -la .worktrees/TEST-123  # Should not exist

# Verify branch was merged to main
git log --oneline -5  # Should show merge commit

# Verify remote branch was deleted
git branch -r | grep TEST-123  # Should be empty
```

**Test 4: Concurrent Worktrees**
```bash
# Create worktrees for two different tickets
/adaptation TEST-456
/adaptation TEST-789

# Verify both exist independently
git worktree list  # Should show both

# Work on both simultaneously
/implementation TEST-456  # Works in .worktrees/TEST-456
/implementation TEST-789  # Works in .worktrees/TEST-789

# No conflicts because they're isolated
```

---

## Common Patterns and Conventions

### Standard Worktree Path Pattern
All commands use the same path pattern:
```bash
TICKET_ID="$1"
REPO_ROOT=$(git rev-parse --show-toplevel)
WORKTREE_PATH="${REPO_ROOT}/.worktrees/${TICKET_ID}"
```

### Standard Navigation Pattern
```bash
# Save current directory
ORIGINAL_DIR=$(pwd)

# Navigate to worktree
cd "$WORKTREE_PATH"

# ... do work ...

# Return to original directory
cd "$ORIGINAL_DIR"
```

### Standard Validation Pattern
```bash
if [ ! -d "$WORKTREE_PATH" ]; then
    echo "❌ ERROR: Worktree not found"
    echo "Run /adaptation $TICKET_ID first"
    exit 1
fi
```

### Branch Naming Convention
```bash
# Single ticket
BRANCH_NAME="feature/${TICKET_ID}-${description}"

# Multi-ticket project (not using worktrees yet)
BRANCH_NAME="project/${project-name}"
```

---

## Troubleshooting

### Issue: Worktree already exists

**Symptom**: `fatal: '.worktrees/TICKET-123' already exists`

**Solution**: The adaptation command includes user prompting:
```bash
# User is prompted with options:
# 1. Reuse existing worktree (continue current work)
# 2. Remove and recreate (⚠️ DESTROYS uncommitted changes)
# 3. Cancel (investigate manually)
```

### Issue: Worktree not registered with git

**Symptom**: Directory exists but `git worktree list` doesn't show it

**Solution**: Use repair command:
```bash
git worktree repair .worktrees/TICKET-123
```

The validation logic includes this automatically.

### Issue: Stale worktree metadata

**Symptom**: Worktree removed manually but git still tracks it

**Solution**: Prune stale worktrees:
```bash
git worktree prune
```

This is included in the cleanup process after merge.

### Issue: Merge conflicts during security review

**Symptom**: `git merge` fails with conflicts

**Solution**: Security review includes guidance:
```bash
# Conflicts detected - provide manual resolution steps:
echo "1. Review conflicts: git status"
echo "2. Resolve each conflict manually"
echo "3. Stage resolved files: git add [files]"
echo "4. Complete merge: git commit"
echo "5. Push changes: git push origin main"
echo "6. Re-run security review to complete cleanup"

# Abort merge to leave clean state
git merge --abort
```

---

## Benefits After Migration

### Concurrent Development
- ✅ Multiple tickets can be worked on simultaneously
- ✅ Each worktree is completely isolated
- ✅ No conflicts between concurrent sessions

### Clean History
- ✅ Each ticket produces clean, isolated commits
- ✅ Changes merged atomically to main
- ✅ Easy to review ticket-specific changes

### Easy Debugging
- ✅ Inspect any ticket's worktree independently
- ✅ Directory structure: `cd .worktrees/TICKET-123`
- ✅ Each worktree shows only relevant changes

### Automatic Cleanup
- ✅ Worktrees removed after successful merge
- ✅ Remote branches cleaned up automatically
- ✅ No manual worktree management needed

---

## Architecture Decisions

### Why Not Use Worktrees for Multi-Ticket Projects?

Current implementation only uses worktrees for **single tickets**, not multi-ticket projects.

**Reason**: Multi-ticket projects already have a shared branch (`project/name`), so worktree isolation isn't beneficial. All subtickets work in the same branch, making coordination easier.

**Future Enhancement**: Could extend to use worktrees for multi-ticket projects if needed, creating:
```
.worktrees/
├── PROJECT-123/          # Parent project worktree
├── PROJECT-123-SUB-1/    # Subticket worktree
└── PROJECT-123-SUB-2/    # Subticket worktree
```

### Why Document Worktree Path in Linear?

The worktree path is documented in Linear ticket comments because:
1. **Downstream commands need it**: Implementation, testing, documentation, code review, and security review all need to find the worktree
2. **Persistence**: Linear comments persist across sessions
3. **Discoverability**: Engineers can see where work is happening
4. **Pattern consistency**: All commands use `${REPO_ROOT}/.worktrees/${TICKET_ID}`

### Why Only Security Review Merges?

Security review is the **final gate** in the workflow. It's the only phase that:
1. Validates all quality gates passed
2. Has authority to close tickets
3. Performs the final merge to main

This ensures:
- ✅ No premature merging
- ✅ All quality checks complete
- ✅ Clean merge history
- ✅ Atomic ticket completion

---

## Feature Branch Workflow (Configurable Merge Target)

### Overview

The updated worktree system now supports **configurable merge targets**, allowing tickets to be developed within the context of a larger feature branch, then merged back to that feature branch (not main).

### Use Case: Epic-Level Feature Development

**Scenario**: You're building a large feature (e.g., "User Management System") that requires multiple tickets to complete. You want concurrent development on individual tickets, but all should merge to the feature branch first, then the feature branch merges to main when complete.

### Workflow Example

**Step 1: Create Parent Feature Branch**
```bash
# On main branch, create parent feature branch
git checkout main
git checkout -b feature/user-management-system
git push -u origin feature/user-management-system
```

**Step 2: Run Adaptation from Feature Branch**
```bash
# CRITICAL: Run adaptation WHILE ON the feature branch
git checkout feature/user-management-system

# Adaptation auto-detects you're on a feature branch
/adaptation TICKET-123

# Outputs:
# ✅ Parent feature branch detected: feature/user-management-system
#    This ticket will merge back to: feature/user-management-system (NOT main)
#    Worktree will be created from: feature/user-management-system
```

This creates:
- **Worktree**: `.worktrees/TICKET-123/`
- **Branch**: `feature/TICKET-123` (based from `feature/user-management-system`)
- **Merge Target**: `feature/user-management-system` (documented in Linear ticket)

**Step 3: Complete Ticket Workflow**
```bash
/implementation TICKET-123   # Works in worktree
/testing TICKET-123          # Works in worktree
/documentation TICKET-123    # Works in worktree
/codereview TICKET-123       # Works in worktree
/security_review TICKET-123  # Merges to feature/user-management-system, NOT main
```

Security review will:
1. Extract merge target: `feature/user-management-system`
2. Merge worktree to feature branch
3. Push to `origin/feature/user-management-system`
4. Remove worktree
5. Close ticket
6. **Show reminder**: "Remember to merge feature/user-management-system to main when ready"

**Step 4: Repeat for Other Tickets**
```bash
# Still on feature/user-management-system
/adaptation TICKET-124  # Merges back to feature/user-management-system
/adaptation TICKET-125  # Merges back to feature/user-management-system
# ... run full workflow for each ...
```

All tickets merge to `feature/user-management-system`, building up the feature incrementally.

**Step 5: Merge Feature Branch to Main**
```bash
# After all tickets complete
git checkout main
git merge --no-ff feature/user-management-system -m "Complete User Management System

All tickets implemented and tested:
- TICKET-123: User authentication
- TICKET-124: User profiles
- TICKET-125: User permissions

Ready for production"

git push origin main
git branch -d feature/user-management-system
git push origin --delete feature/user-management-system
```

### Benefits of Feature Branch Workflow

1. **Incremental Integration**: Each ticket merges to feature branch, ensuring compatibility
2. **Isolated Testing**: Feature branch can be deployed to staging for comprehensive testing
3. **Parallel Development**: Multiple developers/agents work on different tickets simultaneously
4. **Clean Main Branch**: Main only receives complete, tested features
5. **Easy Rollback**: If feature branch has issues, main remains unaffected
6. **Better Reviews**: Feature-level code review possible before main merge

### How Merge Target Detection Works

**Detection Logic** (in `adaptation.md`):
```bash
PARENT_BRANCH=$(git branch --show-current)

if [[ "$PARENT_BRANCH" =~ ^(feature|feat|project)/ ]] && \
   [[ "$PARENT_BRANCH" != "main" ]] && \
   [[ "$PARENT_BRANCH" != "master" ]]; then
    # On a feature branch - use it as merge target
    MERGE_TARGET="$PARENT_BRANCH"
    BASE_BRANCH="$PARENT_BRANCH"
else
    # Default: merge to main
    MERGE_TARGET="main"
    BASE_BRANCH="origin/main"
fi
```

**Supported Branch Patterns**:
- `feature/*` → Merges to feature branch
- `feat/*` → Merges to feat branch
- `project/*` → Merges to project branch
- `main` → Merges to main (default)
- `master` → Merges to main (default)

### Linear Ticket Documentation

The merge target is **documented in Linear ticket comments** by adaptation phase:

```markdown
### 🛠️ Development Environment
- **Worktree Path**: `/repo/.worktrees/TICKET-123`
- **Branch**: `feature/TICKET-123`
- **Parent Branch**: `feature/user-management-system`
- **Merge Target**: `feature/user-management-system` ⚠️ CRITICAL
```

Security review **reads this** to determine where to merge.

### Current Limitation: Placeholder Implementation

**IMPORTANT**: The current implementation uses a **placeholder** for extracting merge target from Linear:

```bash
# PLACEHOLDER in security_review.md:
MERGE_TARGET=$(git branch --list | grep -o "feature/[^[:space:]]*" | head -1)
if [ -z "$MERGE_TARGET" ]; then
    MERGE_TARGET="main"
fi
```

**Note**: For full Linear integration, use `mcp__linear-server__list_comments` to retrieve ticket comments and parse for the pattern: `**Merge Target**: \`[branch-name]\``. The current fallback implementation works for most cases.

---

## Summary

This migration adds git worktree support to enable concurrent development with complete isolation between tickets. The changes are focused on command files only - no agent modifications needed.

**Migration Effort**: ~2-3 hours
**Complexity**: Medium (mostly copy-paste with careful placement)
**Risk**: Low (backward compatible - existing workflows still work)

**Key Files**:
1. **NEW**: `_worktree_helpers.md` (copy entire file)
2. **MODIFY**: `adaptation.md` (replace branch creation section)
3. **MODIFY**: `implementation.md` (replace branch management section)
4. **MODIFY**: `testing.md` (add worktree context setup)
5. **MODIFY**: `documentation.md` (add worktree context setup)
6. **MODIFY**: `codereview.md` (add worktree context setup)
7. **MODIFY**: `security_review.md` (add worktree merge and cleanup)

**Result**: Concurrent ticket development with automatic worktree lifecycle management.
