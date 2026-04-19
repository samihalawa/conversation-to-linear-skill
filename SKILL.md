---
name: conversation-to-linear
description: >
  Extract actionable items from the current conversation or an explicitly requested recent-history window and sync only genuinely useful,
  non-duplicate items into Linear. Best for Oulang work when the user says "sync to linear", "add to linear", "scan recent conversations",
  or wants opportunities captured after analysis.
---

# Conversation to Linear

## Purpose

Turn real conversation output into clean Linear tracking without polluting the backlog.

This skill is for:

- current-thread issue capture
- explicit recent-history scans such as "since last run", "last 24h", or "recent conversations"
- Oulang opportunity capture after analysis, especially monetization, conversion, trust, and revenue-leak themes

This skill is not for:

- dumping every idea into Linear
- copying raw analysis into tickets
- inventing speculative roadmap items

## Hard Outcome

By the end of the run, every selected item must be one of:

- `created`
- `extended`
- `already_tracked`
- `rejected_for_fit`

If nothing qualifies, say exactly:

> "All items from this conversation are already tracked in Linear — nothing new to add."

## Fixed Targets

| Project | Use when |
| --- | --- |
| `samihalawa/2026-MANUS-oulang` | code fixes, implementation tasks, technical debt, UX fixes, instrumentation, operational debugging |
| `Oulang Ideas & Inspirations` | business ideas, monetization opportunities, product opportunities, strategic insights |

Team is always `Pime`.

These project names are not advisory. They are the required routing contract.

- `samihalawa/2026-MANUS-oulang` is for real code work and execution-backed product/debugging tasks.
- `Oulang Ideas & Inspirations` is for opportunities, ideas, strategy, monetization concepts, and product directions that are not yet implementation tasks.
- If the best matching existing issue is in the wrong project, move that issue to the correct project instead of duplicating it.
- Never leave a clear idea/opportunity in `samihalawa/2026-MANUS-oulang` just because it was created there historically.
- Never leave a real implementation/debugging issue in `Oulang Ideas & Inspirations` just because it was brainstormed there first.

## Actual Tooling Rules

Use only the Linear tools that actually exist in the environment.

Expected core tools:

- `list_issues`
- `save_issue`
- `fetch`
- `save_comment` if available
- `research` when you need cross-issue or project-aware Linear reasoning
- `list_issue_statuses` when status names are unclear in the current workspace

Do not assume tools like `get_issue` or `list_projects` exist unless you verified them in the current environment.

Use the full Linear surface when available:

- fetch full issue content before deciding a duplicate is “close enough”
- read issue descriptions, comments, labels, statuses, assignees, project, and relations when they matter
- prefer `research` over brittle multi-step guessing when the question spans projects, issue movement, or historical cleanup intent

Tool discovery is not enough by itself.

- If `research`, `fetch`, `save_comment`, or any other advanced tool errors on first real use, immediately fall back to the verified core tools instead of continuing as if the tool exists.
- Do not write a workflow that depends on `research` unless the current environment actually executes it successfully.
- Record the fallback in the run notes so future passes do not repeat the same broken assumption.

If comments are unsupported, extend existing issues by updating the issue description with `save_issue(id=...)` only when the addition is clearly valuable and non-destructive.

## Two Modes

### 1. Current-thread mode

Default mode.

Use when the user asks to sync this conversation or add items from the current chat.

Sources:

- current conversation only

### 2. Recent-history mode

Use only when the user explicitly asks to scan beyond the current thread with wording like:

- "scan recent conversations"
- "since last run"
- "last 24h"
- "from recent logs/notes"

Sources:

- current conversation
- recent local conversation history
- optionally recent Notion or Google Drive notes if the user explicitly asked for those source families

When using recent-history mode, follow the source-discipline patterns from `$conversation-history-recovery-skill`:

- build a source ledger first
- state what sources were fully read vs sampled
- prefer recent, relevant sources over broad archive trawls
- do not claim full coverage if you sampled

The purpose here is not a full forensic report. It is targeted issue extraction from recent evidence.

If the user explicitly asks for `code issues`, `regressions`, or `not opportunities`, enforce that filter strictly even in recent-history mode:

