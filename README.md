# Fancy UI — Claude Code plugin

[![Fancy UI suite](art/fancy-ui.svg)](https://particle.academy)

Installs the **Fancy UI registry MCP** + skills for building with the
[Fancy UI](https://ui.particle.academy) kit — ~100 React + PHP UI primitives
engineered for **Human+ UX** (humans and AI agents sharing one UI surface).

## Install

```text
/plugin marketplace add Particle-Academy/fancy-ui-plugin
/plugin install fancy-ui@fancy-ui
```

Approve the `fancy-ui` MCP server when prompted. Claude can now browse, search,
and install Fancy UI components conversationally.

## What you get

- **MCP server** — the hosted registry at `https://ui.particle.academy/mcp`
  (streamable HTTP, `"type": "http"`). Tools to list, search, fetch, and get
  install instructions for every component.
- **Skills**
  - `fancy-ui:design` — pick a visual direction before building: browse the Inspiration Gallery (60 read-only blueprints in collections — fictional businesses/apps each designed 20 ways: the FIELDWORK studio site, the Mom-n-Pops food truck, and chart-driven Dashboards), grab or blend the recipes, and synthesize a design direction (tokens, component palette, layout).
  - `fancy-ui:components` — find, install, and compose Fancy UI components.
  - `fancy-ui:human-plus` — make a running app agent-inhabitable via MCP bridges.
  - `fancy-ui:building-apps` — build a full app with the suite (Fancy Core stack, which package to reach for, suite-wide best practices).
  - `fancy-ui:ssr` — server-side rendering + hydration with Inertia (`FancyClientOnly`, what can't SSR, mismatch gotchas).
  - `fancy-ui:seo` — crawlable / shareable / LLM-legible output: `<Seo>` + JSON-LD, the `fancy-seo` server baseline, sitemap / robots / llms.txt.
  - `fancy-ui:realtime` — real-time via Laravel Echo + `fancy-query` (broadcast → invalidate → refetch, presence).
  - `fancy-ui:update-fancy` — upgrade a project to the latest Fancy UI / Particle Academy packages (npm + Composer), review what changed, and adapt the code to breaking changes + new APIs.

## Why a plugin instead of a raw `.mcp.json`?

A per-project `.mcp.json` is easy to misconfigure — a missing `"type": "http"`
on a remote server silently fails to connect, and project configs aren't always
picked up. The plugin registers the server correctly and travels with the
install, across every project.

## Manual MCP config (if you'd rather not use the plugin)

```json
{
  "mcpServers": {
    "fancy-ui": {
      "type": "http",
      "url": "https://ui.particle.academy/mcp"
    }
  }
}
```

The `"type": "http"` line is required — remote streamable-HTTP MCP servers won't
connect without it.

## Links

- Site — https://ui.particle.academy
- MCP docs — https://ui.particle.academy/docs/mcp
- Components — https://ui.particle.academy/packages

## License

MIT © Particle Academy

---

## ⭐ Star Fancy UI

If this plugin or the Fancy UI packages are useful to you, a quick ⭐ on the repos really helps us build a better kit. Thank you!
