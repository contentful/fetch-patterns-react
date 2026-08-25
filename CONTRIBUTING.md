# Contributing

This repo is a small teaching demo owned by `@contentful/team-devrel`
(`.github/CODEOWNERS`). It illustrates React Server Components versus Client
Components; changes should make that contrast clearer, not add features.

## Setup

Requires Node.js (the repo pins `@types/node` at 20.4.9) and pnpm.

```
pnpm install
pnpm dev
```

Then open <http://localhost:3000>.

Use **pnpm**. The lockfile is `pnpm-lock.yaml` (lockfileVersion 6.0), and
`npm install` would generate a competing `package-lock.json`. The `npm i && npm
run dev` instructions in `README.md` are older than that lockfile — if you touch
`README.md` for another reason, correcting them is welcome.

`.npmrc` sets `ignore-scripts=true`, so dependency install scripts are skipped by
design. Do not remove it to make an install "work"; see
`docs/ADRs/2026-08-25-ignore-dependency-install-scripts.md` for the one known
side effect (`fsevents` is not built).

## Scripts

```
pnpm dev      # next dev
pnpm build    # next build
pnpm start    # next start (requires a prior build)
pnpm lint     # next lint
```

## Before you open a PR

There are no GitHub Actions workflows in this repository, so nothing verifies your
change for you. Run these locally:

1. `pnpm lint` — must be clean.
2. `pnpm build` — must succeed.
3. `pnpm dev`, then look at `/`, `/server`, and `/client` in a browser. Both
   columns on `/` should show a dog photo; reload a few times.

Note that `app/server/page.tsx` and `app/client/page.tsx` are imported as
components by `app/page.tsx`, so editing one affects both its own route and the
matching column on `/`. Check both surfaces.

## Conventions

- TypeScript throughout, `strict: true`. No new `.js`/`.jsx` source files.
- Styling is Tailwind utility classes in JSX. Do not introduce CSS modules or a
  styling library.
- Pin dependency versions exactly, with no `^` or `~`, matching `package.json`.
- Double-quoted strings and semicolons, matching the existing files.
- Keep the deliberate asymmetry between the server and client fetches (the
  server one passes `cache: "no-cache"`, the client one passes nothing). It is
  the subject of the demo.
- `@next/next/no-img-element` is off in `.eslintrc.json` so `app/Img.tsx` can use
  a raw `<img>`. Leave it off.

## Commits and PRs

Commit subjects in this repo's history use Conventional Commit prefixes — `fix:`,
`chore:` (`git log`). Follow that. There is no release automation and no
published package (`package.json` sets `"private": true`), so no prefix triggers
a release.

Open PRs against `main`. `@contentful/team-devrel` owns every path and will be
requested automatically.

## Secrets

There are none, and there should not be. This app calls one public API
(`https://dog.ceo`) and needs no credentials — not even Contentful ones. Never
commit `.env` files, tokens, or keys, and do not edit `.gitignore` to permit them.
