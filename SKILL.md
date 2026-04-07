---
name: conversation-to-linear
description: >
  Extract actionable items from the current conversation and create Linear issues, deduplicating against existing issues.
  Use when: user says "sync to linear", "create linear issues", "add to linear", "update linear from this chat",
  or at the end of an analysis/brainstorm session to capture outcomes.
  Idempotent: running multiple times in the same conversation only adds genuinely new items not already in Linear.
---

# Conversation to Linear — Skill

## Purpose

Scan the current conversation for **explicitly mentioned** actionable items (bugs, features, ideas, monetization plays, implementation tasks) and create them as Linear issues in the correct project — deduplicating against what already exists.

## Idempotency Contract

This skill is safe to run repeatedly in the same conversation:

1. **First run**: Creates all new issues found in conversation messages.
2. **Second+ run**: Compares conversation items against Linear issues (including those just created). Only adds genuinely new items. If nothing new exists, responds: "All items from this conversation are already tracked in Linear — nothing new to add."

## Workflow

### Step 1 — Collect conversation items

Scan ALL previous messages in the current conversation (both user and assistant). Extract only items that are:

- **Explicitly stated** as tasks, bugs, ideas, features, fixes, or action items
- **Concrete enough** to be an issue (not vague discussion or context)

Do NOT invent issues from:
- General discussion or background context
- Questions asked but not answered
- Things mentioned only as examples or hypotheticals
- Information that is purely analytical with no action implied

For each item, determine:
- `title`: concise issue title
- `description`: 2-3 sentence description with context from the conversation
- `type`: one of `code` | `idea`
  - `code` = bug fix, feature implementation, technical debt, infrastructure — anything an agent could implement
  - `idea` = business strategy, monetization play, partnership move, pricing change, analysis insight — requires human decision
- `priority`: 1=Urgent, 2=High, 3=Medium, 4=Low
- `labels`: appropriate labels (e.g. "💰 Revenue", "Bug", "Feature", "Improvement")

### Step 2 — Fetch existing Linear issues

Query BOTH projects to check for duplicates:

```
Linear:list_issues(project="samihalawa/2026-MANUS-oulang", team="Pime", limit=250)
Linear:list_issues(project="Oulang Ideas & Inspirations", team="Pime", limit=250)
```

Build a list of existing issue titles and descriptions.

### Step 3 — Deduplicate

For each conversation item, check if a substantially similar issue already exists by comparing:
- Title similarity (same concept, even if worded differently)
- Description overlap (same core action/outcome)

If an existing issue covers the same ground — even partially — do NOT create a duplicate. Mark it as "already tracked" with the existing issue ID.

Be aggressive about deduplication. When in doubt, it's already tracked.

### Step 4 — Create new issues

For each genuinely new item:

- `code` type → project: `samihalawa/2026-MANUS-oulang`
- `idea` type → project: `Oulang Ideas & Inspirations`
- team: `Pime`

Use `Linear:save_issue` with title, description, priority, labels, project, team.

### Step 5 — Report

Output a summary table:

| Status | Issue | Project | Priority |
|--------|-------|---------|----------|
| ✅ Created | PIM-XXX: title | project | priority |
| ⏭️ Already tracked | PIM-YYY: existing title | project | — |
| 🚫 Nothing new | — | — | — |

If ALL items were already tracked, say exactly:
> "All items from this conversation are already tracked in Linear — nothing new to add."

## Hard Rules

- ONLY extract items explicitly mentioned in conversation messages. Never invent or extrapolate.
- ALWAYS fetch existing issues before creating anything.
- ALWAYS deduplicate aggressively — false negatives (missing a new item) are better than false positives (duplicates).
- Never create issues about the Linear sync process itself.
- Never create issues about things the user rejected or dismissed in conversation.
- The two target projects are fixed: `samihalawa/2026-MANUS-oulang` (code) and `Oulang Ideas & Inspirations` (ideas).
- Team is always `Pime`.
