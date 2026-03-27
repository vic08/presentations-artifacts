# Plan First, Code Second — Knowledge Artifacts

Companion materials for the "Plan First, Code Second" LiveSpeech presentation (March 17, 2026). These artifacts are derived from real Cursor agent conversations and plan documents produced during the Acme project (February–March 2026).

## Documents

### Templates & Methodology

| # | Document | What It Covers |
|---|----------|---------------|
| 01 | [Plan Document Template](01-plan-document-template.md) | Two reusable templates: Bug Fix plan and Architecture/Design plan. Derived from 28 real plan documents. Includes section structure, formatting conventions, and examples. |
| 02 | [Initial Prompt Template](02-initial-prompt-template.md) | Four prompt templates for starting a plan-first session: Bug Fix, Feature/Rewrite, Consolidation, and Test Plan. Each includes the real prompt that was used. |
| 03 | [Follow-Up Review Structure](03-follow-up-review-structure.md) | The 6-phase conversation pattern (Investigate → Plan → Review → Decide → Implement → Verify) with exact phrases, anti-patterns, and timing breakdown. |

### Conversation Transcripts

| # | Document | Conversation | Turns | What It Demonstrates |
|---|----------|-------------|-------|---------------------|
| 04 | [Consumer Rewrite](04-transcript-consumer-rewrite.md) | ACME-6848 single consumer rewrite | 23 | Full lifecycle: context loading, plan creation, multi-round review, 8 decision points resolved, catching a fundamental misunderstanding before code, post-implementation quality review |
| 05 | [ID Consolidation](05-transcript-id-consolidation.md) | ResearchSite ID investigation | 47 | Investigation-to-plan: vague problem report → deep understanding through 3 rounds of "I don't understand" → user proposes own architecture → live code review during implementation |
| 06 | [FK Error Fix](06-transcript-fk-error-fix.md) | RabbitMQ FK violation | 34 | Debug-to-plan: error reproduction, test plan creation, 1.2MB of log analysis across 6 rounds, fix plan, post-fix test coverage enforcement |

## Source Data

- **Plan documents:** `/Users/bunhackr/projects/acme/plans/` (28 markdown files)
- **Cursor transcripts:** `~/.cursor/projects/Users-bunhackr-projects-acme/agent-transcripts/` (18 conversations)
- **Presentation extracts:** `../cursor_*.md` (3 pre-extracted transcripts in the presentation folder)

## The Approach in One Paragraph

Every task — whether a bug fix, feature, or architectural change — starts with a planning phase where the AI agent investigates code, asks clarifying questions, and produces a markdown plan document. The human reviews the plan with line-level precision, catches misunderstandings, answers decision points, and only gives the explicit "implement" signal when all questions are resolved. Implementation follows the plan as a checklist, with the human reviewing code quality, enforcing test coverage, and verifying completion against the plan document. The planning phase typically consumes 30–60% of total session time but eliminates most rework.
