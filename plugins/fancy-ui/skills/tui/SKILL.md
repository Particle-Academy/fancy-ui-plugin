---
name: tui
description: Build, redesign, or review terminal applications with @particle-academy/fancy-tui, Ink 7, React 19, and Human+ MCP event delivery. Use for TUI layouts, agent consoles, scrollback-safe chat/logs, multiline terminal input, ANSI Markdown/code, keyboard capability handling, responsive terminal UI, stable surface handles, or pushed and pull-based MCP workflows.
---

# Build with Fancy TUI

Use `@particle-academy/fancy-tui` for the terminal presentation layer and Ink's
Yoga renderer for layout. Pursue functional and workflow parity with browser
surfaces; do not imitate browser pixels, animation, or pointer-only interaction.

## Start

1. Inspect the host's Node, React, and Ink versions. Require Node 22+, React
   19.2+, and Ink 7.1+.
2. Install the published packages:

   ```bash
   npm install @particle-academy/fancy-tui ink react
   ```

3. Wrap the app in `FancyTuiProvider`. Add `TuiSurfaceProvider` with an explicit
   registry when agents need to inhabit the running surface.
4. Prefer package imports. Do not copy browser Fancy components or translate
   Tailwind classes into ad hoc terminal styling.

## Compose the surface

- Layout: `Screen`, `Box`, `Stack`, `Row`, `Column`, `Panel`, `Card`, `Header`,
  `StatusBar`, `Sidebar`, `Responsive`.
- Feedback: `Badge`, `Callout`, `Spinner`, `Progress`, `ActivityIndicator`,
  `Timeline`, `ToolCall`.
- Input: controlled `Input`, `MultilineInput`, `Composer`, choices, switches,
  selectors, and `Command`.
- Data/content: `Table`, `TreeNav`, `FileBrowser`, `Markdown`, `CodeView`.
- Agent conversation: commit completed entries through `MessageList`/`StaticList`;
  render mutable thinking and tool status in `LiveRegion` below it.

Keep all agent-readable state controlled. Give every interactive component a
stable, meaningful `id`.

## Handle terminal constraints

- Treat terminal width as dynamic. Use `Responsive` and test narrow and wide
  layouts; never assume fixed columns.
- Use `Alt+Enter` as the universal multiline fallback. Enable Shift+Enter only
  when enhanced keyboard capability detection confirms a distinct sequence.
- Preserve grapheme-aware cursor movement. Do not split emoji or combining
  characters with manual string indexing.
- Emit ANSI deliberately and sanitize untrusted Markdown/content.
- Keep committed output separate from live repainting to protect scrollback.

## Add Human+ MCP

Install `@particle-academy/agent-integrations` and register the TUI registry:

```ts
const registry = createTuiSurfaceRegistry();
registerTuiBridge(server, { registry, eventStore });
```

Use a durable event store for real applications. The bridge must:

- persist an event before sending `notifications/human_plus/event`;
- retain the same event for `human_plus_events_list`;
- acknowledge per consumer, with at-least-once delivery;
- emit observable `AgentActivity`;
- stage destructive or human-visible actions for confirm/reject.

Agents act through `tui_surface_read` and `tui_action_invoke`, never terminal
screen scraping or synthetic keystrokes.

## Verify

- Test reducers, registry behavior, duplicate handles, and keyboard capability
  branches without a terminal.
- Render representative components with `ink-testing-library`.
- Assert committed messages and the live region stay structurally separate.
- Exercise resize/reflow and a non-interactive smoke mount.
- Run test, typecheck, build, and audit before publishing.

For visual comparison and copyable examples, use the Fancy UI sandbox at
`/fancy-tui`, which renders shared demos as both HTML and ANSI console surfaces.
