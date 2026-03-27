# Follow-Up Review Messages: The Conversation Pattern

This document describes the structured conversation flow that happens between the initial prompt and the "go implement" signal. These patterns are derived from real Cursor agent conversations on the Acme project.

---

## The 6-Phase Conversation Pattern

```
┌─────────────────────────────────────────────────────┐
│  Phase 1: INVESTIGATE                               │
│  User provides context → Agent reads code, asks Qs  │
├─────────────────────────────────────────────────────┤
│  Phase 2: PLAN                                      │
│  User requests plan doc → Agent writes to @plans/   │
├─────────────────────────────────────────────────────┤
│  Phase 3: REVIEW                                    │
│  User reviews with line refs → Catches errors       │
├─────────────────────────────────────────────────────┤
│  Phase 4: DECISION POINTS                           │
│  Agent surfaces open Qs → User answers each one     │
├─────────────────────────────────────────────────────┤
│  Phase 5: IMPLEMENT                                 │
│  User gives explicit go → Agent codes to the plan   │
├─────────────────────────────────────────────────────┤
│  Phase 6: VERIFY                                    │
│  User reviews code → Runs tests → Checks vs plan    │
└─────────────────────────────────────────────────────┘
```

---

## Phase 1: Investigation — Answering Agent Questions

After the initial prompt, the agent typically asks clarifying questions. Your job is to answer them precisely and challenge anything that doesn't make sense.

**Pattern: Answer + Challenge**

```
- "Should the API consumer now update both databases...":
  It's the API consumer's job now. The legacy bulk pipeline is
  gonna be removed at some point.

- "Do you want to fully retire the OrgListIngest pipeline...":
  Switch it off via config and create a .md doc describing the
  problem with it.

- "You mentioned NTT table...":
  This must be a macOS dictation error. I was referring to entities
  table in ctgov database.
```

**Pattern: Push Back When You Don't Understand**

> "I don't understand this, please explain"

> "I don't get it. In fact the incoming rabbitmq message is not only for updates but for creates as well. Why would we expect a particular nct id to be present in some of our tables in case of 'create'?"

> "I still don't understand why this deterministic uuid approach is required? If the facility id comes from ctgov anyway, then why different consumers don't have access to that same id?"

**Key principle:** Don't accept explanations you don't understand. The plan will be wrong if your understanding is wrong.

---

## Phase 2: Plan Creation — The Readiness Gate

Before the agent writes the plan, confirm they have enough information:

```
Are you ready to compose the plan document right now?
Please only confirm, don't start writing anything.
```

Then, only after confirmation:

```
All right, then please write down the markdown document
with the plan. Put it into @plans/ folder as usual.
```

**Why two steps?** The confirmation gate ensures the agent doesn't start writing a plan based on partial understanding. If they say "I still have questions about X", you answer those first.

---

## Phase 3: Review — Line-Level Feedback on the Plan

Review the plan document with specific line references. This is where you catch misunderstandings before they become code.

**Pattern: Reference specific lines**

> `@plans/ACME-6848-single-consumer-legacy-id-rewrite-plan.md:124-126` — is this actually the right order?? I thought studies must be written first, because entities table is populated there and then the id of entity must match created sponsor and research site ids.

**Pattern: Catch fundamental misunderstandings**

> Dude, sorry this is wrong. We may have misunderstood each other. We need a way to preserve the ID on re ingest. Please reintroduce the section and let me know if there is an open question still.

**Pattern: Request structural changes to the plan itself**

> Please rewrite your document plan in a form of actionable step by step implementation plan, where it's visible which files are to be changed and what is the goal of that exact change.

**Pattern: Add missing sections**

> One more thing: please add the integration tests step with outline of tests to be written to cover new changes. Add this section to document plan and give me for review.

---

## Phase 4: Decision Points — The Pre-Implementation Gate

This is the most critical phase. Before implementation, explicitly ask:

```
Ok, any open questions / decision points before you
are able to implement this plan?
```

The agent should surface remaining unknowns. Answer each one:

```
- "Feature flag strategy...": No need for feature flag.

- "OrgList write transaction model...": Ideally, I would like
  to have both writes as a single transaction. If something fails
  during this transaction then everything must be rolled back.

- "ID reuse match keys...": I'm not sure to be honest whether
  we should do this. I don't want to increase risks.
```

Then request a final consolidated review:

```
Please update the plan and give me a final review
before you start implementing.
```

And one more confirmation:

```
Please reflect this in the planning document. And confirm
again that you have all you need for implementation.
Just answer, don't do code yet.
```

---

## Phase 5: Implementation — The Green Light

Only after the plan is reviewed and all decisions are made:

```
Please implement everything according to the plan.
```

Or, for phased implementation:

```
Ok, please go ahead to next phase please.
```

Or, the simplest form:

```
Yes, just execute the core plan.
```

---

## Phase 6: Verification — Post-Implementation Review

### Code Quality Check

```
Please doublecheck if all your code changes follow the SOLID
principles, and in general that the code is maintainable and clean.
```

### Specific Code Challenges

> `@acme-backend/.../CtgovIntegrationTestBase.cs:162-163` — This method looks weird. Why would you want to check the schema of the database? And especially with just plain SQL? If anything changes in future, it's not gonna be highlighted automatically with code like this.

### Cost-Benefit Questions

```
What is the best ratio of effort to stability of solution?
Which fix would you recommend?
```

### Scope Control

> We don't have exact requirements for new UI, we can't add new UI proactively. If there is error in data, explicit failure is fine.

### Plan Completion Check

```
Have you covered all the points in your plan? Please confirm.
```

```
Have you verified that no code paths rely on 'legacy numeric
id format' for code path decisions? Please confirm this first,
before moving on.
```

### Test Coverage Enforcement

```
First of all please can you update the integration test for the
consumer that you have just fixed, because the current integration
test did not catch the issue! Please cover it with a proper test.
```

### Cleanup Before Commit

```
Alright, now please cleanup your changes and prepare them
for being committed.
```

```
You need to clean up excessive debug logging as well.
```

---

## Anti-Patterns to Avoid

| Anti-Pattern | What to Do Instead |
|---|---|
| Accepting an explanation you don't understand | Push back: "I don't get it, please explain" |
| Letting the agent add unrequested features | "We can't add new UI proactively" |
| Skipping the decision points gate | Always ask: "Any open questions before implementation?" |
| Reviewing code without referencing the plan | "Have you covered all the points in your plan?" |
| Letting debug code ship | "Clean up excessive debug logging" |
| Accepting hard-coded values | "Can you somehow pull it from a single source?" |

---

## The Rhythm in Practice

A typical session flows like this (times are approximate):

```
[5-10 min]  Opening prompt with context + "don't change any code"
[5-15 min]  Agent asks questions, you answer and challenge
[2 min]     "Are you ready? Please only confirm."
[5-10 min]  Agent writes plan document
[10-20 min] You review, catch errors, request changes
[5 min]     "Any open questions / decision points?"
[5-10 min]  Agent lists unknowns, you answer each one
[2 min]     "Please update plan and give me final review"
[5 min]     Final review pass
[1 min]     "Please implement everything according to the plan"
[15-30 min] Implementation (you may review code incrementally)
[10-15 min] Run tests, verify against plan, cleanup
```

Total: 60-120 minutes for a well-scoped task. The planning phase takes 30-60% of the total time — and that's the point. The implementation phase has far fewer surprises.
