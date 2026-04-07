---
name: conversation-to-linear
description: >
  Extract actionable items from the current conversation and create Linear issues, deduplicating against existing issues.
  Use when: user says "sync to linear", "create linear issues", "add to linear", "update linear from this chat",
  or at the end of an analysis/brainstorm session to capture outcomes.
  Idempotent: running multiple times in the same conversation only adds genuinely new items not already in Linear.
---

# Conversation → Linear Issue Sync Skill

## Purpose

Scan the current conversation for **explicitly mentioned** actionable items (bugs, features, ideas, monetization plays, implementation tasks) and sync them to Linear — deduplicating against what already exists, extending existing issues when valuable new info is found, and creating new issues only when genuinely needed.

## Target Projects (FIXED — never change these)

| Project | Purpose | Route here when... |
|---------|---------|--------------------|
| `samihalawa/2026-MANUS-oulang` | Code/dev/implementation work | Bug fixes, feature implementations, technical debt, infrastructure, UX fixes, code cleanup — anything an engineer could implement |
| `Oulang Ideas & Inspirations` | Strategy/ideas/business decisions | Business strategy, monetization plays, pricing changes, partnership moves, market analysis insights — requires human/business decision |

**Team**: always `Pime`

## Available Linear Tools (USE ONLY THESE)

| Tool | Purpose | When to use |
|------|---------|-------------|
| `list_issues` | List issues with filters | ALWAYS first — fetch all existing issues from both projects |
| `get_issue` | Get full issue details | When title match is close but you need full description to confirm duplicate |
| `save_issue` | Create or update an issue | Create new issues OR update existing ones (pass `id` to update) |
| `list_comments` | List comments on an issue | Check if insight was already added as a comment |
| `save_comment` | Add comment to an issue | Extend an existing issue with new valuable info |
| `list_issue_labels` | List available labels | Reference for valid label names |
| `list_issue_statuses` | List available statuses | Reference for valid status names |
| `list_projects` | List projects | Verify project names if needed |

### ⛔ NEVER USE: `research` tool

The `research` tool is an AI-powered search — you ARE already AI. Using it is redundant and wastes a call. Always use `list_issues` with filters instead.

## Idempotency Contract

This skill is safe to run repeatedly in the same conversation:

1. **First run**: Lists all existing issues, extracts conversation items, creates only genuinely new ones.
2. **Second+ run**: Re-fetches existing issues (including those just created), compares again. Only adds genuinely new items or extends existing ones with valuable new info.
3. **Nothing new**: If all items are already tracked and no valuable extensions exist, respond exactly:
   > "All items from this conversation are already tracked in Linear — nothing new to add."

## Workflow

### Step 1 — Fetch ALL existing Linear issues (ALWAYS DO THIS FIRST)

Before extracting anything from the conversation, fetch the complete issue list from BOTH projects:

```
list_issues(project="samihalawa/2026-MANUS-oulang", team="Pime", limit=250)
list_issues(project="Oulang Ideas & Inspirations", team="Pime", limit=250)
```

Build a compact inventory of all existing issues. Output this inventory as a compact paragraph:

> **Existing issues (X total across 2 projects):** PIM-332: Reprice tiers, PIM-333: Payment CTA at free_limit wall, PIM-334: Gate contact reveals, ...

This inventory is essential context for deduplication.

### Step 2 — Extract conversation items

Scan ALL previous messages in the current conversation (both user and assistant). Extract only items that are:

- **Explicitly stated** as tasks, bugs, ideas, features, fixes, or action items
- **Concrete enough** to be an issue (not vague discussion or context)
- **Mentioned in the main conversation flow** (not just metadata or skill instructions)

Do NOT extract items from:
- General discussion, background context, or preamble
- Questions asked but not answered with a decision
- Things mentioned only as examples or hypotheticals
- Information that is purely analytical with no action implied
- Things the user rejected or dismissed in conversation
- The Linear sync process itself

For each extracted item, determine:
- `title`: concise, consistent tone — not too specific (no filenames), not too vague. Same style as existing issues.
- `description`: 2-4 sentences with context from the conversation. Include evidence/numbers if mentioned.
- `type`: `code` (engineer can implement) or `idea` (requires business decision)
- `priority`: 1=Urgent, 2=High, 3=Medium, 4=Low
- `labels`: from available labels: `💰 Revenue`, `Bug`, `Improvement`, `Feature`

### Step 3 — Deduplicate and decide action for each item

For each conversation item, compare against the existing issue inventory:

#### Action 1: SKIP (already tracked)
If an existing issue covers the same concept — even if worded differently — mark it as "already tracked".
Be aggressive about deduplication. When in doubt, it's already tracked.

#### Action 2: EXTEND (existing issue + valuable new info)
If an existing issue covers the same topic BUT the conversation contains genuinely valuable new data (numbers, evidence, a sub-task, a new angle), then EXTEND it:
- **Option A**: Use `save_comment` to add a comment with the new insight to the existing issue.
- **Option B**: If the new info is a distinct sub-task, use `save_issue` with `parentId` set to the existing issue ID to create a sub-issue.
- Choose Option A (comment) by default. Only use Option B (sub-issue) if the new info is a clearly separate actionable task.

#### Action 3: CREATE (genuinely new)
If no existing issue covers this concept at all, create a new issue:
- `code` type → project: `samihalawa/2026-MANUS-oulang`
- `idea` type → project: `Oulang Ideas & Inspirations`
- team: `Pime`
- Use `save_issue` with: title, description, priority, labels, project, team.

### Step 4 — Execute actions

For each item, execute the decided action (skip / comment / sub-issue / create).

### Step 5 — Report

Output a summary table:

| Status | Issue | Project | Action |
|--------|-------|---------|--------|
| ✅ Created | PIM-XXX: title | project | New issue |
| 📝 Extended | PIM-YYY: title | project | Added comment/sub-issue |
| ⏭️ Already tracked | PIM-ZZZ: title | project | No action needed |
| 🚫 Nothing new | — | — | — |

If ALL items were already tracked and no extensions were made, say exactly:
> "All items from this conversation are already tracked in Linear — nothing new to add."

## Hard Rules

1. **ONLY extract items explicitly mentioned in conversation messages.** Never invent or extrapolate.
2. **ALWAYS fetch existing issues FIRST** before extracting or creating anything.
3. **ALWAYS output the compact existing-issues inventory** so the user can see what's already tracked.
4. **Deduplicate aggressively** — false negatives (missing a new item) are better than false positives (creating duplicates).
5. **Extend over create** — if the idea exists but has new valuable info, add a comment rather than a new issue.
6. **Sub-issues only when clearly a separate actionable task** under an existing parent concept.
7. **Consistent tone** — titles should be concise, action-oriented, same style as existing issues. Not too specific (no filenames/line numbers), not too vague.
8. **Never create issues about** the Linear sync process itself, skill execution, or meta-process.
9. **Never create issues about things the user rejected** or dismissed in conversation.
10. **Never use the `research` tool** — it's AI-powered and redundant. Use `list_issues` with filters.
11. **Available labels**: `💰 Revenue`, `Bug`, `Improvement`, `Feature` — use the most relevant one(s).
12. **Available statuses**: Backlog, Todo, In Progress, In Review, Done, Canceled, Duplicate — new issues default to Backlog.
13. **The two target projects are fixed** — never create issues in other projects.
14. **Team is always `Pime`.**
15. **When updating an existing issue**, pass its `id` (e.g. `PIM-370`) to `save_issue`. When creating new, do NOT pass `id`.