- exclude product ideas
- exclude monetization opportunities
- exclude speculative growth items
- extract only concrete code defects, regressions, broken flows, or tracker hygiene defects directly tied to code work

## Idempotency Contract

Safe to run repeatedly.

1. Fetch existing issues first.
2. Compare candidate items against the live backlog.
3. Only create items that are genuinely new.
4. Prefer extending an existing issue over creating a sibling duplicate.
5. If an issue is materially correct but lives in the wrong project, re-home it instead of creating a new one.

## Selection Standard

A candidate item must pass all required gates.

### Gate 1: Explicitness

The item must be explicitly stated in the conversation or directly implied by a concrete analytical conclusion in the same conversation.

Allowed:

- "add this to Linear"
- "this is a revenue leak"
- "this should become an issue"
- a clearly stated opportunity after analysis

Not allowed:

- background context
- generic observations with no action
- examples or hypotheticals
- "could maybe someday" speculation

If the user explicitly limits the run to code issues only, a candidate also fails Gate 1 when it is merely an opportunity framed as a bug.

### Gate 2: Stack-fit

The item must make sense for the current product and codebase.

Prefer:

- improvements that reuse existing routes, features, or data
- low-schema or no-schema opportunities
- fixes or ideas that fit the current marketplace, jobs, talent, chat, recharge, premium, and admin surfaces

Reject or down-rank:

- ideas that need a replatform
- ideas that assume a totally different app architecture
- ideas that require large speculative DB redesigns
- ideas that mostly copy competitor implementation details rather than product logic

### Gate 3: Monetization relevance

For Oulang-focused runs, prefer only items that are directly or indirectly monetization-relevant:

- direct revenue opportunities
- conversion improvements
- trust/quality fixes that protect monetization
- broken commercial flows
- ugly or confusing user-facing defects that make revenue surfaces look weak

Do not add edge enhancements unless the user explicitly asks for a broad backlog sweep.

### Gate 4: Non-duplication

If the same core action or opportunity already exists in Linear, do not create a duplicate.

When in doubt, it is already tracked.

Misfiled issues are not “new.” They are routing mistakes.

- If an idea exists in `samihalawa/2026-MANUS-oulang`, fix the project field and continue from the same issue.
- If a real implementation item exists in `Oulang Ideas & Inspirations`, move it into `samihalawa/2026-MANUS-oulang`.
- Only create a fresh issue when the action is materially different, not just better filed.

Already-completed issues are not “new” either.

- Before creating, extending, moving, or reopening a code issue, verify that it is not already obviously completed in the current project state.
- Do not rely on repo docs or prior audit markdown alone to close or reopen a code issue when current repo or live evidence is available.
- For code issues, current repo state and current live verification outrank historical docs.

## Candidate Scoring

Use this scoring lens before creating anything:

- `impact`: does it plausibly improve revenue, conversion, retention, or trust?
- `fit`: does it fit the current stack and existing product shape?
- `effort realism`: can it start without major schema upheaval?
- `appeal`: would this feel obviously useful to the product, not just technically clever?

Only create issues that are high enough on all four.

## Workflow

### Step 1. Build a source ledger

Before extraction, list the exact sources you will use.

For current-thread mode:

- current conversation

For recent-history mode:

- current conversation
- selected recent Codex sessions
- selected recent Claude sessions
- selected recent repo notes / docs / Notion / Drive items if explicitly requested

Record whether each source was fully read or sampled.

### Step 2. Fetch existing issues first

Always fetch both target projects before extracting candidates:

```text
list_issues(project="samihalawa/2026-MANUS-oulang", team="Pime", limit=250)
list_issues(project="Oulang Ideas & Inspirations", team="Pime", limit=250)
```

Build a compact inventory of likely duplicates.

For code-focused runs, also mark each close match as one of:

- `open and still relevant`
- `already completed`
- `misfiled`
- `stale/hallucinated`

Also build a compact inventory of likely routing mistakes:

- opportunities or idea-language issues living in `samihalawa/2026-MANUS-oulang`
- real code/debugging/UX-fix issues living in `Oulang Ideas & Inspirations`
- addressed or canceled items that should block new creation

### Step 3. Extract candidate items

For each candidate capture:

- `title`
- `description`
- `type`: `code` or `idea`
- `priority`
- `labels`
- `source_refs`
- `why_now`

