# Plan Document Templates

This document contains three plan templates derived from 28 real plan documents produced during the Acme project (Feb-Mar 2026). Every plan was written by an AI coding agent (Cursor) under human direction, reviewed and iterated before any code was written.

**Which template to use:**

| Situation | Template |
|-----------|----------|
| Something is broken, users report wrong behavior | Template 1: Bug Fix |
| New capability, pipeline, integration, refactor | Template 2: Feature Implementation |
| System redesign, cross-service changes, multi-phase migration | Template 3: Architecture / Design |

> In all templates: placeholders are marked with `[brackets]`. Author guidance is in **(Author note: ...)** callouts.

---
---

## Template 1: Bug Fix Plan

# TICKET-ID: Short Descriptive Title

**Status:** Investigation complete — awaiting review
**Priority:** High / Medium / Low · Sprint NN
**Affected version:** X.Y.Z
**Environment:** Dev / UAT / Production
**Label:** bug / regression

**(Author note: Not all metadata fields are required. Omit what's not applicable.)**

## Bug Summary

**(Author note: 1-3 paragraphs. Describe the symptom, the scenario, and the expected behavior. Use Actual/Expected format.)**

When a user performs [action] as [role] on [page], the system [observed behavior].

- **Actual:** `[what the user sees or what the API returns]`
- **Expected:** `[what should happen instead]`

## Reproduction Flow

1. Login as [role]
2. Navigate to [page]
3. Perform [action]
4. **Expected:** [correct behavior]
5. **Actual:** [observed behavior]

## Root Cause Analysis

**(Author note: The core investigative section. Break into sub-issues if there are multiple root causes across layers. Each issue gets its own subsection with file path, code snippet, and plain-English explanation.)**

### Issue A — [Component/layer where the problem originates]

**File:** `project/path/to/File.cs` (lines NN-NN)

**(Author note: Include the actual problematic code as a fenced code block with language tag.)**

```csharp
// paste the actual problematic code here
```

**Explanation:** Plain-English description of WHY this code is wrong, including the data flow that leads to the symptom.

### Issue B — [Cascading or secondary problem]

**File:** `project/path/to/AnotherFile.ts`

```typescript
// paste the actual problematic code here
```

**Explanation:** How Issue A causes this secondary problem.

### Issue C — [Optional: additional contributing factor]

**(Author note: Add as many Issue subsections as needed. Real plans had 1-5 issues.)**

## Data Flow Diagram

**(Author note: Optional but very effective for cross-layer bugs. Use ASCII art to show how data moves and mark the bug location.)**

```
[Source Table]
    |
    |  field_a = "value"
    |  field_b = 2026-02-18
    |
    v
[Mapper/Service Layer]   <-- BUG: includes field_b in output
    |
    v
[API Response]
    |
    v
[Frontend Component]   <-- visible to user
```

## Affected Files

**(Author note: Table summarizing ALL files that will be touched. Include files verified to need NO changes — this shows thoroughness.)**

| # | File | Change |
|---|------|--------|
| 1 | `path/to/File.cs` | Remove date from title construction |
| 2 | `path/to/File.ts` | No change required (verified) |
| 3 | `path/to/Tests.cs` | New: regression test covering the bug |

## Fix Plan

**(Author note: Numbered steps. Each step covers ONE logical change. Every step includes File, Change description, Before/After code, and Rationale.)**

### Step 1: [Clear title of the change]

**File:** `full/path/to/File.cs`
**Change:** [Description of what to change and why]

**Before:**
```csharp
// the code as it exists today (copy-paste from codebase)
```

**After:**
```csharp
// the corrected code
```

**Rationale:** Why this fixes the issue without introducing side effects.

### Step 2: [Next change]

**File:** `full/path/to/AnotherFile.cs`
**Change:** [Description]

**Before:** / **After:** [same pattern]

### Step 3: Verify no other consumers depend on the old behavior

**(Author note: This verification step appeared in most real plans. It's a table proving the change is safe.)**

| Consumer | How it uses the data | Impact of fix |
|----------|---------------------|---------------|
| `ComponentA.ts` | Parses the field | No impact — handles new format correctly |
| `ComponentB.ts` | Reads a separate field | No impact — unrelated |

## Testing Checklist

**(Author note: Checkbox list. Cover: happy path, error cases, edge cases, regression checks, direct API verification. Be specific — name the user role, the page, and the expected result.)**

- [ ] [Role] performs [action] on [page] — verify [expected result]
- [ ] [Error case: what happens with bad/missing data]
- [ ] [Edge case: boundary condition]
- [ ] [Regression: verify previously working feature still works]
- [ ] API: `GET /api/endpoint` returns [expected response shape]

## Unit Tests

### Test file location
`Project/Tests/Path/FeatureTests.cs`

### Test cases

**(Author note: Table format with specific inputs and expected outputs. Test case #1 should always be the direct regression test for the bug.)**

| # | Scenario | Input | Expected Output |
|---|----------|-------|----------------|
| 1 | Regression test (the bug) | [specific input] | [expected output] |
| 2 | Normal case | [specific input] | [expected output] |
| 3 | Null/empty edge case | [specific input] | [expected output] |

## Risk Assessment

**(Author note: Optional for small changes. Required for anything touching multiple layers or shared data.)**

- **Risk level:** Very low / Low / Medium / High
- **Blast radius:** [which components/flows are affected]
- **Rollback plan:** [how to undo if something goes wrong]

## Files Modified (Final Summary)

**(Author note: Separated by project/layer as a final confirmation checklist.)**

### Backend
| File | Change |
|------|--------|
| `path/File.cs` | [description] |

### Frontend
No changes required. *or* [list of changes]

---
---

## Template 2: Feature Implementation Plan

**(Author note: Use this for new capabilities, pipelines, integrations, or refactors — anything where you're building something rather than fixing something. The key structural differences from the Bug Fix template: "Requirements" replaces "Bug Summary", "Current Behavior Analysis" replaces "Root Cause Analysis", and "Implementation Steps" replaces "Fix Plan".)**

# TICKET-ID: Short Descriptive Title

**Status:** Plan ready — awaiting review
**Priority:** High / Medium / Low · Sprint NN
**Environment:** Dev / UAT / Production
**Label:** feature / enhancement / refactor

## Summary

**(Author note: 1-3 paragraphs. What needs to be built and why. Focus on the business need or technical motivation, not on implementation details.)**

[What the feature/change is, why it's needed, and what success looks like.]

## Requirements

### Functional Requirements

**(Author note: What the system should DO after this work is complete. Numbered for easy reference during review.)**

1. [System should support X]
2. [When Y happens, the system should Z]
3. [Data from A should be available in B]

### Non-Functional Requirements

**(Author note: Optional. Constraints on performance, compatibility, security, etc.)**

1. [Must not break existing API contracts]
2. [Must handle N concurrent requests]
3. [Must be backward-compatible with existing data]

## Current Behavior Analysis

**(Author note: What the system does TODAY in the area you're changing. Include code references. This is the equivalent of "Root Cause Analysis" but for features — understanding the starting point rather than diagnosing a bug.)**

### [Area 1 — e.g., Current data flow]

**File:** `project/path/to/File.cs`

```csharp
// relevant current code
```

**How it works today:** [Explanation of current behavior and its limitations]

### [Area 2 — e.g., Current API contract]

**File:** `project/path/to/Controller.cs`

**How it works today:** [Explanation]

## Design Decisions

**(Author note: Bullet list of decisions made during the planning conversation. Include the alternatives considered and why they were rejected.)**

- **[Decision]:** [chosen approach] over [rejected alternative] because [reasoning]
- **[Decision]:** Do **not** [excluded approach] because [reason]
- **[Decision]:** [approach] — confirmed by [who/when]

## Affected Files

| # | File | Change |
|---|------|--------|
| 1 | `path/to/NewService.cs` | New: service implementing [feature] |
| 2 | `path/to/ExistingHandler.cs` | Add call to new service after [step] |
| 3 | `path/to/Config.json` | Add new configuration section |

## Implementation Steps

**(Author note: Same Before/After pattern as the Bug Fix template, but steps are about building new behavior rather than correcting existing behavior. Steps that introduce NEW code won't have a "Before" block — use "New code" instead.)**

### Step 1: [Clear title of the change]

**File:** `full/path/to/File.cs`
**Change:** [Description of what to build and why]

**New code:**
```csharp
// the new code to add
```

**Rationale:** Why this design, how it fits with existing architecture.

### Step 2: [Wire into existing flow]

**File:** `full/path/to/ExistingHandler.cs`
**Change:** [Description]

**Before:**
```csharp
// existing code before integration
```

**After:**
```csharp
// existing code with new feature wired in
```

### Step 3: [Configuration changes]

**File:** `full/path/to/appsettings.json`
**Change:** [Description]

### Step 4: Verify no regressions in adjacent flows

| Adjacent Flow | How it could be affected | Verified safe? |
|---------------|------------------------|----------------|
| [Flow A] | [Shares data model] | Yes — [reason] |
| [Flow B] | [Uses same endpoint] | Yes — [reason] |

## Testing Checklist

- [ ] [New feature happy path: specific user actions and expected result]
- [ ] [New feature edge case: boundary condition]
- [ ] [Integration: new component communicates correctly with existing components]
- [ ] [Regression: existing feature X still works unchanged]
- [ ] API: `POST /api/new-endpoint` returns [expected response]

## Unit / Integration Tests

### Test file location
`Project/Tests/Path/FeatureTests.cs`

### Test cases

| # | Scenario | Input | Expected Output |
|---|----------|-------|----------------|
| 1 | Happy path | [specific input] | [expected output] |
| 2 | Missing optional field | [specific input] | [expected output] |
| 3 | Duplicate/idempotency | [specific input] | [expected output] |
| 4 | Failure/retry scenario | [specific input] | [expected output] |

## Risk Assessment

- **Risk level:** Very low / Low / Medium / High
- **Blast radius:** [which components/flows are affected]
- **Rollback plan:** [how to undo if something goes wrong]

## Files Modified (Final Summary)

### Backend
| File | Change |
|------|--------|
| `path/NewService.cs` | New: [description] |
| `path/ExistingHandler.cs` | Modified: [description] |

### Frontend
[list of changes] *or* No changes required.

---
---

## Template 3: Architecture / Design Plan

**(Author note: Use this for system redesigns, multi-phase migrations, cross-service changes, and large refactors. The key structural differences: phased implementation, explicit scope boundaries, rollout/rollback plans, and a deliverables checklist.)**

# TICKET-ID: Architecture or Design Title

## Objective

**(Author note: Numbered list of concrete outcomes. What should be true after this work is complete.)**

1. [Primary goal]
2. [Secondary goal]
3. [Constraint or non-goal framed positively]

## Confirmed Decisions

**(Author note: Bullet list of ALL decisions made during the planning conversation. These are the answers to the decision points the agent raised. Include rationale.)**

- [Decision 1 with rationale]
- Do **not** [thing explicitly excluded] because [reason]
- Use [approach A] over [approach B] because [tradeoff reasoning]

## Scope and Non-Goals

### In Scope
- [Bulleted list of what's included]

### Out of Scope
- [Bulleted list of what's explicitly excluded, with brief reason]

## Target End-State Architecture

**(Author note: Numbered description of the desired architecture after all phases are complete. This is the "north star" for reviewing each phase.)**

1. **[Component A]** — [what it does after the change]
2. **[Component B]** — [what it does after the change]
3. **[Removed/disabled component]** — [why it's gone and how it's handled]

## Implementation Plan

### Phase 0 — Baseline and Guardrails

**(Author note: Verification steps BEFORE any code changes. Capture current state, confirm starting conditions.)**

1. [Verify starting state]
2. [Capture baseline metrics/data]
3. [Confirm non-goal note in PR description]

### Phase 1 — [First logical unit of work]

#### 1.1 [Specific change]
**Primary file:** `path/to/file`
**Plan:** [Description of what changes and why]

#### 1.2 [Next specific change]
**Primary file:** `path/to/file`
**Plan:** [Description]

### Phase 2 — [Second logical unit of work]

**(Author note: Same structure as Phase 1. Add as many phases as needed.)**

### Phase N — Tests and Verification

#### N.1 [Test category]
**Cases:**
- [Specific test scenario with expected outcome]
- [Another test scenario]

#### N.2 [Another test category]
**Cases:**
- [Scenario]

## Rollout Plan

**(Author note: Ordered deployment steps.)**

1. [Deploy config change first]
2. [Deploy code changes]
3. [Run verification]
4. [Monitor logs]

## Rollback Plan
- [How to undo each phase if needed]

## Risks and Mitigations

- **Risk:** [What could go wrong]
  - **Mitigation:** [How to prevent or handle it]

## Deliverables Checklist

**(Author note: Final checkbox list. The "definition of done" for this plan.)**

- [ ] [Deliverable 1]
- [ ] [Deliverable 2]
- [ ] All tests passing
- [ ] No runtime errors in logs after deployment

---
---

## Key Formatting Conventions

These conventions were consistent across all 28 real plan documents:

1. **File paths are always absolute** from the project root: `acme-backend/Acme/Acme.DTO/...`
2. **Code blocks use language tags**: ` ```csharp `, ` ```ts `, ` ```sql `
3. **Before/After pattern** for every code change (bug fix and feature integration steps)
4. **"New code" block** for steps that introduce entirely new files/classes (feature plans)
5. **Bold** for file paths and key terms within prose
6. **Tables** for affected files, test cases, and impact analysis
7. **ASCII diagrams** for data flow visualization
8. **Checkbox lists** for testing checklists and deliverables
9. **Line number references** when citing specific code: `File.cs (lines 182-183)`
