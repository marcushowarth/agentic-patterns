<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

# Pattern — README kept honest by a test

When a README documents **code-derived facts** (tool/endpoint tables, counts, supported formats, a "Tools (N)" header), back it with a test that **fails the build** when code and README drift.

**Why:** docs silently rot. [KanbanMCP](https://github.com/marcushowarth/KanbanMCP)'s README sat at "Tools (23)" while the code had 30 (missing `moveTaskToProject`) and nobody noticed. A test turns drift into a red build instead of a stale doc.

**How:**
- Discover the source of truth at runtime/reflection (registered tool names, mapped routes) — don't hardcode the list in the test.
- Assert every discovered item appears in the README, and any `(N)` count matches the discovered size.
- CI runs the tests on deploy, so a stale README blocks the deploy automatically — no manual remembering.

**Reference:** `ReadmeToolsTest` in KanbanMCP — reads `README.md`, checks each registered tool name appears as `` `name` ``, parses `## Tools (N)` against the registered count. (Promoted from a memory note.)
