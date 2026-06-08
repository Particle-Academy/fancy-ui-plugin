---
name: realtime
description: Use when adding real-time / live updates to a Fancy UI + Laravel app — Laravel Echo / broadcasting (Reverb, Pusher), live cache invalidation, presence, or syncing server state across clients and agents. Triggers on "Echo", "broadcasting", "real-time", "live updates", "websocket", "Reverb", "Pusher", "presence channel", "invalidate on event", "fancy-query".
---

# Real-time with Echo + fancy-query

The Fancy way to do real-time is **broadcast → invalidate → refetch**, not hand-rolled socket state. `@particle-academy/fancy-query` (a TanStack Query wrapper) wires Laravel Echo events to query-cache invalidation, so a server broadcast simply marks the affected queries stale and they refetch with fresh, authoritative data.

## Wiring (client)

```tsx
import {
  FancyDataRoot, useInertiaHydration,
  useFancyQuery, useFancyEchoInvalidation,
} from "@particle-academy/fancy-query";

// 1. Provide the QueryClient + Echo client once near the root.
<FancyDataRoot echo={echo}>…</FancyDataRoot>

function Board({ boardId }) {
  // 2. Seed the cache from Inertia page props — no refetch flash on first load,
  //    and SSR'd data is reused.
  useInertiaHydration();

  // 3. Read server state through a query.
  const { data } = useFancyQuery(["board", boardId], () => api.board(boardId));

  // 4. When the server broadcasts, invalidate the matching keys → refetch.
  useFancyEchoInvalidation(`board.${boardId}`, {
    "BoardUpdated": ["board", boardId],
    "ItemMoved":    ["board", boardId],
  });

  return <Board data={data} … />;
}
```

`useEchoClient()` exposes the raw Echo client if you need presence channels or whisper/typing events directly.

## Wiring (server — Laravel)

1. Run a broadcaster — **Reverb** (first-party, self-hosted) or Pusher. Set `BROADCAST_CONNECTION`.
2. Make domain events broadcast: implement `ShouldBroadcast`, `broadcastOn()` returns the channel (`new Channel("board.{$id}")` or `PrivateChannel` / `PresenceChannel`), `broadcastAs()` names the event the client listens for (`"BoardUpdated"`).
3. Fire the event after the write — the client's `useFancyEchoInvalidation` refetches authoritative state. **Broadcast the fact, not the whole payload**; let the query refetch be the source of truth (avoids divergence + ordering bugs).

## Presence (humans *and* agents)

- Use a **presence channel** for who's-here / cursors. Authorize it server-side in `routes/channels.php`.
- For agents sharing the surface, presence + activity flow through `agent-integrations` (it broadcasts `AgentActivity` events). Real-time human presence (Echo) and agent presence (bridges) compose — see the `human-plus` skill.

## Best practices

- **Invalidate, don't patch.** Echo events should trigger a refetch, not mutate the cache by hand — the server stays authoritative and you avoid drift.
- **Scope channels tightly** (`board.{id}`, not a firehose) so a client only wakes for relevant changes.
- **Optimistic UI** via `useFancyMutation` for the actor; the Echo invalidation reconciles everyone (including the actor) to server truth.
- **Pair with SSR:** `useInertiaHydration` means the first render uses server data; Echo keeps it live after. See the `ssr` skill.
