# AGENTS.md

Operating notes for coding agents working in `contentful/fetch-patterns-react`.

## What this repo is

A single-page Next.js 13 App Router demo called **GoodBois Club**. It renders the
same "fetch a random dog photo" behaviour twice side by side — once as a React
Server Component, once as a Client Component — to illustrate the difference. It
was written as the companion demo for an article on React Server Components (see
`README.md`).

It is a teaching artifact, not a product. `catalog-info.yaml` declares
`contentful.com/service-tier: "4"` and `owner: group:team-devrel`. Treat clarity
of the demonstrated pattern as the primary quality bar.

## Commands

Only four scripts exist (`package.json`):

```
pnpm dev      # next dev    — local dev server on http://localhost:3000
pnpm build    # next build
pnpm start    # next start  — serve a prior build
pnpm lint     # next lint   — ESLint via eslint-config-next
```

There is **no test suite** and **no test script**. Do not claim tests pass, and
do not add a test runner unless the task explicitly asks for one.

Use `pnpm` — the repo is locked with `pnpm-lock.yaml` (lockfileVersion 6.0).
`README.md` still says `npm i && npm run dev`; that text predates the lockfile
situation and using `npm` will produce a `package-lock.json` that does not belong
in this repo. See `CONTRIBUTING.md`.

## Layout you need to know

- `app/layout.tsx` — root layout, Inter via `next/font/google`, page metadata.
- `app/page.tsx` — the `/` route. Imports the two route pages below **as
  components** and renders them in a two-column comparison.
- `app/server/page.tsx` — Server Component. `async` page, awaits `fetch(...)`
  with `cache: "no-cache"` during render.
- `app/client/page.tsx` — Client Component (`"use client"`). Fetches in a
  `useEffect` and holds the URL in `useState`.
- `app/Img.tsx` — the one shared presentational component both sides render.
- `app/globals.css` — Tailwind directives plus the `--foreground-rgb` /
  `--background-rgb` custom properties and their `prefers-color-scheme: dark`
  overrides.

Because `app/server/page.tsx` and `app/client/page.tsx` are both route pages and
imported components, editing either changes **two** surfaces: the `/server` or
`/client` route, and the corresponding column on `/`. Check both.

## Constraints to respect

- **The asymmetry is the point.** The server side passes `cache: "no-cache"` and
  the client side passes no options. Do not "fix" this into consistency — it is
  the behaviour the demo exists to show.
- **Dependency versions are exact-pinned**, no `^` or `~` (`package.json`).
  Keep that style when changing versions.
- **`.npmrc` sets `ignore-scripts=true`.** Do not remove it. See
  `docs/ADRs/2026-08-25-ignore-dependency-install-scripts.md`.
- `@next/next/no-img-element` is deliberately disabled in `.eslintrc.json` so
  `app/Img.tsx` can use a raw `<img>`. Leave that rule off.
- There is no `next.config.js`. Next.js defaults apply.
- Path alias `~/*` maps to the repo root (`tsconfig.json`); the existing code
  uses relative imports instead.
- The upstream image source is the public `https://dog.ceo` API. No credentials,
  no `.env` file, and nothing in this repo needs a Contentful token.

## CI

`.github/` contains only `CODEOWNERS` (`* @contentful/team-devrel`). There are no
GitHub Actions workflows in this repo, so nothing lints or builds your change
automatically — run `pnpm lint` and `pnpm build` yourself before opening a PR.
