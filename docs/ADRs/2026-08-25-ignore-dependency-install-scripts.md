# Ignore dependency install scripts

- **Date:** 2026-08-25
- **Status:** Accepted

> This record was written on 2026-08-25 from the commit history. It documents an
> existing decision rather than a new one. The rationale below is reconstructed
> from the change itself and its measurable effect on this repository; the
> original pull request carried no description, so the stated motivation is an
> inference and is labelled as such.

## Context

`fetch-patterns-react` installs its dependency tree from `pnpm-lock.yaml`. By
default, npm and pnpm execute `preinstall`, `install`, and `postinstall`
lifecycle scripts declared by packages in that tree — arbitrary code from
transitive dependencies, run on any machine that installs the project, including
a contributor's laptop.

For this repository the cost of allowing that is close to zero. Inspecting
`pnpm-lock.yaml` for packages that declare install scripts turns up exactly one
entry marked `requiresBuild: true`:

```
/fsevents@2.3.2
```

`fsevents` is an optional macOS-only native file-watching module. Nothing in the
four scripts this project exposes — `next dev`, `next build`, `next start`,
`next lint` (`package.json`) — depends on a dependency lifecycle script to
produce a working install.

## Decision

Set `ignore-scripts=true` in `.npmrc` at the repository root, so that no
dependency lifecycle script runs during install.

Evidenced by commit
[`b1612ca`](https://github.com/contentful/fetch-patterns-react/commit/b1612ca1cbf8885ed4bfed424024c51100ab3109)
— *"chore: [] ignore npm scripts (#6)"*, James Bourne, 2025-11-26 — which added
`.npmrc` as a single-line file and changed nothing else. The empty `[]` in the
subject line suggests it arrived as part of a templated change applied across
repositories rather than as a decision taken for this repository specifically,
but this repository's history alone does not establish that.

## Consequences

- Install-time arbitrary code execution from the dependency tree is removed as an
  attack surface. This is the benefit the change buys.
- `fsevents` is not compiled. On macOS, Next.js dev-mode file watching therefore
  cannot use the native FSEvents binding and falls back to another watcher. In
  practice `pnpm dev` still picks up edits; expect it to be somewhat less
  efficient than it would be otherwise. This is the only side effect identified
  in the current lockfile.
- Adding a dependency that genuinely needs a build step — a native addon, or a
  tool that fetches a binary in `postinstall` — will fail or silently
  half-install while this setting is in place. Anyone doing that must either pick
  a different dependency or replace this blanket setting with a narrower
  allowlist (for example pnpm's `onlyBuiltDependencies`), and should update this
  record rather than deleting `.npmrc`.
- The setting applies to any package manager that reads `.npmrc`, so the
  behaviour is the same whether a contributor runs `pnpm install` or, against
  this repo's convention, `npm install`.
