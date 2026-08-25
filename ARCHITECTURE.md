# Architecture

`fetch-patterns-react` is a Next.js 13.4 application using the App Router. It has
one meaningful screen and exists to put two data-fetching strategies next to each
other so the difference is visible in a browser.

## Component and route map

```
app/
  layout.tsx        root layout — <html>/<body>, Inter font, metadata
  page.tsx          route "/"  — two-column comparison
  server/page.tsx   route "/server" — Server Component fetch
  client/page.tsx   route "/client" — Client Component fetch
  Img.tsx           shared <img> presentational component
  globals.css       Tailwind entry + CSS custom properties
```

`app/page.tsx` does something worth calling out explicitly: it imports
`app/server/page.tsx` and `app/client/page.tsx` and renders them as ordinary
components inside its own layout.

```tsx
import ServerRequest from "./server/page";
import ClientRequest from "./client/page";
```

So each file is simultaneously a routable page and a child component of `/`. The
`/server` and `/client` routes render the bare image with no surrounding chrome;
`/` renders both under headings. There is no separate `components/` directory —
the route module *is* the component.

## The two fetch paths

Both call the public `https://dog.ceo/api/breeds/image/random` endpoint and both
render the resulting URL through `app/Img.tsx`. Everything else differs.

**Server** (`app/server/page.tsx`) — the default in the App Router; no directive.
The page component is `async` and awaits the fetch during render:

```tsx
const resp = await fetch("https://dog.ceo/api/breeds/image/random", {
  cache: "no-cache",
});
```

The request happens on the server, `cache: "no-cache"` opts this fetch out of
Next.js's Data Cache so each render hits the API, and the HTML arrives at the
browser with the image URL already in it. No client JavaScript is needed to
populate it, and no loading state exists because there is nothing to wait for
once the response is streamed.

**Client** (`app/client/page.tsx`) — marked `"use client"`. The same fetch runs in
the browser after hydration, from inside `useEffect`, and the result lands in
`useState`:

```tsx
const [boi, setBoi] = useState("");
useEffect(() => { /* fetch, then setBoi(...) */ }, []);
```

The initial render therefore has `boi === ""`, which means `app/Img.tsx` renders
`<img src="">` for one frame before the effect resolves. That empty first paint is
part of what the demo shows: the server column is populated on arrival, the client
column is not.

## Styling

Tailwind CSS 3.3, configured in `tailwind.config.ts` with `content` scoped to
`./app/**/*.{js,ts,jsx,tsx,mdx}` and an otherwise empty `theme`. All styling is
utility classes in JSX. `postcss.config.js` wires `tailwindcss` and
`autoprefixer`. `app/globals.css` holds the three `@tailwind` directives plus the
light/dark colour custom properties applied to `body`.

## Build and configuration

There is no `next.config.js`; the app runs on stock Next.js defaults.
`tsconfig.json` is the Create Next App baseline with `strict: true`,
`moduleResolution: "bundler"`, `noEmit: true`, the `next` TypeScript plugin, and a
`~/*` path alias pointing at the repo root. ESLint extends
`next/core-web-vitals` with `@next/next/no-img-element` turned off.

`package.json` declares every dependency — including `typescript`, `eslint`, and
the `@types/*` packages — under `dependencies` rather than `devDependencies`, all
at exact pinned versions. Dependencies are installed with pnpm
(`pnpm-lock.yaml`, lockfileVersion 6.0), and `.npmrc` sets `ignore-scripts=true`
so no dependency lifecycle script runs on install
(`docs/ADRs/2026-08-25-ignore-dependency-install-scripts.md`).

## What is not here

No test suite, no CI workflows (`.github/` holds only `CODEOWNERS`), no server of
its own, no database, no authentication, and no Contentful SDK or credentials —
despite the repository name, this app does not talk to Contentful. `README.md`
points at a `goodbois.vercel.app` deployment run by the original author;
this repository contains no deployment configuration for it.
