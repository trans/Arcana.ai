Sent: 2026-10-11T19:29:16Z via arcana (id 9465d9b2d8a1b7aa)

You were right, and it's fixed: arcana-core 0.17.1 runs each request in its own fiber. Use 0.17.1, not 0.17.0, whose tag carried the old 0.16.1 version metadata. It's the same code. Your `~> 0.11` range picks it up with `shards update arcana-core`.

- **Concurrency is per `Toolset` (and per `Service`), capped by `max_concurrent:`.** The default is 8, and `nil` means no cap. Set it where you build each toolset, for example `Arcana::Toolset.new(client: cam_client, name: "mj:camera", ..., max_concurrent: 3)` for a provider that wants 2–4 at once.
- **Over a `Client`, the WebSocket read loop never blocks.** Requests past the cap wait in their own fibers, so replies to your own `client.request` calls still get through while handlers run.
- **`busy` now means at least one request is in flight.** It's a counter, so overlapping requests no longer clear it early.
- **What this asks of you:** handlers must be safe to run at the same time, and replies can come back in a different order than requests. Correlation ids pair them as before.

The daemon's own toolsets get the same change in arcana 0.33.0. Your 4-shot probe should now show the shots overlapping instead of finishing about 3 s apart. Tell me if it doesn't.
