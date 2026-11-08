<!-- Generated from a private repo by export-public.py; edit the source there, not here. -->

# Pattern — Version & Build Provenance

**Applies to every new project, captured at scaffold time.** Wiring this in upfront is ~15 min; retrofitting later is fiddly (learned on [FIN OPTICS](https://github.com/marcushowarth#fin-optics)).

Every **deployable artifact** exposes its build provenance, so you can tell at a glance exactly what's live.

## Three layers
- **Semantic version** — from the build tool (Maven `<version>`, `package.json` version). Bump at milestones; keep it meaningful (not `0.0.1-SNAPSHOT` forever).
- **Build number** — a CI monotonic counter (`github.run_number`), injected at deploy time as an env var / build constant. The cheap, always-increasing, human-readable handle.
- **Git sha** — exact commit, for precision (tooltip / detail line).

## By artifact type
- **Service** → a `/version` (or `/api/version`) endpoint returning `{version, buildNumber, gitSha, builtAt, <key dependency versions>}`.
- **UI** → a footer showing semver + build number (sha in tooltip); also fetch the service's `/version` so the page shows **both**, e.g. `UI v0.1 build 12 · API v0.1 build 47`.
- **Library** (not independently deployed) → version = its published artifact version; **surface it via the consumer's** `/version`. Don't give a non-deployed library its own build number — what matters is *which library version the live service was built against*.

## Wiring (GitHub Actions)
- Deploy job injects build metadata as env: `-e APP_BUILD_NUMBER=${{ github.run_number }}` (plus git sha and build time).
- The app reads them from env — e.g. Quarkus `@ConfigMapping`, Vite `define` → compile-time constants.

## Why
For multi-repo systems this closes the provenance loop — at a glance you know which UI build, which API build, and which engine/library version are live together. Invaluable when answering "what's actually deployed right now?".

**Reference implementation:** FIN OPTICS — `fin-optics-api` `/api/version`, `fin-optics-ui` footer (`UI <sha> · API <sha>`), `fin-model` version threaded through the API.
