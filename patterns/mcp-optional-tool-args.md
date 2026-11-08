<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

# Pattern: Optional tool arguments in quarkus-mcp-server adapters

**Context:** `@Tool` methods in the Quarkus MCP adapters ([KanbanMCP](https://github.com/marcushowarth/KanbanMCP) / [MediaWikiMCP](https://github.com/marcushowarth/MediaWikiMCP)) that wrap a backend JSON-RPC / HTTP call.

## The pattern

For an optional tool argument, use a **boxed type** with `@ToolArg(required=false)` (`Integer`, `String`, or `Optional<T>` — all produce a correct *optional* MCP schema).

**Never use a primitive for an optional arg** — both failure modes are traps (verified empirically 2026-06-16, primitive-optional probe):
- `int` *without* `required=false` → schema marks it **required**; the model always sends it (not optional at all).
- `int` *with* `required=false` → schema marks it **optional**, but omitting it at call time fails with JSON-RPC **`-32603 "Internal error"`** — the framework can't bind "absent" to a primitive and does **not** default it to `0`; the tool body never runs. The schema *invites* a crash.

A boxed `Integer` / `Optional<T>` with `required=false` is the only correct optional: omitted → `null`, which you map to your sentinel (`0` for `moveTaskPosition`). ([quarkiverse issue 194](https://github.com/quarkiverse/quarkus-mcp-server/issues/194), closed: the maintainers steer optional args to `Optional<T>` / `required=false` rather than an annotation-level default — i.e. exactly this pattern.)

When the arg is omitted the framework passes the method `null` (proven through the real `tools/call` path by `MoveTaskPositionMcpTest`, not assumed). How you then build the outgoing RPC params depends on **whether the backend tolerates an absent key** — two styles:

| Backend tolerates absent key? | Null-handling | Example |
|---|---|---|
| Yes — absent = "leave unchanged" | **Omit the key**: `if (x != null) params.put(...)` | `updateSubtask` (status), `moveTaskToProject` |
| No — key must be present | **Send a sentinel**: `x != null ? x : 0` | `moveTaskPosition` (`swimlane_id`; `0` = "keep current lane") |

## The trap, and the rule

`moveTaskPosition` is the odd one out: Kanboard's RPC **rejects** an absent `swimlane_id` (`-32602 "Wrong number of arguments"`, whole move rejected) and treats `0` as "leave the swimlane as-is". Copy-pasting the omit-key style there would silently error.

Both ends are **undocumented**:
- the framework's optional-arg → `null` behaviour (not in the annotations reference),
- Kanboard's `swimlane_id: 0` = keep-lane.

So the rule: **verify the RPC's absent-key tolerance empirically** (throwaway card + raw JSON-RPC probe), choose omit-key vs sentinel, then **lock it with a test** instead of re-deriving:
- method-level — `BoardToolsTest` (null → sentinel; explicit → passthrough)
- protocol-level — `MoveTaskPositionMcpTest` (McpAssured `tools/call` with the arg omitted → sentinel reaches the client)

**Where the framework proof lives:** the generic behaviour (primitive vs boxed optional) is pinned in isolation in [QuarkusMCPPlaypen](https://github.com/marcushowarth/QuarkusMCPPlaypen) (`OptionalArgProbes` + `OptionalArgBindingTest`, FINDINGS.md #1) — adapters reference it rather than re-prove it. KanbanMCP keeps only tests of its own tool (`BoardToolsTest`, `MoveTaskPositionMcpTest`).

**Provenance:** [KanbanMCP](https://github.com/marcushowarth/KanbanMCP) PR 1 (2026-06-16).
