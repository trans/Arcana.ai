# Status

*Updated 2026-10-11*

Released: arcana 0.32.2 (`2af84ec`), arcana-core 0.16.0, arcana-ai 0.4.0.
All three are tagged with GitHub releases; arcana's packages build from the
tag (see [release.md](release.md)).

## Open

- **Fixed in `6eb9fb7` (0.32.1):** arcana now uses arcana-ai `~> 0.4.0`, so the
  bus chat services retry by default; `ChatAgent` no longer falls back to
  `"gpt-4o-mini"` for every provider; and a project that gets `.ai/` after
  its first registration keeps its owner token (moved from XDG state into
  `.ai/arcana.token`).
- **Fixed in `2af84ec` (0.32.2):** a daemon restart broke every running agent's
  arcana MCP tools until its session restarted (the bridge kept a dead
  keep-alive socket). Bridges from before 0.32.2 still do this, so restart
  agent sessions after upgrading.
- **Open, being fixed (the user approved 2026-10-11):** `Toolset` and
  `Service` handle one request at a time. Over a WebSocket `Client` the
  handler runs on the socket's read loop; in-process one fiber drains the
  mailbox. So a service's throughput is 1 / handler latency: @mj measured
  `mj:camera` at ~20 images/minute (4 concurrent ~2.8 s shots finished
  3.0–3.4 s apart), whatever provider pools it adds. Plan: each request in
  its own fiber, `max_concurrent:` (default 8, `nil` = unlimited), and
  `busy` = at least one request in flight (a counter). `ChatAgent` stays
  serial to keep each conversation in order. @mj offered to test with a
  reproducible 4-shot probe.
- **Not yet released:** `e1f3115` (2026-10-10) makes the session-end hook
  write unread mail to `.ai/mail/inbox/` as mail files; the installed
  0.32.2 hook still writes to `.ai/inbox/`.
- **Open:** TTS defaults to `gpt-4o-mini-tts`, which OpenAI's model list
  shows as deprecated (reported by @mj, 2026-10-06). It still works; no
  replacement chosen.
- **Open (the user is deciding):** retiring the `runware` toolset and the
  `tts` tool on `openai` from `bin/arcana.cr`; see
  [services.md](services.md).

Next feature: shared-log topics, see [topics.md](topics.md).
