---
name: building-apps
description: Use when building or scaffolding a whole application with Fancy UI — not dropping in one component, but choosing the stack, structuring a Laravel + Inertia + React 19 + Tailwind v4 app, deciding which Fancy packages to reach for (Fancy Core = react-fancy + fancy-inertia + fancy-query), and applying suite-wide best practices. Triggers on "build an app with fancy ui", "set up / scaffold a fancy ui project", "fancy core", "which fancy package should I use", "fancy ui best practices", "full-stack fancy app", "starter kit", "project structure for fancy".
---

# Building a complete app with Fancy UI

Fancy UI is a *suite*, not a widget grab-bag. Used to its fullest, it gives you a whole app: the chrome, the data layer, real-time, agent inhabitation, and ~100 primitives. This skill is the map.

## Fancy Core — the minimal stack for a real app

Three packages compose into everything else. Start here:

| Package | Role |
|---|---|
| `@particle-academy/react-fancy` | ~50 React + Tailwind v4 primitives (the visible UI) |
| `@particle-academy/fancy-inertia` | Laravel/Inertia bridge — `FancyAppRoot`, `useFancyForm`, `FancyClientOnly` (SSR boundary), schema-driven screens |
| `@particle-academy/fancy-query` | Server-state — TanStack Query with Inertia hydration + Echo-driven cache invalidation |

A normal web app needs nothing more than Fancy Core. Reach for the rest as features demand.

## Reach for the right package

- **Documents / data:** `fancy-sheets` (spreadsheet + headless formula engine), `fancy-slides` (decks), `fancy-code` (editor), `fancy-echarts` (charts), `holy-sheet` / `dark-slide` (server-side xlsx / pptx writers, agent-friendly).
- **Canvas / spatial:** `fancy-whiteboard`, `fancy-artboard`, `fancy-flow` (node graphs), `fancy-3d` (+ `-babylon` / `-three` engine adapters), `fancy-motion` (timeline/animation).
- **App shell / multi-screen:** `fancy-screens` (bring your own Zustand store), `fancy-cms-ui` + `fancy-cms` (WYSIWYG page building).
- **Agents in the running app:** `agent-integrations` (MCP bridges + presence) — see the `human-plus` skill.
- **Analytics:** `fancy-pixel` (badge + collector), `fancy-heuristics` (PHP ingestion), `fancy-heuristics-js`, `fancy-diff`.

Use the registry MCP (`list` / `search` / `get-install-instructions`) to confirm names + install commands — see the `components` skill. Don't guess slugs.

## Suite-wide best practices

1. **Controlled state, always.** Every stateful component takes `value` + `onChange`. Own the state in the host — never rely on internal-only state an agent or a server round-trip might need to touch. This is the Human+ contract (see `human-plus`).
2. **JSON-friendly props.** Arrays of objects, primitives, discriminated unions — so humans *and* agents can author them. Avoid forcing React children for data the agent must populate.
3. **Server state vs. UI state.** Put server-derived data behind `fancy-query` (cache, refetch, real-time invalidation). Keep ephemeral UI state local. Don't refetch what Inertia already shipped — hydrate it (`useInertiaHydration`, from `@particle-academy/fancy-query/inertia` since 0.5.0).
4. **SSR-first.** Render real content on the first byte; guard anything that touches `window`/measurement with `FancyClientOnly`. See the `ssr` skill.
5. **Real-time as invalidation.** Wire Laravel Echo to query invalidation rather than hand-rolling socket state. See the `realtime` skill.
6. **Tailwind v4 (CSS-first `@theme`), React 19.** Watch the cascade: unlayered CSS beats `@layer utilities`, so a stray `section { padding }` overrides `p-5`.
7. **Make it inhabitable.** If agents will share the surface, register an `agent-integrations` bridge from day one — retrofitting is harder. (`human-plus` skill.)

## Where each concern lives
- *Find/install a component* → `components` skill (registry MCP)
- *Make the running app agent-driveable* → `human-plus` skill
- *Server-side rendering + hydration* → `ssr` skill
- *Real-time / broadcasting* → `realtime` skill
