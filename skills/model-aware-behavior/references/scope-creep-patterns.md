# Scope Creep Anti-Patterns

This reference documents common scope creep patterns to avoid when making code changes.

## Red Flags in Thinking

Stop immediately if about to say or think:
- "While I'm here, I'll also..."
- "It would be better to also..."
- "For future extensibility..."
- "Let me also optimize..."
- "I noticed this could use..."
- "This would be a good time to..."
- "Since we're touching this file..."

## Scope Creep Examples

### Example 1: The Helpful Refactor
**Request:** "Fix the typo in the button label"

**Scope creep (WRONG):**
- Fix typo ✓
- Refactor button component to use new design system
- Add aria-label for accessibility
- Update tests for new component structure
- Add documentation

**Correct (RIGHT):**
- Fix typo ✓
- Done

### Example 2: The Future-Proof Addition
**Request:** "Add email validation to the signup form"

**Scope creep (WRONG):**
- Add email validation ✓
- Create reusable validation library
- Add phone validation "while we're at it"
- Build validation configuration system
- Add i18n support for error messages

**Correct (RIGHT):**
- Check for existing validation utilities
- Use existing validateEmail() if found
- Add email validation only ✓
- Done

### Example 3: The Optimization Tangent
**Request:** "Fix the null pointer exception in getUserById"

**Scope creep (WRONG):**
- Fix null check ✓
- Add caching to improve performance
- Refactor to use repository pattern
- Add comprehensive error handling throughout module

**Correct (RIGHT):**
- Fix null check ✓
- Done

## Self-Check Before Every Change

Before making ANY code change, verify:

1. **Requested?** - Was this specific change requested?
2. **Necessary?** - Is this required to complete the request?
3. **Minimal?** - Is this the smallest change that works?

If any answer is "no", do not make the change.

## The "One More Thing" Test

When tempted to add something extra, ask:
- Would the user notice if I didn't do this?
- Did the user ask for this?
- Is the request incomplete without this?

If the answer to all three is "no", skip it.
