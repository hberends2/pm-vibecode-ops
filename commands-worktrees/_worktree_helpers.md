# Worktree Helper Functions

This file contains reusable bash functions for git worktree management across all workflow commands.

**DO NOT run this file directly** - these functions are sourced/used by other commands.

---

## Core Worktree Utilities

### Function: get_worktree_path_from_linear()

Extracts the worktree path from Linear ticket comments.

```bash
get_worktree_path_from_linear() {
    local ticket_id=$1

    # Use Linear MCP to get all comments
    # Search for "**Worktree Path**:" pattern in comments
    # This is added by the adaptation phase

    # Use mcp__linear-server__list_comments to get comments
    # Parse output for worktree path

    # Return: absolute path to worktree or empty string if not found

    echo "This function should be implemented by calling mcp__linear-server__list_comments"
    echo "and parsing for the pattern: **Worktree Path**: [path]"

    # Example output format:
    # /absolute/path/to/repo/.worktrees/TICKET-123
}
```

### Function: validate_worktree_exists()

Validates that a worktree exists and is in good state.

```bash
validate_worktree_exists() {
    local worktree_path=$1
    local ticket_id=$2

    # Check if worktree directory exists
    if [ ! -d "$worktree_path" ]; then
        echo "❌ ERROR: Worktree not found at $worktree_path"
        echo "Expected worktree for ticket $ticket_id but directory doesn't exist."
        echo ""
        echo "This usually means:"
        echo "1. The adaptation phase hasn't been run yet for this ticket"
        echo "2. The worktree was manually deleted"
        echo "3. The worktree path in Linear ticket is incorrect"
        echo ""
        echo "Solutions:"
        echo "- Run /adaptation $ticket_id to create the worktree"
        echo "- Check Linear ticket comments for correct worktree path"
        echo "- Run 'git worktree list' to see active worktrees"
        return 1
    fi

    # Validate it's actually a git worktree
    if [ ! -f "$worktree_path/.git" ]; then
        echo "❌ ERROR: Directory exists but is not a valid git worktree"
        echo "Path: $worktree_path"
        echo ""
        echo "The directory exists but doesn't appear to be a git worktree."
        echo "This may indicate corruption or manual modification."
        echo ""
        echo "Solution:"
        echo "- Remove the directory: rm -rf \"$worktree_path\""
        echo "- Re-run /adaptation $ticket_id to recreate worktree"
        return 1
    fi

    # Check if worktree is registered with git
    if ! git worktree list | grep -q "$worktree_path"; then
        echo "⚠️  WARNING: Worktree exists but not registered with git"
        echo "Path: $worktree_path"
        echo ""
        echo "Attempting to repair worktree registration..."
        git worktree repair "$worktree_path"

        if [ $? -eq 0 ]; then
            echo "✅ Worktree repaired successfully"
        else
            echo "❌ Failed to repair worktree"
            echo "Manual intervention required"
            return 1
        fi
    fi

    echo "✅ Worktree validated: $worktree_path"
    return 0
}
```

### Function: check_existing_worktree()

Checks for existing worktree and prompts user for action.

```bash
check_existing_worktree() {
    local ticket_id=$1
    local repo_root=$2
    local worktree_path="${repo_root}/.worktrees/${ticket_id}"

    if [ -d "$worktree_path" ]; then
        echo "⚠️  Existing worktree detected for ticket $ticket_id"
        echo "Path: $worktree_path"
        echo ""

        # Check worktree status
        cd "$worktree_path"
        local branch_name=$(git branch --show-current)
        local has_uncommitted=$(git status --porcelain | wc -l)
        local commit_count=$(git rev-list --count HEAD ^origin/main 2>/dev/null || echo "unknown")

        echo "Worktree Status:"
        echo "  Branch: $branch_name"
        echo "  Uncommitted changes: $has_uncommitted files"
        echo "  Commits ahead of main: $commit_count"
        echo ""

        # Return to original directory
        cd - > /dev/null

        # Use AskUserQuestion tool to prompt
        # Options:
        # 1. "Reuse existing worktree" - Continue with existing work
        # 2. "Remove and recreate" - Start fresh (warns about data loss)
        # 3. "Cancel operation" - Stop and let user investigate

        echo "ACTION REQUIRED: Use AskUserQuestion tool with options:"
        echo "1. Reuse existing worktree (continue current work)"
        echo "2. Remove and recreate (⚠️  DESTROYS uncommitted changes)"
        echo "3. Cancel (investigate manually)"

        # Return code indicates worktree exists
        return 1
    fi

    # No existing worktree
    return 0
}
```

