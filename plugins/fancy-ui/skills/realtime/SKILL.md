---
name: realtime
description: Use when adding real-time / live updates to a Fancy UI + Laravel app — Laravel Echo / broadcasting (Reverb, Pusher), live cache invalidation, presence, or syncing server state across clients and agents. Triggers on "Echo", "broadcasting", "real-time", "live updates", "websocket", "Reverb", "Pusher", "presence channel", "invalidate on event", "fancy-query", "streaming chat", "useFancyStream".
---

# Real-time with Echo + fancy-query

The Fancy way to do real-time is **broadcast → invalidate → refetch**, not hand-rolled socket state. `@particle-academy/fancy-query` (a TanStack Query wrapper) wires Laravel Echo events to query-cache invalidation, so a server broadcast simply marks the affected queries stale and they refetch with fresh, authoritative data.

## Wiring (client)

```tsx
import {
  FancyDataRoot,
  useFancyQuery, useFancyEchoInvalidation,
} from "@particle-academy/fancy-query";
// Inertia-only hook — own subpath (since fancy-query 0.5.0) so non-Inertia
// apps never touch the optional @inertiajs/react peer.
import { useInertiaHydration } from "@particle-academy/fancy-query/inertia";

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

## Streaming — patch the cache instead of invalidating

Invalidate-and-refetch is right for **list** surfaces. For **chat / agentic / token-stream** surfaces it's the wrong tool: a refetch drops optimistic + in-flight state and flickers. `useFancyStream` maps Echo events onto `setQueryData` **reducers** so the cache is patched in place:

```tsx
import { FancyDataRoot, useFancyStream } from "@particle-academy/fancy-query";

const { data: messages, isStreaming, append } = useFancyStream(["chat", chatId], {
  channel: `private-chat.${chatId}`,
  fetchInitial: () => api.get(`/api/chat/${chatId}/history`),
  on: {
    "post.created":     (cache, e) => [...(cache ?? []), e.post],   // append
    "post.delta":       (cache, e) => patchLast(cache, e.delta),    // token-stream
    "stream.completed": (cache, e) => reconcile(cache, e),          // settle
  },
  poll: { while: "streaming", intervalMs: 4000 },                   // missed-broadcast recovery
});

// Optimistically show the user's own message before the server echoes it back:
const send = (text) => { append({ id: tempId(), text, pending: true }); api.post(/* … */); };
```

`isStreaming` flips on the start/end events (default `stream.started` / `stream.completed`). Same channel-prefix rules + `FancyDataRoot` wiring as `useFancyEchoInvalidation`; the consumer still owns the Echo connection. Reach for this when a refetch would clobber state you're mid-way through — otherwise prefer invalidation below.

## Wiring (server — Laravel)

1. Run a broadcaster — **Reverb** (first-party, self-hosted) or Pusher. Set `BROADCAST_CONNECTION`.
2. Make domain events broadcast: implement `ShouldBroadcast`, `broadcastOn()` returns the channel (`new Channel("board.{$id}")` or `PrivateChannel` / `PresenceChannel`), `broadcastAs()` names the event the client listens for (`"BoardUpdated"`).
3. Fire the event after the write — the client's `useFancyEchoInvalidation` refetches authoritative state. **Broadcast the fact, not the whole payload**; let the query refetch be the source of truth (avoids divergence + ordering bugs).

## Presence (humans *and* agents)

- Use a **presence channel** for who's-here / cursors. Authorize it server-side in `routes/channels.php`.
- For agents sharing the surface, presence + activity flow through `agent-integrations` (it broadcasts `AgentActivity` events). Real-time human presence (Echo) and agent presence (bridges) compose — see the `human-plus` skill.

## Best practices

- **Invalidate for lists; patch for streams.** Default to invalidate-and-refetch — the server stays authoritative and you avoid drift. Reach for `useFancyStream`'s in-place reducers only on streaming/chat surfaces where a refetch would drop optimistic + in-flight state.
- **Scope channels tightly** (`board.{id}`, not a firehose) so a client only wakes for relevant changes.
- **Optimistic UI** via `useFancyMutation` for the actor; the Echo invalidation reconciles everyone (including the actor) to server truth.
- **Pair with SSR:** `useInertiaHydration` means the first render uses server data; Echo keeps it live after. See the `ssr` skill.
