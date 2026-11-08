---
name: source-of-truth-precedence
description: Ground facts in code/config first, then README, then bubble-up docs, then wiki — verify up, propagate corrections down
metadata:
  type: feedback
---

<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

A guessed fact that lands on a public surface is expensive — it has to be chased down across every place it leaked. Establish facts from the most authoritative tier before stating them.

**Precedence (highest authority first):**
1. **Code / config / running system** — source files, `application.yml`/properties, deployed container logs, `docker stats`, a live `tools/list` / health probe. This is ground truth.
2. **Repo README** — written next to the code; usually right but can lag a change.
3. **Bubble-up docs** — GitHub profile README, external sites. Derived / marketing; assume stale until checked against 1–2.
4. **Resource docs (wiki)** — reference pages (project pages, comparison pages); most derived, most prone to drift.

**How to apply:**
- **Verify upward.** To learn or assert a fact, start at tier 1. Lower tiers are hints, not evidence.
- **Never infer behaviour from a name.** A dependency/artifact id, class name, or filename is not proof of behaviour — check what the code/runtime does.
- **On disagreement, the higher tier wins**, and you **propagate the correction downward**: fix the README, then the bubble-up docs, then the wiki — in that order, so the chain stays consistent.
- **Don't ship an unverified tech claim** when tier 1 is cheaply checkable (one `grep`, one `curl`, one `docker logs`).

**Cautionary example (2026-06-15):** I asserted "MediaWikiMCP uses SSE" from the `quarkus-mcp-server-sse` artifact id in its pom — a name, not behaviour. At 1.12.0 that artifact is relocated to `-http` and ships both transports; the deployed logs showed clients connecting with `transport=STREAMABLE_HTTP`. The wrong "SSE" had already propagated to the portfolio site and a comparison page on the wiki before it was caught. Grounding in tier 1 (the logs / the client URL `/mediawiki/mcp`) would have prevented it.