### Function: create_worktree_for_ticket()

Creates a new worktree with proper error handling.

```bash
create_worktree_for_ticket() {
    local ticket_id=$1
    local repo_root=$2
    local branch_name=$3
    local worktrees_dir="${repo_root}/.worktrees"
    local worktree_path="${worktrees_dir}/${ticket_id}"

    echo "Creating worktree for ticket $ticket_id..."
    echo "  Repository: $repo_root"
    echo "  Branch: $branch_name"
    echo "  Worktree path: $worktree_path"
    echo ""

    # Ensure .worktrees directory exists
    mkdir -p "$worktrees_dir"

    # Ensure .gitignore excludes .worktrees/
    if [ -f "${repo_root}/.gitignore" ]; then
        if ! grep -q "^\.worktrees/" "${repo_root}/.gitignore"; then
            echo "⚠️  Adding .worktrees/ to .gitignore"
            echo ".worktrees/" >> "${repo_root}/.gitignore"
            echo ".worktree-registry.json" >> "${repo_root}/.gitignore"
        fi
    else
        echo "⚠️  No .gitignore found - creating one"
        cat > "${repo_root}/.gitignore" << 'EOF'
# Git worktrees for concurrent development
.worktrees/
.worktree-registry.json
EOF
    fi

    # Fetch latest from origin to ensure we have up-to-date refs
    echo "Fetching latest changes from origin..."
    git fetch origin

    # Create worktree with new branch from origin/main
    echo "Creating new worktree..."
    if git worktree add "$worktree_path" -b "$branch_name" origin/main; then
        echo "✅ Worktree created successfully"
        echo "   Path: $worktree_path"
        echo "   Branch: $branch_name"

        # Verify worktree was created correctly
        cd "$worktree_path"
        local current_branch=$(git branch --show-current)
        if [ "$current_branch" != "$branch_name" ]; then
            echo "❌ ERROR: Worktree branch mismatch"
            echo "   Expected: $branch_name"
            echo "   Got: $current_branch"
            cd - > /dev/null
            return 1
        fi
        cd - > /dev/null

        # Return success with worktree path
        echo "$worktree_path"
        return 0
    else
        echo "❌ ERROR: Failed to create worktree"
        echo "Common causes:"
        echo "1. Branch '$branch_name' already exists"
        echo "2. Worktree path already in use"
        echo "3. Git repository is in a bad state"
        echo ""
        echo "Troubleshooting:"
        echo "- Check existing worktrees: git worktree list"
        echo "- Check existing branches: git branch -a"
        echo "- Clean up stale worktrees: git worktree prune"
        return 1
    fi
}
```

### Function: remove_worktree()

Safely removes a worktree with validation.

```bash
remove_worktree() {
    local worktree_path=$1
    local force=${2:-false}

    echo "Removing worktree: $worktree_path"

    # Validate worktree exists
    if [ ! -d "$worktree_path" ]; then
        echo "⚠️  Worktree directory not found (already removed?)"
        return 0
    fi

    # Check for uncommitted changes unless force=true
    if [ "$force" != "true" ]; then
        cd "$worktree_path"
        if [ -n "$(git status --porcelain)" ]; then
            echo "❌ ERROR: Worktree has uncommitted changes"
            echo "Uncommitted files:"
            git status --short
            echo ""
            echo "Options:"
            echo "1. Commit changes: cd \"$worktree_path\" && git add . && git commit"
            echo "2. Force removal (loses changes): Pass force=true parameter"
            cd - > /dev/null
            return 1
        fi
        cd - > /dev/null
    fi

    # Remove worktree using git
    if [ "$force" = "true" ]; then
        git worktree remove "$worktree_path" --force
    else
        git worktree remove "$worktree_path"
    fi

    if [ $? -eq 0 ]; then
        echo "✅ Worktree removed successfully"
        return 0
    else
        echo "❌ Failed to remove worktree"
        echo "Try manual removal: git worktree remove --force \"$worktree_path\""
        return 1
    fi
}
```

