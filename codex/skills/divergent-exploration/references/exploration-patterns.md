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

**Red flag**: If your options are "Option A", "Option A but faster", and "Option A with better UI"-you haven't diverged. Those are variations, not alternatives.

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

**Premature convergence**: "The obvious approach is..." - Stop. Generate alternatives first.

**False diversity**: Presenting the same idea with different labels. Each option should have a different core mechanism.

**Analysis paralysis**: Exploration has a time limit. Set one. Three solid options beat seven half-baked ones.

**Hidden preference**: If you already know what you want to recommend, be explicit about it-but still generate real alternatives.

**Skipping for "simple" decisions**: Many "simple" decisions have non-obvious alternatives that produce better outcomes.

## Integration with Workflow

This skill activates during:

- **Epic Planning** - Before defining epic scope
- **Planning** - Before technical decomposition
- **Adaptation** - When multiple implementation approaches exist
- **Discovery** - When analyzing patterns with multiple valid interpretations

The output feeds into decision documentation, ensuring future readers understand why alternatives were rejected.

## Example: Real Exploration Session

**Problem**: Users need to export their data in multiple formats.

### Option 1: Client-Side Export
**Approach**: Generate exports in the browser using JavaScript libraries.

- **Pros**: No server load, instant feedback, works offline
- **Cons**: Large datasets crash browser, format support limited
- **Effort**: S (2-3 days)
- **Dependencies**: None
- **Best if**: Data is small (<10MB), limited formats needed

### Option 2: Server-Side Queue
**Approach**: Submit export request, process in background, email link when done.

- **Pros**: Handles any size, all formats, scalable
- **Cons**: Delay before user gets file, requires email setup
- **Effort**: L (2 weeks)
- **Dependencies**: Background job system, email service
- **Best if**: Large datasets common, many format requirements

### Option 3: Streaming Download
**Approach**: Server streams file directly as download, generated on-the-fly.

- **Pros**: Immediate start, handles large files, simpler than queue
- **Cons**: Long connection required, can timeout, server load during download
- **Effort**: M (1 week)
- **Dependencies**: None
- **Best if**: Medium datasets, immediate feedback important

---

### Recommendation
**If typical export < 10MB**: Option 1 (client-side) - fastest to implement, good UX
**If typical export > 100MB**: Option 2 (queue) - necessary for reliability
**If 10-100MB typical**: Option 3 (streaming) - balance of UX and capability

### Questions Before Deciding
1. What's the 95th percentile export size?
2. Do users need exports immediately or can they wait for email?
3. Do we already have a background job system?
