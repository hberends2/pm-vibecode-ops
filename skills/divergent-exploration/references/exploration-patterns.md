# Divergent Exploration Patterns

Detailed patterns and templates for structured exploration.

## Self-Check Questions to Generate Distinct Options

1. What if we solved the opposite problem instead?
2. What would a competitor with unlimited resources do?
3. What would a scrappy startup with no resources do?
4. What if we eliminated this requirement entirely?
5. What existing system could we extend vs. building new?
6. What would users build if we gave them primitives instead?
7. What's the "do nothing" option and its consequences?
8. What if the constraint we're assuming doesn't actually exist?

**Red flag**: If your options are "Option A", "Option A but faster", and "Option A with better UI"—you haven't diverged. Those are variations, not alternatives.

## Evaluation Dimensions

For each option, assess:

| Dimension | Questions to Answer |
|-----------|---------------------|
| **User Impact** | Who benefits? Who loses? How much friction? |
| **Technical Complexity** | New systems? Integration points? Maintenance burden? |
| **Time to Value** | When do users see benefit? Incremental delivery possible? |
| **Dependencies** | What must exist first? External teams? Third parties? |
| **Reversibility** | Can we change course later? Lock-in risks? |
| **Risk Profile** | What could go wrong? Known unknowns? |

## Output Format Template

When presenting divergent exploration results, use this structure:

```markdown
## Exploration: [Problem Statement]

### The Core Question
[One sentence framing the decision to be made]

### Option 1: [Descriptive Name]
**Approach**: [2-3 sentence summary]

- **Pros**: [Bullet list]
- **Cons**: [Bullet list]
- **Effort**: [T-shirt size + brief justification]
- **Dependencies**: [What's needed first]
- **Best if**: [Conditions that make this the right choice]

### Option 2: [Descriptive Name]
[Same structure]

### Option 3: [Descriptive Name]
[Same structure]

---

### Recommendation
**If [condition], choose Option X because [reasoning].**
**If [different condition], choose Option Y because [reasoning].**

### Questions Before Deciding
1. [Clarifying question that would change the recommendation]
2. [Constraint question that might eliminate options]
3. [Priority question that would reorder options]
```

## Calibrating Depth

| Decision Type | Options Needed | Evaluation Depth |
|---------------|----------------|------------------|
| Epic scope | 3-5 | Full template |
| Architecture pattern | 3-4 | Full template |
| Feature approach | 3 | Abbreviated (pros/cons/effort) |
| Implementation detail | 2-3 | Quick comparison |
| Bug fix strategy | 2 | Inline analysis |

## Anti-Patterns to Avoid

**Premature convergence**: "The obvious approach is..." — Stop. Generate alternatives first.

**False diversity**: Presenting the same idea with different labels. Each option should have a different core mechanism.

**Analysis paralysis**: Exploration has a time limit. Set one. Three solid options beat seven half-baked ones.

**Hidden preference**: If you already know what you want to recommend, be explicit about it—but still generate real alternatives.

**Skipping for "simple" decisions**: Many "simple" decisions have non-obvious alternatives that produce better outcomes.

## Integration with Workflow

This skill activates during:

- `/epic-planning` — Before defining epic scope
- `/planning` — Before technical decomposition
- `/adaptation` — When multiple implementation approaches exist
- `/discovery` — When analyzing patterns with multiple valid interpretations

The output feeds into decision documentation, ensuring future readers understand why alternatives were rejected.
