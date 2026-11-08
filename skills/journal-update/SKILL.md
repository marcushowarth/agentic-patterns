---
name: journal-update
description: Add the work done in this session to the right daily journal pages on the wiki, marked done or open, each item on the day it actually happened. Use when the user says "update the journal", "since I last asked update the relevant journal page", "journal what we did", "add today's work to the journal", or anything about recording session work on the wiki journal. Not for creating a new day's page; that's journal-daily.
---

<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

# Journal Update

Records what was worked on in this session onto the existing `Journal:YYYY-MM-DD` pages on wiki.example.com. A session can span midnight, so each item goes on the day it happened, dated from kanban and GitHub, not from when it's written up.

Wiki and kanban via `mcp__claude_ai_MediaWikiMCP__*` and `mcp__claude_ai_KanbanMCP__*` (fall back to `mcp__mediawiki__*` / `mcp__kanboard__*` if those are the ones connected).

## Step 1: What's in scope

"Since I last asked" means everything after the last journal update in this conversation (or in the summary, if compacted). With no earlier update, the whole session.

List the work items: tickets touched, PRs merged, decisions made, things sent or posted. Skip chatter and dead ends.

## Step 2: Date each item from the systems, not memory

- **Kanban:** `getAllComments` on each ticket touched; `date_creation` is a Unix timestamp. Tasks created: `getTask` → `date_creation`. Moves: `date_moved`.
- **GitHub:** `gh pr list -R <owner/repo> --state merged --json number,title,mergedAt`; runs: `gh run view <id> --json createdAt`.
- Convert in the local zone: `TZ=Europe/London date -r <ts> '+%F %H:%M'`. GitHub times are UTC, so convert those too (BST is UTC+1).
- Things with no system record (a message the owner sent, a call) take the date from the conversation; if unclear, ask.

## Step 3: Read each target page

- `getPage` `Journal:{date}` for every date in scope. If a page doesn't exist, say so and offer journal-daily; don't create it here.
- `getSections` for the Tasks section number.
  - **From 2026-10-26** the day job has its own `== Day Job ==` section on work days, so Tasks has no child and `editSection` on Tasks touches only Tasks.
  - **On older pages** Tasks has a `===work===` child. `editSection` replaces the child too, so the new content must reproduce it unchanged.

## Step 4: Write the lines

- In `== Tasks ==`: session work from this repo is personal projects. On older pages, put it before `===work===`. Place it next to related existing lines.
- `* [x]` done, `* [ ]` still open (waiting on someone, a follow-up left). Open items from an earlier day that are still open go on the latest page only.
- One line per outcome: project name, kanban link, what happened. Sub-bullets `**` for detail, sparingly.
- Kanban refs as `[https://kanban.example.com/?controller=TaskViewController&action=show&task_id=NNN #NNN]`. PRs as plain "PR 34" (no GitHub links needed).
- Don't duplicate a line the owner already wrote; tick it or add a sub-bullet instead.
- Don't edit Day Job (or a legacy `===work===` subsection), Standup or Events; those are the owner's.

## Step 5: Edit and report

- **Re-read just before each write.** Immediately before each `editSection`, re-read the page (`getPage`, plus `getPageHistory` limit 1 for the latest revid). If anything changed since Step 3, rebuild the section from the fresh text. Never write from a copy read earlier in the turn: the owner edits these pages through the day, and `editSection` silently overwrites the whole section.
- The owner often has today's page open in the editor. An unsaved browser edit can't be seen or merged from here, so before writing say "saving to Journal:{date} now, save or close any open edit first".
- `editSection` per page, summary "Add <project> work for {date} (dates from kanban comments and PR merges)".
- Report: pages edited, items per page, which are left open.
