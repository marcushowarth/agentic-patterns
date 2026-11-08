# Agentic patterns and skills

![Claude Code](https://img.shields.io/badge/Claude%20Code-D97757?logo=claude&logoColor=white)
![Markdown](https://img.shields.io/badge/Markdown-000000?logo=markdown&logoColor=white)
![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-6E56CF)
![License: MIT](https://img.shields.io/badge/License-MIT-green)

The working rules I give a coding agent (Claude Code) day to day, taken from a private repo that serves as its long-term memory. Each one started as a mistake or a near miss, became a note, and was kept because it changed what the agent did next time.

Everything under `patterns/`, `skills/` and `guardrails/` is **generated** from that repo (see [How this repo is published](#how-this-repo-is-published)), so it's the text the agent actually reads, with personal detail taken out.

## How the setup works

The private repo is loaded into every session:

- **Rules file:** always loaded. Safety rules, communication style, and the guardrails below.
- **Patterns:** coding conventions, read when a task needs them, and at the start of every new project.
- **Memories:** one fact per file. Corrections are written down with the reason and how to apply them, so the same mistake isn't made twice.
- **Skills:** Claude Code skills (`SKILL.md`) for repeated jobs: the daily journal, board reviews, job leads.

A self-hosted kanban board and wiki sit alongside, reached through two MCP servers I wrote, [KanbanMCP](https://github.com/marcushowarth/KanbanMCP) and [MediaWikiMCP](https://github.com/marcushowarth/MediaWikiMCP). The agent reads and updates tasks and docs there, rather than starting from scratch each conversation.

## What's here

### Patterns

| Pattern | Problem it solves | Before and after |
|---|---|---|
| [Canon TDD](patterns/canon-tdd.md) | An agent "doing TDD" writes every test, sees one red, implements everything, sees one green. Red-first holds, but nothing makes it think about the cases first. | **Before:** two ordering bugs reached dev and were found by hand. **After:** a behaviour test list goes on the PR first, then one test at a time. The trial also caught a red test committed through a grep-and-`&&` chain, so commits now gate on the test runner's exit code. Based on Kent Beck's [Canon TDD](https://newsletter.kentbeck.com/p/canon-tdd). |
| [README kept honest by a test](patterns/readme-sync-test.md) | READMEs that list tools or endpoints drift from the code. | **Before:** a README said "Tools (23)" while the code had 30. **After:** a test reads the README and fails the build when a registered tool is missing or the count is wrong. |
| [Version and build provenance](patterns/version-build-provenance.md) | "What's actually deployed right now?" across several repos. | Every deployable exposes semver, CI build number and git sha. The UI footer shows its own build and the API's. Built in at scaffold time, because retrofitting it was fiddly. |
| [Optional MCP tool arguments](patterns/mcp-optional-tool-args.md) | In quarkus-mcp-server, an optional `int` argument advertises itself as optional, then crashes with `-32603` when it's left out. | Boxed types only, null mapped deliberately (omit the key or send a sentinel, depending on the backend), checked against the real backend and locked with a protocol-level test. |
| [Coding context template](patterns/coding-context-template.md) | Agents get a request without the context behind it. | A checklist for the Context half of Prompt = Context + Request, pointing at where each part already lives, plus the test list. |

### Guardrails

| Guardrail | Problem it solves | Example |
|---|---|---|
| [Source-of-truth precedence](guardrails/source-of-truth-precedence.md) | A guessed fact lands on a public page and has to be chased everywhere it spread. | The agent said a server "uses SSE" from an artifact name. The logs showed Streamable HTTP, but the wrong claim had already reached the portfolio site and the wiki. Now: code and running system first, then README, then derived docs, and corrections flow downward. |

### Skills

| Skill | What it does |
|---|---|
| [board-review](skills/board-review/SKILL.md) | Reviews the kanban board: stale cards, WIP limit, cards in Ready that aren't ready, rescues before the cleanup cron moves them, and cards to split so the agent can finish one part alone. It proposes first and applies only what's approved. |
| [journal-update](skills/journal-update/SKILL.md) | Writes the session's work onto the right days of a wiki journal, dated from kanban comments and PR merge times rather than memory. It re-reads the page just before each write, because the human edits the same page through the day. |

The skills name a specific Kanboard board and MediaWiki journal layout (column and user IDs, page names). Treat them as worked examples to adapt rather than drop-in tools.

## Using them

- **Patterns and guardrails:** copy the ones you want into your own agent instructions (`CLAUDE.md`, a rules file, or a patterns folder the agent is told to read).
- **Skills:** copy a skill's folder into `.claude/skills/` in your project, then change the IDs, URLs and tool names to yours.

## How this repo is published

The source is a private repo, and this one is generated from it:

1. A file is marked with `public: true` in its frontmatter.
2. Inside a marked file, anything personal (ticket numbers, internal links, hostnames, private paths) sits in `<!-- private -->` blocks. The export removes them, so the agent still sees the full detail and this repo doesn't.
3. Links between notes are rewritten to links between files here. A link to anything not published stops the export.
4. A leak check looks for email addresses, own domains, internal hostnames, names, ticket numbers, private paths, IPs, account IDs and credential-like strings. **If anything matches, nothing is written.**
5. The diff is reviewed by hand before it's committed here.

It's the same approach as [ha-energy-automations](https://github.com/marcushowarth/ha-energy-automations), where a Home Assistant label picks what gets published.

## Licence

[MIT](LICENSE)
