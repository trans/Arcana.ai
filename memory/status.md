# Status

*Updated 2026-10-11*

Released: arcana 0.33.0 (`0d7b900`), arcana-core 0.17.1 (`443b63b`), arcana-ai 0.4.0.
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
- **Fixed in arcana-core 0.17.1 (`443b63b`), arcana 0.33.0 (`0d7b900`):**
  `Toolset` and `Service` took one request at a time (1 / handler
  latency; @mj measured ~20 images/minute from `mj:camera`). Each request
  now runs in its own fiber, at most `max_concurrent` at once (default 8,
  `nil` = no cap; the user approved 2026-10-11); `busy` = at least one in
  flight. `ChatAgent` stays serial. Told @mj, who offered to verify with a
  4-shot probe (see `mail/sent/`). **Confirmed by @mj 2026-10-11:** wall time 13.0 s →
  3.8 s; `mj:camera` went from ~20 to ~130 images/minute. @mj also found
  (a) a per-Toolset cap is too coarse for a service fronting several
  independent backends (mj limits per provider pool inside the handler,
  with `max_concurrent: 16` as a backstop), and (b) handlers with shared
  mutable state need a lock now: mj's append-only spend ledger could
  interleave mid-line. Arcana's own calls (the user left them to this
  project's agent, 2026-10-11): both points are in the `Toolset`/`Service`
  docs (arcana-core `9ed6730`, not yet in a release), and the daemon's
  `runware` toolset caps at 3 (`b78fff2`, ships with the next arcana
  release). Nobody else uses `Toolset`/`Service`: memo, merch's
  shoppe-service and Copious handle bus requests their own way.
- **Released in 0.33.0:** the session-end hook writes unread mail to
  `.ai/mail/inbox/` (`e1f3115`), and this `.ai/` moved to the `ai` branch.
- **Open:** TTS defaults to `gpt-4o-mini-tts`, which OpenAI's model list
  shows as deprecated (reported by @mj, 2026-10-06). It still works; no
  replacement chosen.
- **Open (the user is deciding):** retiring the `runware` toolset and the
  `tts` tool on `openai` from `bin/arcana.cr`; see
  [services.md](services.md).

Next feature: shared-log topics, see [topics.md](topics.md).
