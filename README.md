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
  - `fancy-ui:components` — find, install, and compose Fancy UI components.
  - `fancy-ui:human-plus` — make a running app agent-inhabitable via MCP bridges.

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
