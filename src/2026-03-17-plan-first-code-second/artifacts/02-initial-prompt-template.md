# Initial Prompt Template: How to Request a Plan from an AI Coding Agent

This document shows how to compose the first message that starts a "plan first, code second" session. The templates are derived from real conversations with Cursor agent on the Acme project.

---

## The Core Principle

Your opening prompt should accomplish three things:
1. **Provide context** — what you're working on, what's broken, or what needs to be built
2. **Request a plan, not code** — explicitly state that no code should be written yet
3. **Invite questions** — tell the agent to surface unknowns before committing to a plan

---

## Template A: Bug Fix / Investigation

Use this when you have a bug report or unexpected behavior to investigate.

```
I have [bug/problem description]. Here is what I know:

[Paste relevant context: error logs, JIRA ticket, stack trace, reproduction steps]

@[reference existing analysis docs, related plans, or code files]

Can you please investigate [specific areas to look at]:
- [Area 1: e.g., check the database migrations]
- [Area 2: e.g., check the RabbitMQ consumer logic]
- [Area 3: e.g., verify the API endpoint behavior]

If you need additional verification steps before you do the final analysis,
please let me know.

The final version of analysis please put into .md file in @plans/ folder.
```

**Real example from the project:**

> My colleague claims that the data in Organization database ResearchSite table is broken. He says that the ResearchSite data is populated from the aact database ctgov schema and the id of the research site is taken from there and should not be changed.
>
> Can you please investigate, what exactly can ruin the data in organization database?? please check the database migrations in orglist project, if they can break anything. and please check the rabbitmq consumer (which actually update the entities in org list database). This logic may be dropping the entities and recreating them with new ids...
>
> If you need additional verification steps before you do the final analysis, please let me know. The final version of analysis please put into .md file in @plans/ folder.

---

## Template B: New Feature / Rewrite with Prior Context

Use this when you want to implement something new, especially when there's existing work or a previous attempt to learn from.

```
I had [this problem/feature], reported and analyzed [previously].
@[reference prior analysis docs, branches, or plans]

[Describe what the previous approach was and why it needs to change]

Right now, I would like you to [describe the new approach at a high level].

Please analyze all the context that I gave to you and create a step-by-step
implementation plan. If you have any open questions or decision points,
please ask me before writing the final version of the plan.

Don't change any code. Just give me the plan for review or a list of questions.
```

**Real example from the project:**

> I had this problem, reported and analyzed several days ago @plans/orglist-researchsite-id-dev-vs-uat-analysis.md. We have made a solution with you in a different chat. That solution is stored in a different branch bugfix/ACME-6848-dev.
>
> Please analyze the commits in that 6848-dev branch, and please understand what was that proposed solution. basically the main idea there was keeping two distinct rabbitmq consumers and have those dependent IDs be generated in a deterministic way independently so that two consumers could keep working in parallel independently.
>
> Right now, I would like you to get rid of two independent consumers and keep only one consumer in the Acme.API project, and remove the consumer from organization list project.
>
> Please analyze all the contacts that I gave to you and create a step-by-step implementation plan for the thing that I asked you. If you have any open questions or decision points, please ask me before writing the final version of the plan. Don't change any code. Just give me the plan for review or a list of questions.

---

## Template C: Consolidation / Architectural Change

Use this when you need a comprehensive plan to fix a systemic problem.

```
[Describe the systemic problem and constraints]

I need to eliminate all these problems:
- [Constraint 1: e.g., there must never be any duplicates]
- [Constraint 2: e.g., the IDs must not differ between X and Y]
- [Constraint 3: e.g., is it possible to do X inside Y?]

Please give me a comprehensive consolidation plan.
```

**Real example from the project:**

> Alright, actually I need to eliminate all these problems, there must never be any duplicates, the ids must not differ between the legacy sql ingest and new rabbitmq based ingest. Is it possible to do that deterministic id inside legacy sql ingest? Please give me a comprehensive consolidation plan.

---

## Template D: Test Plan (After Code Changes, Before Testing)

Use this when code exists but you need a structured testing approach.

```
Now we have updated code and some automation scripts.

Can you please write a proper step by step plan of everything that needs
to be tested. Put it into .md file in @plans/ folder, don't change any code.
```

**Real example from the project:**

> Now we have updated code and some automation scripts. Can you please write a proper step by step plan of everything that needs to be tested. put it into .md file in @plans/ folder, don't change any code.

---

## Critical Phrases That Enforce the "Plan First" Discipline

### When you need these phrases

These phrases are essential when you work with a **pure agent mode** — a vanilla AI chat or agent session where the agent has no built-in concept of "planning vs. coding" phases. In pure agent mode, the agent will default to writing code as soon as it understands the problem. You need explicit verbal gates to prevent that.

| Phrase | Purpose |
|--------|---------|
| `"Don't change any code."` | Hard stop on implementation |
| `"Just give me the plan for review or a list of questions."` | Forces plan output |
| `"If you have any open questions or decision points, please ask me before writing the final version."` | Ensures unknowns surface early |
| `"Put it into .md file in @plans/ folder."` | Creates a reviewable artifact |
| `"Please only confirm, don't start writing anything."` | Gate before plan creation |
| `"Just answer, don't do code yet."` | Micro-gate during Q&A phase |

### Reducing repetitiveness with modern agent features

Modern AI coding agents have built-in mechanisms that understand planning intent, so you don't need to repeat "don't change any code" in every message:

- **Cursor: Plan mode.** When Cursor detects that you're asking for a design or implementation plan (or when you manually switch), it enters a dedicated planning mode where the agent only reads, analyzes, and writes plan documents — tool calls that modify code are suppressed. In the real transcripts, Cursor would announce: *"I'm switching to planning mode first since you asked for a design/implementation plan with possible decision points."* Once in plan mode, you can discuss freely without guarding against premature code changes.

- **Claude Code: Plan mode (`/plan`).** Claude Code has an explicit plan mode toggled with `/plan` or by asking Claude to "think about this first." In plan mode, Claude produces a structured plan and asks for approval before taking any action. The agent understands that it should research and propose, not execute.

- **System prompts / custom instructions.** Both Cursor (`.cursorrules`) and Claude Code (`CLAUDE.md`) support project-level instructions. You can encode the planning discipline once — e.g., *"For any task involving more than 3 files, always produce a plan document in `plans/` and wait for approval before implementation"* — and the agent follows it for every conversation in that project.

- **Agentic workflows with checkpoints.** Tools like Cursor's multi-step agent and Claude Code's task system allow you to define explicit checkpoints: "after investigation, pause for review." The agent treats these as natural stopping points rather than requiring verbal gates.

**Bottom line:** The phrases in the table above are the universal fallback — they work with any AI agent, any mode, any tool. But if your agent supports planning mode or custom instructions, use those to reduce the conversational overhead. The discipline stays the same; the enforcement mechanism becomes the tool rather than your words.

---

## What Makes a Good Opening Prompt

1. **Reference existing artifacts** — point the agent to prior analysis, related branches, existing plans. The more context, the better the plan.

2. **Describe the "what" and "why", not the "how"** — let the agent propose the implementation approach. You review it.

3. **Be explicit about scope** — what's in, what's out. "We're NOT doing X in this task."

4. **Name the output format** — "put it into .md file in @plans/ folder" ensures you get a reviewable document, not just chat text.

5. **Invite questions over assumptions** — "ask me before writing the final version" is better than getting a plan based on wrong assumptions.
