---
name: human-plus
description: Use when making a Fancy UI surface agent-driveable — building an app where humans AND AI agents share the same UI and trade control over MCP bridges (agent-integrations), wiring registerWhiteboardBridge / registerFlowBridge / registerSheetsBridge / registerSlidesBridge / registerFormBridge / etc., adding agent presence + cursors, satisfying the Human+ UX component contract (controlled state, stable handles, bridgeable surface), or generating per-client install affordances with ConnectorButtons / writeMcpbBundle. Triggers on "let an agent drive/control the UI", "agent shares the UI", "MCP bridge", "agent presence", "human plus", "human+ ux", "agent-integrations", "add to claude / cursor button", ".mcpb bundle".
---

# Human+ UX: make the app agent-inhabitable

Fancy UI's thesis is **Human+ UX** — running apps where agents are first-class participants that read/write component state through MCP tool bridges and stable handles, **not** Playwright or DOM scraping.

## The component contract
Every stateful or interactive component must satisfy all of:
- **Controlled state** — `value` + `onChange`; no internal-only state an agent might need to read or write.
- **Stable handles** — each interactive element has a stable identity (`id`, `data-*`, or a selector prop). Agents never guess at the DOM.
- **JSON-friendly inputs** — arrays of objects, primitives, simple discriminated unions.
- **Bridgeable surface** — a `register<Surface>Bridge(server, { adapter })` exposes MCP tools.
- **Observable activity** — mutations broadcast `AgentActivity` events so presence / undo / coaching layers compose for free.
- **Trust-but-verify** — destructive or human-visible actions support a staged-write / pending mode: agents propose, humans confirm.

## Wiring a bridge (`@particle-academy/agent-integrations`)
1. Make the component controlled (most `fancy-*` components already are).
2. `import { registerXBridge } from "@particle-academy/agent-integrations/bridges/X"`.
3. Provide an **adapter** — host getters/setters over the surface's state (often `setState(s => reduceX(s, op))`).
4. Register against a per-session MCP server (`MicroMcpServer`). The bridge wraps every mutation with activity broadcasts + an undo stack.
5. Mount the presence/activity layer once near the root so events flow into the UI.

Existing bridges (tool prefixes): whiteboard (`whiteboard_*`), artboard, flow (`flow_*`), form (`form_*`), sheets (`sheet_*`), code (`code_*`), charts (`chart_*`), scene (`scene_*`), screens (`screen_*`), slides (`deck_*` / `slide_*` / `element_*`) — plus cross-cutting `agent_undo` / `agent_redo` / `agent_history`.

## Authoring vs. inhabiting — keep them straight
- The **registry MCP** that ships with this plugin is **install-time only** — it helps you *find and install* components.
- To let agents drive a **running** app, register the **agent-integrations** bridges inside the host app. They are different surfaces.
