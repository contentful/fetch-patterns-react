# Architectural Decision Records

Decision records for `fetch-patterns-react`. One file per decision, named
`YYYY-MM-DD-kebab-case-title.md`.

- [2026-08-25 — Ignore dependency install scripts](./2026-08-25-ignore-dependency-install-scripts.md) — `.npmrc` sets `ignore-scripts=true`, removing install-time code execution from the dependency tree at the cost of `fsevents` not being built.