### Function: merge_worktree_to_main()

Merges a worktree branch to main and handles conflicts.

```bash
merge_worktree_to_main() {
    local worktree_path=$1
    local ticket_id=$2
    local repo_root=$3

    echo "Merging worktree branch to main..."
    echo "  Worktree: $worktree_path"
    echo "  Ticket: $ticket_id"
    echo ""

    # Get branch name from worktree
    cd "$worktree_path"
    local branch_name=$(git branch --show-current)
    cd - > /dev/null

    echo "  Branch to merge: $branch_name"

    # Navigate to repo root
    cd "$repo_root"

    # Ensure we're on main
    git checkout main
    if [ $? -ne 0 ]; then
        echo "❌ ERROR: Failed to checkout main branch"
        return 1
    fi

    # Pull latest changes
    echo "Pulling latest changes from origin/main..."
    git pull origin main
    if [ $? -ne 0 ]; then
        echo "❌ ERROR: Failed to pull latest changes"
        return 1
    fi

    # Attempt merge
    echo "Attempting merge of $branch_name into main..."
    if git merge --no-ff "$branch_name" -m "Merge $ticket_id: Complete ticket workflow

✅ All phases complete:
- Adaptation
- Implementation
- Testing
- Documentation
- Code Review
- Security Review

Closes $ticket_id"; then
        echo "✅ Merge successful!"

        # Push to origin
        echo "Pushing merged changes to origin/main..."
        if git push origin main; then
            echo "✅ Changes pushed to origin/main"

            # Delete remote branch
            echo "Cleaning up remote branch..."
            git push origin --delete "$branch_name" 2>/dev/null || true

            cd - > /dev/null
            return 0
        else
            echo "❌ ERROR: Failed to push to origin/main"
            echo "Local merge was successful but push failed"
            echo "You may need to manually push: git push origin main"
            cd - > /dev/null
            return 1
        fi
    else
        echo "❌ ERROR: Merge conflict detected"
        echo ""
        echo "Conflicts in the following files:"
        git diff --name-only --diff-filter=U
        echo ""
        echo "AUTOMATIC MERGE FAILED - Manual intervention required"
        echo ""
        echo "Resolution steps:"
        echo "1. Review conflicts: git status"
        echo "2. Resolve each conflict manually"
        echo "3. Stage resolved files: git add [files]"
        echo "4. Complete merge: git commit"
        echo "5. Push changes: git push origin main"
        echo "6. Re-run security review to complete cleanup"
        echo ""

        # Abort merge to leave clean state
        git merge --abort

        cd - > /dev/null
        return 1
    fi
}
```

### Function: execute_in_worktree()

Executes commands within worktree context, then returns to original directory.

```bash
execute_in_worktree() {
    local worktree_path=$1
    shift  # Remove first argument, leaving command and its arguments
    local command_to_run="$@"

    # Validate worktree exists
    if [ ! -d "$worktree_path" ]; then
        echo "❌ ERROR: Worktree path does not exist: $worktree_path"
        return 1
    fi

    # Save current directory
    local original_dir=$(pwd)

    echo "Executing in worktree: $worktree_path"
    echo "Command: $command_to_run"
    echo ""

    # Navigate to worktree
    cd "$worktree_path"

    # Execute command
    eval "$command_to_run"
    local exit_code=$?

    # Return to original directory
    cd "$original_dir"

    if [ $exit_code -eq 0 ]; then
        echo "✅ Command completed successfully in worktree"
    else
        echo "❌ Command failed in worktree (exit code: $exit_code)"
    fi

    return $exit_code
}
```

### Function: list_all_worktrees()

Lists all active worktrees with status information.

```bash
list_all_worktrees() {
    echo "Active Git Worktrees:"
    echo "===================="
    echo ""

    git worktree list --porcelain | while IFS= read -r line; do
        if [[ $line == worktree* ]]; then
            path=${line#worktree }
            echo "📁 $path"
        elif [[ $line == branch* ]]; then
            branch=${line#branch refs/heads/}
            echo "   Branch: $branch"
        elif [[ $line == "" ]]; then
            echo ""
        fi
    done

    echo ""
    echo "To work in a specific worktree:"
    echo "  cd [worktree-path]"
    echo ""
    echo "To remove a worktree:"
    echo "  git worktree remove [worktree-path]"
}
```

