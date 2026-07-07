# yeetable

> Throw element state at remote contexts. *Yeet-able.*

`cast-yee` projects a target's content to a secondary display via the Presentation API. That's one transport. `yeetable` is the general framework: a transport-agnostic projection interface, with cast as the first concrete transport and postMessage / popup / WebRTC / BroadcastChannel as the rest.

## Why

There's a recurring shape across the yee family:

- `cast-yee` — Presentation API
- `flightcomp` SW — page state projected to HTTP clients
- `macro-refid` SW bridge — DOM containers projected to a SOLID pod
- moq publishing — DOM state serialized into MoQ objects

Each is *"serialize this element (or subtree), ship it somewhere else, keep it in sync."* Each invents the same scaffolding: a transport interface, a mutation observer, a serialization format, a connection lifecycle. `yeetable` factors that out.

## The interface

```ts
interface YeetTransport {
  // open a connection to the remote context
  connect(): Promise<Connection>
  // optional: declare what serialization this transport supports
  formats?: SerializationFormat[]
}

interface Connection {
  // send a snapshot
  send(data: YeetPayload): void
  // stream of incoming messages from the remote
  messages(): AsyncIterable<unknown>
  // close the connection
  close(): Promise<void>
}

interface YeetPayload {
  format: SerializationFormat
  body: Uint8Array | string | object
  // metadata for merge-patch / diff-patch / OT
  patch?: unknown
}
```

The element side:

```html
<div id="content">
  <h1>Hello</h1>
</div>

<yeet source="#content" transport="cast">
  <yeet-format name="innerHTML"></yeet-format>
</yeet>
```

`<yeet>` watches `#content`, serializes mutations, and ships them via the named transport. Children declare what formats are offered.

## Transports

| Transport | Status | Notes |
|---|---|---|
| `cast` | delegate | Wraps `cast-yee` — Presentation API |
| `postmessage` | planned | iframe / window.postMessage target |
| `popup` | planned | `window.open` with BroadcastChannel pingback |
| `webrtc` | planned | DataChannel-based, peer-to-peer |
| `broadcastchannel` | planned | Same-origin multi-tab |
| `sharedworker` | planned | Project page state into a worker |
| `sw` | planned | Project into the service worker namespace (pairs with `flightcomp`, `macro-refid`) |

## Transports as a peer to yee

`<yeet>` is itself a yee applier: on `apply(target)` it sets up the observer and connection; on `unapply(target)` it tears them down (via [`yee-goeth`](https://github.com/rektide/yee-goeth) for graceful teardown). It also emits [`yee-there`](https://github.com/rektide/yee-there) events during connection setup so a `<yee-there-gate>` can wait for projection readiness.

So yeetable slots into the same lifecycle contracts as everything else in the family.

## Why "yeetable"

*Yeet* — modern slang for throwing something with abandon. The pun is the point: we are throwing element state at remote contexts, sometimes carefully, sometimes recklessly. The `-able` suffix matches the pattern of yee-there (an element that *can* signal its *thereness*).

## Status

Baseline. Interface sketch and transport registry only — no implementations. Work tracks under beads prefix `yeet`.

## Related

- [`cast-yee`](https://github.com/rektide/cast-yee) — first concrete transport.
- [`yee-there`](https://github.com/rektide/yee-there), [`yee-goeth`](https://github.com/rektide/yee-goeth) — lifecycle contracts yeetable adopts.
- [`flightcomp`](https://github.com/rektide/flightcomp), [`macro-refid`](https://github.com/rektide/macro-refid) — already doing projection ad-hoc; yeetable is the extracted interface.

## License

MIT
