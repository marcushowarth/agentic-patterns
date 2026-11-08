---
name: board-review
description: Review the owner's kanban TODO board (project 7) — stale cards, WIP limit, Ready that isn't ready, Someday candidates, Done to close, agent-doable split and breakdown. Use on "board review", "review the board", "groom the backlog", "quick wins", "what can you pick up", or the fortnightly reminder. Proposes first, applies only what the owner approves.
---

<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

# Board Review

Keeps the TODO board (project 7) honest. The daily cron already does the dumb part (Backlog→Someday at 90 days, Ready→Backlog at 60 days without a move — see the kanban memory note). This skill does the judgement part.

Board facts (verify against the kanban memory note before relying on them):
- Columns: Someday 67 → Backlog 31 → Ready 32 → WIP 33 → Done 34.
- Lanes: Focus 42 (the current busy project) and General 9.
- Who does it is the **assignee**: `claude` (user 29) only when Claude can finish it autonomously, end to end; the owner (user 1) for anything real-world or mixed. Unassigned = not yet triaged.
- The aim is more `claude` cards: break mixed cards down so the autonomous part can run on its own (feeds human-in-the-loop work and a home agent host).

## Modes

- **Full review** (default, roughly fortnightly): every step below.
- **Quick wins** ("quick wins", "what can you pick up"): only Step 4, Ready + Backlog, top five each side.

## Step 1: Snapshot

Pull the board with `mcp__kanboard__getBoard` (project 7). For each open card note: column, lane, tags, category, due date, `date_moved`, last comment date. Staleness = days since the latest of `date_moved` and the last comment. Never use `date_modification` — bulk edits touch it.

Save the snapshot to the scratchpad (not the repo) so the apply step works from the same data.

## Step 2: Checks

| Check | Flag when |
|-------|-----------|
| WIP | more than 5 cards in WIP, or a WIP card with no move/comment for 14 days |
| Ready | a card that isn't actually startable (blocked link, waiting on someone, no clear next step) |
| Backlog | 60+ days stale — candidate for Someday before the cron does it blindly |
| Rescue | a card the cron will move within 7 days that shouldn't go (due date soon, security, live real-world event) |
| Due dates | overdue, or due within 14 days and not in Ready/WIP |
| Focus lane | a card in Focus that isn't the focus project, or a focus-project card left in General |
| Triage | no assignee, or no category |
| Breakdown | assigned to the owner but with a part Claude could finish alone — propose splitting it into a child card assigned to `claude` |
| Done | cards waiting to be closed (count only) |
| Duplicates | cards with near-identical titles — propose a link or a merge |
| Quick win | small and self-contained with a clear next step, whoever it's assigned to, and not yet tagged → propose the global `quick-win` tag. Also flag a tagged card that has grown or stalled → propose removing the tag |

A Kanboard same-column move does **not** reset `date_moved`. To rescue a card that stays in its column, move it out and back (e.g. Backlog→Ready→Backlog), then check `date_moved` changed.

## Step 3: Propose

Report in chat, grouped by action, one line per card: `#NNN title — reason → proposed action`. Lead with rescues (they're time-bound), then WIP, Ready, Focus, triage, quick-win tags, Someday candidates. End with the Done count.

Ask open questions in plain text (not AskUserQuestion). Keep personal/real-world calls to the owner — don't guess.

## Step 4: Agent queue

List the cards assigned to `claude` in Ready, then Backlog, ordered by: `quick-win` tag first, then Focus lane, then due date, then smallest-looking. For each, one line on what doing it would involve. These are what Claude can pick up next.

Then the breakdown candidates from Step 2. List the owner's cards in Ready/WIP briefly, so the owner sees their own queue.

Quick-wins mode stops here.

## Step 5: Apply

Only what the owner approved, card by card:
- Column moves: `mcp__kanboard__moveTaskPosition`, omit `swimlaneId` unless changing lane.
- Assignee, due dates, categories: `mcp__kanboard__updateTask` (`ownerId` 29 = claude, 1 = the owner).
- Breakdowns: `createTask` in project 7 assigned to 29, then `createTaskLink` child→parent (link_id 6).
- Tags: `mcp__kanboard__setTaskTags` replaces the whole set — read `getTaskTags` first and send the union. `quick-win` is a **global** tag (id 98): `getTagsByProject` doesn't list it, but `setTaskTags` resolves it by name. To remove it, send the set without it. The assignee, not a tag, says who does a card (no `agent` tag).
- Closing Done cards: follow whatever the kanban memory note currently says about closing (the rule there wins over this file).

Re-read each changed card to confirm, then add one comment per substantive change (not for routine moves).

## Step 6: Report

What changed, what was left and why, and the Done count. If the review surfaced a rule change (thresholds, lanes, assignment), propose it for `kanban.md` — don't edit the memory without saying so.
