---
name: ssr
description: Use when server-side rendering a Fancy UI + Inertia app or fixing SSR problems — getting real HTML on the first byte, wiring createFancyServer / setupFancyApp / hydrateRoot + the SSR node process, fixing hydration mismatches (React #418), and deciding what can/can't render on the server (FancyClientOnly). This is the RENDERING mechanic; for meta tags / Open Graph / sitemaps / structured data see the `seo` skill. Triggers on "SSR", "server-side rendering", "Inertia SSR", "hydration mismatch", "FancyClientOnly", "window/document is not defined", "first paint / flash of empty content", "renderToString", "hydrateRoot".
---

# SSR + hydration with Fancy UI

Fancy UI apps should render real content on the **first byte** — for crawlers, social/LLM scrapers, perceived speed, and a stable first paint — then hydrate to interactive. `@particle-academy/fancy-inertia` is built for this.

## The setup (Laravel + Inertia + Vite)

Two entries share `FancyAppRoot`:

```tsx
// resources/js/app.tsx — client
import { FancyAppRoot } from "@particle-academy/fancy-inertia";
createInertiaApp({ /* … */, setup: ({ el, App, props }) => FancyAppRoot.hydrate(el, App, props) });

// resources/js/ssr.tsx — server (Node)
import ReactDOMServer from "react-dom/server";
createInertiaApp({ page, render: ReactDOMServer.renderToString, /* … */ });
```

Build the SSR bundle (`vite build --ssr`) and run the Node SSR server (`php artisan inertia:start-ssr`). The Blade root still emits server-rendered SEO/meta on full loads — Inertia XHR visits return JSON, so bots get real HTML.

## The core rule: not everything can SSR

Anything that touches `window`, `document`, `ResizeObserver`, canvas/WebGL, or measures layout **cannot run on the server**. Wrap those in the SSR boundary instead of guarding by hand:

```tsx
import { FancyClientOnly } from "@particle-academy/fancy-inertia";

<FancyClientOnly fallback={<Skeleton />}>
  <Whiteboard … />   {/* canvas — client only; renders fallback on the server */}
</FancyClientOnly>
```

`FancyClientOnly` renders the fallback during SSR + the first hydration pass, then swaps in the real children — so the server HTML and the first client render **match**, avoiding hydration mismatches.

Good candidates to wrap: `fancy-whiteboard`, `fancy-artboard`, `fancy-3d*`, `fancy-flow`, anything from `fancy-motion`, and charts that measure their container.

## Gotchas (learned the hard way)

- **Floating/anchored positioning must re-measure when the node actually mounts — a passive effect alone isn't enough under SSR.** Popovers (Dropdown/Tooltip/Popover/Select) render through a `<Portal>` whose SSR-safe mount gate attaches the floating node *without re-rendering the component that positions it* — so a "measure after every commit" effect never sees that mount, and the element stays at its off-screen sentinel until an unrelated scroll fires a recompute ("menu only opens after I click and scroll"). The fix is an **rAF poll that waits for the node to land in the DOM, then a `ResizeObserver`** (react-fancy's `useFloatingPosition` does this since 4.4.6). If you build a custom anchored surface, don't assume a single post-commit measurement runs.
- **The duplicate `<title>` under SSR.** If you use the `fancy-seo` Blade baseline AND the client `<Seo>`, turning SSR on makes BOTH emit the head server-side → two `<title>`s / two of every tag in the first byte. Set `clientOnly: true` on the `<Seo>` provider so the Blade baseline owns the SSR head. See the `seo` skill's trap box. Verify: `curl -s <url> | grep -c '<title'` is `1`.
- **No `Date.now()` / `Math.random()` in render** — they differ between server and client → hydration mismatch. Compute them in an effect or pass them as props.
- **Read `window`/`localStorage` in effects only**, never in the render body or module top-level (the SSR bundle has no `window`).
- **Tailwind v4 cascade:** unlayered CSS beats `@layer utilities`. A server-rendered global like `section { padding: 96px 0 }` will override a `p-5` utility — scope globals or use a non-semantic element.
- **Hidden/offscreen tabs throttle rAF + CSS animations.** Don't rely on animation/rAF completion for correctness; drive state from effects/events.

## Deploying SSR (the three things that silently break it in prod)

SSR engaging in prod needs the bundle built, the daemon running, and the ports matched. Each fails quietly (empty `<div id="app">` + a graceful client-render fallback), so check all three:

1. **The deploy must build the SSR bundle.** A plain `npm run build` (just `vite build`) is client-only — the daemon then logs *"Inertia SSR bundle not found."* Make `build` run `vite build && vite build --ssr` (emits `bootstrap/ssr/ssr.js`), or run `npm run build:ssr` in the deploy. Pin `inertia.ssr.bundle => base_path('bootstrap/ssr/ssr.js')`.
2. **The Node daemon must run persistently** — `php artisan inertia:start-ssr` as a supervised process (e.g. a Forge **Daemon**), not started by hand. A deploy step of `php artisan inertia:stop-ssr` only *bounces* it (so it reloads the new bundle) IF something restarts it; if it's unsupervised, that line just kills SSR. The daemon loads the bundle **at startup**, so it must restart after every deploy.
3. **The daemon port must equal `config('inertia.ssr.url')`.** PHP reads the URL from **cached** config (`php artisan optimize`), while the daemon's Node process reads `INERTIA_SSR_PORT` from its own env — which, with config cached, often *isn't loaded* and falls back to a default. They drift → PHP dials one port, the daemon listens on another → *"can't connect"* + empty `#app`. **Hardcode the port on both sides** (`config/inertia.php` `ssr.url` AND `resources/js/ssr.tsx` `createFancyServer({ port })`) so they can't diverge. After changing the port, **restart the daemon** (it binds at startup; a running one stays on the old port and the deploy's `stop-ssr` can't even reach it to bounce it).

Diagnose with `php artisan inertia:check-ssr`, and check the daemon log says *"Inertia SSR server started on port N"* matching the config URL. The OG-image renderer (`spatie/browsershot` → headless Chrome) is unrelated to SSR; if its log shows `libatk-1.0.so.0: cannot open shared object file`, install Chrome's system libs on the server (`apt-get install libnss3 libatk1.0-0 libatk-bridge2.0-0 libcups2 libgbm1 libasound2 …`).

## Verify it actually rendered
Curl the page (not the browser) and confirm the real content is in the HTML, not just a `<div id="app">` shell. For visual correctness, capture the live URL with a headless screenshot and look at it — DOM presence ≠ correct layout.

For the data layer that pairs with SSR (hydrate the query cache from Inertia props so SSR'd data isn't refetched), see the `realtime` skill's `useInertiaHydration` note and the `building-apps` skill.
