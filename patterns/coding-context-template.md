<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

# Pattern — Coding Context Template

Before handing a non-trivial coding task to an agent (or starting one yourself), assemble the **Context** half of `Prompt = Context + Request` explicitly, rather than assuming it's implied. Framework from a conference talk slide ([source video](https://youtu.be/j1v_ldfSkkU), watched 2026-09-22): Context breaks down into eight categories, each with its own home in this repo.

## The eight categories, and where they already live

| Category | Where it lives here |
|---|---|
| Guidelines | The always-loaded rules file |
| Best Practices | The patterns folder (this one) |
| Examples | Prior code in the same repo, or a comparable sibling project |
| Previous Code | The specific file/module this task should mirror — name it, don't assume the agent will find it |
| Libraries/Frameworks | Tech defaults and the dependency research workflow |
| Where to put focus | Stated per-task — scope the ticket/request explicitly |
| What to avoid | Feedback memories — corrections already learned the hard way |
| Detailed Instructions | The actual task/ticket description |
| Test list | Written before code for new behaviour, on the PR description — see [canon-tdd](canon-tdd.md) |

## How to use it

At the start of a sizable task — new project, unfamiliar stack, or anything an agent is likely to get wrong without steering — run through the eight rows. For each: cite the existing pointer, or note explicitly that there's a gap (no prior example, no established best practice yet). A stated gap is useful signal on its own — it tells you where the agent is working with less context than usual.

## Why lightweight here

For personal TDD Java projects, most categories are already covered piecemeal — the rules file, the patterns folder, and the project code itself. This doc is a checklist pointing at what exists, not a new source of content. No auto-loaded scaffold file yet; promote to one only if a project's context turns out too thin for the checklist to just point at (e.g. a stack with no prior code/best-practice base to draw on, such as a from-scratch C# codebase).

**Reference:** drafted 2026-09-23.
