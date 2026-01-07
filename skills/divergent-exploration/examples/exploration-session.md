# Divergent Exploration Session Example

A complete example demonstrating the three-phase exploration process for a real architectural decision.

---

## Scenario: Choosing a State Management Approach

**Context**: A React application needs to manage complex form state across multiple steps, with validation, auto-save, and the ability to resume incomplete submissions.

---

## Phase 1: Diverge

Generate genuinely distinct approaches (not variations on a theme).

### Option 1: URL-Based State (Query Parameters)

**Approach**: Store all form state in URL query parameters. Each field maps to a parameter. Navigation between steps changes the URL path while preserving state in the query string.

```typescript
// URL: /apply/step-2?name=John&email=john@example.com&plan=premium
const formState = useSearchParams();
const updateField = (field, value) => {
  setSearchParams(prev => ({ ...prev, [field]: value }));
};
```

### Option 2: Server-First with Optimistic UI

**Approach**: Every field change immediately persists to the server. Local state is a cache of server state. The server is the source of truth, with optimistic updates for responsiveness.

```typescript
// Each change triggers API call
const { mutate } = useMutation(updateApplication);
const handleChange = (field, value) => {
  mutate({ applicationId, [field]: value });
};
```

### Option 3: Local-First with Background Sync

**Approach**: Store state in IndexedDB/localStorage. Sync to server periodically or on specific triggers (step completion, blur events). Works offline. Server reconciles conflicts.

```typescript
// Persist locally, sync in background
const [form, setForm] = useLocalStorage('application-draft');
useSyncToServer(form, { interval: 30000, onStepComplete: true });
```

### Option 4: Finite State Machine

**Approach**: Model the entire flow as a state machine using XState. Each step is a state. Transitions validate before advancing. State machine handles edge cases (back navigation, validation, recovery).

```typescript
const applicationMachine = createMachine({
  initial: 'personalInfo',
  states: {
    personalInfo: {
      on: { NEXT: { target: 'planSelection', cond: 'isPersonalInfoValid' } }
    },
    planSelection: { /* ... */ }
  }
});
```

---

## Phase 2: Evaluate

### Option 1: URL-Based State

| Dimension | Assessment |
|-----------|------------|
| **User Impact** | Shareable links, browser back/forward works naturally. Limited to string data, complex state is awkward. |
| **Technical Complexity** | Low. Built-in browser feature. Serialization can get messy for nested objects. |
| **Time to Value** | Immediate. No libraries needed. |
| **Dependencies** | None. |
| **Reversibility** | High. Easy to migrate away. |
| **Risk Profile** | URL length limits (~2000 chars). Sensitive data visible in URL. |

**Best if**: Simple forms, sharing/bookmarking is important, mostly string fields.

### Option 2: Server-First with Optimistic UI

| Dimension | Assessment |
|-----------|------------|
| **User Impact** | Always saved, never lose work. Requires constant connectivity. Slight delay perception. |
| **Technical Complexity** | Medium. Need robust error handling, conflict resolution, optimistic update rollback. |
| **Time to Value** | Medium. Requires API endpoints for partial saves. |
| **Dependencies** | API team must support partial updates. Network reliability. |
| **Reversibility** | Medium. API contract becomes a dependency. |
| **Risk Profile** | Network failures interrupt flow. API rate limits. Complex offline handling. |

**Best if**: Compliance requires server-side persistence, multi-device support needed, strong network guaranteed.

### Option 3: Local-First with Background Sync

| Dimension | Assessment |
|-----------|------------|
| **User Impact** | Instant responsiveness, works offline, resume from any device (after sync). Potential sync conflicts. |
| **Technical Complexity** | High. IndexedDB is complex. Sync logic, conflict resolution, storage quotas. |
| **Time to Value** | Slow. Significant infrastructure for sync and conflict handling. |
| **Dependencies** | Background sync APIs, service worker knowledge. |
| **Reversibility** | Low. Deep architectural commitment. |
| **Risk Profile** | Data loss if sync fails. Storage quota issues. Complex debugging. |

**Best if**: Offline support is critical, very long forms (save progress), poor network conditions expected.

### Option 4: Finite State Machine

| Dimension | Assessment |
|-----------|------------|
| **User Impact** | Predictable flow, clear validation, no invalid states possible. Learning curve for developers. |
| **Technical Complexity** | Medium-high. XState learning curve. Powerful but verbose for simple cases. |
| **Time to Value** | Medium. Upfront design time, but prevents bugs later. |
| **Dependencies** | XState library. Team must learn state machine concepts. |
| **Reversibility** | Medium. Logic is centralized, but migration requires rewrite. |
| **Risk Profile** | Over-engineering for simple forms. Steep debugging without XState devtools. |

**Best if**: Complex flows with many edge cases, validation-heavy, need to prevent invalid states.

---

## Phase 3: Converge

### Comparison Matrix

| Factor | URL | Server-First | Local-First | State Machine |
|--------|-----|--------------|-------------|---------------|
| Offline support | No | No | Yes | Partial |
| Implementation time | 1 day | 1 week | 3 weeks | 1 week |
| Team familiarity | High | High | Low | Medium |
| Complexity ceiling | Low | Medium | High | High |
| Data sensitivity fit | Poor | Good | Medium | Good |

### Recommendation

**If the form is simple (under 10 fields, single page)**: Use **Option 1 (URL-Based)**. Zero dependencies, works immediately, supports sharing and browser navigation.

**If compliance requires audit trails or multi-device is needed**: Use **Option 2 (Server-First)**. Accept the network dependency in exchange for guaranteed persistence and simpler architecture than local-first.

**If the form is complex with conditional logic and validation interdependencies**: Use **Option 4 (State Machine)**. The upfront investment prevents entire categories of bugs. The state visualization aids debugging.

**If offline support is a hard requirement**: Only then consider **Option 3 (Local-First)**. The complexity is justified only when users genuinely cannot rely on connectivity.

### Questions Before Deciding

1. **How long is the form?** (Under 10 fields vs. 50+ fields changes the calculus significantly)
2. **What's the network environment?** (Corporate offices vs. field workers on mobile)
3. **Does compliance require server-side state?** (HIPAA, financial regulations)
4. **How complex is the validation logic?** (Simple required fields vs. cross-field dependencies)

---

## Why This Matters

Without divergent exploration, the team might have defaulted to "React Context + useState" — the most common approach — which wasn't even one of the genuinely distinct options. Each option above solves the problem through a fundamentally different mechanism:

- URL uses the browser as state container
- Server-first uses the database as source of truth
- Local-first uses the client as source of truth with eventual sync
- State machine uses a formal model to prevent invalid states

By forcing distinct alternatives, we discovered that the choice depends on factors (offline needs, compliance, complexity) that would otherwise remain implicit assumptions.
