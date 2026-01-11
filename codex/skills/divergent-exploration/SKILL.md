---
name: divergent-exploration
description: |
  Explores 3-5 distinct approaches before committing to one. Use when:
  - Design decisions: "design", "architect", "plan", "how should we", "what's the best way"
  - Comparing options: "options", "alternatives", "trade-offs", "pros and cons", "compare approaches"
  - Decision making: "help me decide", "how should I implement", "what approach should I take"
  - Major changes: "refactor", "redesign", "rearchitect", "rebuild", "rewrite", "migrate"
  - Asking for recommendations: "which approach", "what are my options", "how would you solve this"
  - Planning: epic creation, feature design, architecture decisions, complex problem solving
  - About to recommend a single approach for a non-trivial decision

  Enforces generating 3-5 GENUINELY DISTINCT alternatives (not variations). Evaluate each on:
  user impact, complexity, time to value, dependencies, reversibility, risk. Present options, don't decide.
---

# Divergent Exploration

Creative work demands divergent thinking before convergent decisions. This skill enforces structured exploration to prevent premature commitment to the first idea that seems reasonable.

## The Three Phases

### Phase 1: Diverge
Generate 3-5 **genuinely distinct** approaches. Not variations on a theme-fundamentally different solutions.

**Red flag**: If options are "Option A", "Option A but faster", and "Option A with better UI"-divergence has not occurred.

### Phase 2: Evaluate
Assess each option on: User Impact, Technical Complexity, Time to Value, Dependencies, Reversibility, Risk Profile.

### Phase 3: Converge
Present options with clear recommendations. Never decide for users on significant choices.

## The Mindset

The first idea that comes to mind is often:
1. The most conventional (not necessarily best)
2. Anchored on recent experience (may not fit this context)
3. Missing creative solutions that require more thought

By forcing generation of genuine alternatives, the outcome is either:
- **Discovering a better approach** that would have been missed
- **Validating the initial instinct** with confidence it survived comparison

Both outcomes improve decision quality.

## Self-Check Questions to Generate Distinct Options

1. What if we solved the opposite problem instead?
2. What would a competitor with unlimited resources do?
3. What would a scrappy startup with no resources do?
4. What if we eliminated this requirement entirely?
5. What existing system could we extend vs. building new?
6. What would users build if we gave them primitives instead?
7. What's the "do nothing" option and its consequences?
8. What if the constraint we're assuming doesn't actually exist?

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

## Anti-Patterns to Avoid

**Premature convergence**: "The obvious approach is..." - Stop. Generate alternatives first.

**False diversity**: Presenting the same idea with different labels. Each option should have a different core mechanism.

**Analysis paralysis**: Exploration has a time limit. Set one. Three solid options beat seven half-baked ones.

**Hidden preference**: If you already know what you want to recommend, be explicit about it-but still generate real alternatives.

**Skipping for "simple" decisions**: Many "simple" decisions have non-obvious alternatives that produce better outcomes.

See `references/exploration-patterns.md` for additional patterns and integration guidance.

---

## How to Use This Skill in Codex

Include this skill's content in your Codex prompt when:
- Making architecture or design decisions
- Planning features or epics
- Comparing implementation approaches
- Refactoring or redesigning systems

Copy the evaluation dimensions and output format to structure divergent exploration sessions.