### Step 4. Dedupe aggressively

For each candidate, decide one:

- `already_tracked`
- `extend_existing`
- `move_existing`
- `create_new`
- `reject_for_fit`

Check similarity by:

- title concept
- action/outcome
- same product surface
- same monetization or bug class
- same core job, even if the current project assignment is wrong

### Step 5. Prefer extending over duplicating

If an issue already exists and the conversation adds real value:

- new evidence
- new numbers
- sharper scope
- clearer stack-fit framing

then update that issue instead of creating a sibling ticket.

Only create a new issue when the task is materially separate.

Never end a cleanup or extraction pass with “completed” language while the checklist still shows unresolved extraction or dedupe steps.

If the issue already exists but is in the wrong project:

- move the existing issue to the correct project first
- then extend it if useful
- do not leave the misfiled issue behind as historical clutter

### Step 6. Route to the correct project

- `code` -> `samihalawa/2026-MANUS-oulang`
- `idea` -> `Oulang Ideas & Inspirations`

Project routing examples:

- “Fix the Twilio spend guardrail regression” -> `samihalawa/2026-MANUS-oulang`
- “Test a jobs employer punch-card package” -> `Oulang Ideas & Inspirations`
- “Investigate dead clicks on a monetization surface” -> `samihalawa/2026-MANUS-oulang`
- “Consider optional WhatsApp / WeCom re-engagement channel strategy” -> `Oulang Ideas & Inspirations`

### Step 7. Write clean issues

#### Title rules

- concise
- action-oriented
- no filenames
- no line numbers
- no raw SQL
- no noisy jargon unless central to the task

#### Description rules

For ideas:

- 2-4 short sentences
- say what the opportunity is
- why it matters
- why it fits now

For code issues:

- concrete current problem
- expected outcome
- key evidence if relevant

Do not dump raw logs or giant analysis text into the issue.

For moved issues:

- preserve the existing issue whenever possible
- update the description only enough to explain the corrected routing or sharper scope
- do not rewrite useful history out of the ticket

## Priority Heuristics

- `1 Urgent`: active revenue leak, broken payment/commercial flow, severe trust break on a monetization-critical surface
- `2 High`: strong monetization or conversion upside, high-confidence fit, likely near-term value
- `3 Medium`: useful but less immediate
- `4 Low`: only if still clearly worth tracking

## Labels

Prefer a small label set.

Use only labels that actually exist in the workspace when possible. Typical useful choices:

- `💰 Revenue`
- `⚡ Fast Win`
- `🎨 UI`

If label availability is unclear, create the issue without forcing bad labels.

## Oulang-specific Guardrails

For Oulang-oriented runs:

- prefer marketplace, jobs, talent, chat, recharge, premium, admin, merchant, and listings surfaces
- prefer opportunities that tighten the loop:
  - discover -> contact -> transact/promote -> return -> manage
- reject giant speculative ideas that would obviously require a new product architecture
- reject things that are interesting but not useful or appealing for current users

## Report Format

Return a compact table:

| Status | Issue | Project | Action |
| --- | --- | --- | --- |
| ✅ Created | PIM-XXX: title | project | New issue |
| 📝 Extended | PIM-YYY: title | project | Updated existing issue |
| ⏭️ Already tracked | PIM-ZZZ: title | project | No action needed |
| 🚫 Rejected for fit | candidate title | — | Not added |

If all items were already tracked or rejected, use the exact required sentence:

> "All items from this conversation are already tracked in Linear — nothing new to add."

If items were moved but no new ones were created, report them explicitly in the table rather than pretending nothing changed.

## Non-Negotiable Rules

1. Fetch existing issues before creating anything.
2. Do not create duplicates when an existing issue is close enough.
3. Do not create issues from vague context or raw brainstorming alone.
4. Do not add items the user rejected.
5. Do not add meta-issues about the sync process itself.
6. In recent-history mode, make source coverage explicit.
7. For Oulang runs, prioritize monetization-relevant, stack-realistic, low-schema items.
8. A smaller, cleaner backlog is better than a comprehensive noisy one.
9. Misfiled issues must be moved to the correct project, not duplicated.
10. Do not treat “already addressed,” “canceled,” or “hallucinated” items as fresh candidates unless new evidence materially reopens them.