### Function: get_repo_root()

Finds the repository root from any path.

```bash
get_repo_root() {
    # Get the root directory of the git repository
    local repo_root=$(git rev-parse --show-toplevel 2>/dev/null)

    if [ -z "$repo_root" ]; then
        echo "❌ ERROR: Not in a git repository"
        return 1
    fi

    echo "$repo_root"
    return 0
}
```

### Function: get_merge_target_from_linear()

Extracts the merge target branch from Linear ticket comments.

```bash
get_merge_target_from_linear() {
    local ticket_id=$1

    # Use Linear MCP to get all comments
    # Search for "**Merge Target**:" pattern in comments
    # This is added by the adaptation phase

    # Use mcp__linear-server__list_comments to get comments
    # Parse output for merge target

    # Return: branch name or "main" if not found

    echo "This function should be implemented by calling mcp__linear-server__list_comments"
    echo "and parsing for the pattern: **Merge Target**: \`[branch-name]\`"

    # Example output format:
    # feature/large-feature
    # or
    # main (default)
}
```

### Function: detect_parent_branch()

Detects if we're currently on a feature branch that should be the merge target.

```bash
detect_parent_branch() {
    # Get current branch
    local current_branch=$(git branch --show-current)

    # Check if we're on a feature/feat/project branch (not main/master)
    if [[ "$current_branch" =~ ^(feature|feat|project)/ ]] && \
       [[ "$current_branch" != "main" ]] && \
       [[ "$current_branch" != "master" ]]; then
        echo "$current_branch"
        return 0
    fi

    # Default to main
    echo "main"
    return 0
}
```

---

## Usage Examples

### Example 1: Creating a worktree in adaptation phase

```bash
# In adaptation command
TICKET_ID="LIN-123"
REPO_ROOT=$(get_repo_root)
BRANCH_NAME="feature/${TICKET_ID}-user-authentication"

# Check for existing worktree
if ! check_existing_worktree "$TICKET_ID" "$REPO_ROOT"; then
    # Worktree exists - user will be prompted
    # Handle response from AskUserQuestion
fi

# Create new worktree
WORKTREE_PATH=$(create_worktree_for_ticket "$TICKET_ID" "$REPO_ROOT" "$BRANCH_NAME")

# Document in Linear ticket
echo "**Worktree Path**: $WORKTREE_PATH"
```

### Example 2: Working in worktree during implementation

```bash
# In implementation command
TICKET_ID="LIN-123"
WORKTREE_PATH=$(get_worktree_path_from_linear "$TICKET_ID")

# Validate worktree
if ! validate_worktree_exists "$WORKTREE_PATH" "$TICKET_ID"; then
    exit 1
fi

# Execute implementation in worktree
execute_in_worktree "$WORKTREE_PATH" "npm run build && npm test"
```

### Example 3: Merging and cleaning up after security review

```bash
# In security review command
TICKET_ID="LIN-123"
WORKTREE_PATH=$(get_worktree_path_from_linear "$TICKET_ID")
REPO_ROOT=$(get_repo_root)

# Merge to main
if merge_worktree_to_main "$WORKTREE_PATH" "$TICKET_ID" "$REPO_ROOT"; then
    # Remove worktree after successful merge
    remove_worktree "$WORKTREE_PATH"
    echo "✅ Ticket $TICKET_ID: Merged and cleaned up"
else
    echo "❌ Merge failed - worktree preserved for manual resolution"
fi
```

---

## Error Handling

All functions include comprehensive error handling:

- ✅ Clear error messages with context
- ✅ Suggested remediation steps
- ✅ Non-zero exit codes on failure
- ✅ State validation before operations
- ✅ Graceful degradation where possible

## Integration with Linear MCP

These functions integrate with Linear MCP tools:

- `mcp__linear-server__get_issue` - Get ticket details
- `mcp__linear-server__list_comments` - Extract worktree path
- `mcp__linear-server__create_comment` - Document worktree operations
- `mcp__linear-server__update_issue` - Update ticket status after merge

---

**Note**: This is a utility file. Commands should source these functions and call them as needed. The actual MCP tool invocations happen in the calling commands, not in these bash functions.
