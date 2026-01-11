---
name: model-aware-behavior
description: |
  Enforces read-before-write discipline and prevents scope creep. Use when:
  - Modifying code: "modify", "change", "update", "edit", "refactor", "fix", "implement"
  - Exploring code: "explore codebase", "understand this code", "how does this work"
  - About to edit any code file without first reading it
  - Proposing changes to files not yet examined

  Enforces reading ALL files before proposing changes. Never speculate about unread code.
  Do ONLY what is requested - no "while I'm here" improvements, no gold-plating, no scope creep.
---

# Model-Aware Behavior

This skill enforces disciplined development practices that ensure code quality and efficiency.

## Code Exploration (Before Any Changes)

**MANDATORY before proposing any code changes:**

1. Read all relevant files before proposing changes
2. Never speculate about code not yet inspected
3. If user references a file path, read it before responding
4. Search for existing implementations before creating new ones
5. Understand existing patterns before implementing features

**Verification checklist:**
- Verify all files to be modified have been read
- Confirm understanding of existing codebase patterns
- Check for similar existing implementations before creating new ones

**Example - Correct approach:**
```
User: "Add email validation to the signup form"

WRONG: Immediately write validation code
RIGHT:
1. Read signup form: src/components/SignupForm.tsx
2. Search for existing validation: grep -r "validation\|validate" src/
3. Find existing pattern: src/utils/validators.ts
4. Use existing validateEmail() from validators.ts
```

## Scope Control

**Do ONLY what is requested:**

- Make only requested changes
- No unrequested improvements, refactoring, or features
- No helpers/utilities for one-time operations
- No error handling for impossible scenarios
- No design for hypothetical future requirements
- Reuse existing abstractions

**Red flags to avoid:**
- "While I'm here, I'll also..."
- "It would be better to also..."
- "For future extensibility..."
- Adding abstractions for single-use code

**Example - Scope discipline:**
```
User: "Fix the typo in the button label"

WRONG:
- Fix typo (check)
- Refactor button component (not requested)
- Add aria-label (not requested)
- Update tests (not requested)

RIGHT:
- Fix typo (check)
- Done
```

## File Operation Efficiency

**Read before write:**

Always read files before modifying them. Serialize dependent operations (read then edit), but parallelize independent operations (reading multiple unrelated files).

**Examples:**
- Reading multiple unrelated files -> parallel
- Searching across multiple directories -> parallel
- Reading file THEN editing it -> sequential (edit depends on read)
- Creating directory THEN writing file inside -> sequential

---

## How to Use This Skill in Codex

Include this skill's content in your Codex prompt when:
- About to modify any code
- Working on bug fixes or feature additions
- Exploring an unfamiliar codebase
- Reviewing proposed changes for scope creep

Copy the verification checklist and scope control rules to enforce disciplined development.
