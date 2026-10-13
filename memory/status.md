# Status

*Updated 2026-10-13*

Released: arcana 0.34.0 (`cf892fa`), arcana-core 0.18.0 (`71adad6`), arcana-ai 0.5.0 (`30ffecf`).
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
  interleave mid-line. **Answered in arcana-core 0.18.0** (the user
  approved, 2026-10-11): lanes (a cap per backend; a request waits for its
  lane before taking a toolset slot) and `ctx.exclusive(:key) { }` (one
  block at a time per key, the rest of the handler concurrent); the README
  has a "Writing a service" section. arcana 0.34.0 depends on it and caps
  the daemon's `runware` toolset at 3 (installed 2026-10-12). Told @mj
  (see `mail/sent/`). **@mj adopted lanes (2026-10-12)** and verified them:
  two requests for a free backend cleared in ~3.5 s behind eight queued
  for a capped one. They kept their ledger's own Mutex instead of
  `ctx.exclusive`, rightly: the mj CLI writes it too. The docs now say
  `exclusive` is for state the service owns (arcana-core `a6906a3`). Nobody else uses `Toolset`/`Service`: memo,
  merch's shoppe-service and Copious handle bus requests their own way.
- **Released in 0.33.0:** the session-end hook writes unread mail to
  `.ai/mail/inbox/` (`e1f3115`), and this `.ai/` moved to the `ai` branch.
- **Released in arcana-ai 0.5.0 (`9158641`, 2026-10-12): usage and pricing**,
  asked for by @onboard and wanted by the user, so every project prices
  calls the same way. `Usage` evens out the providers' token reports;
  `response.cost` is a `Pricing::Cost` (an exact BigDecimal at list
  rates, no estimate flag: the user said a price that's known for sure
  needn't be marked) or `Pricing::Unpriced` with a reason. Rates live in
  `src/arcana-ai/pricing/rates.yml`, checked against the Anthropic,
  OpenAI, Gemini and Voyage pricing pages on 2026-10-12. Grok and
  DeepSeek have no rates yet (their calls come back Unpriced). The same
  release fixes OpenAI streams, which never recorded usage, and the
  Anthropic default, retired on 2026-06-15 (now `claude-sonnet-5-5`).
  Replied to @onboard with the API and a migration path off its old arcana
  0.1.0 snapshot (see `mail/sent/`); Onboard's own fixes and its billing
  reconciliation are its call.
  - **Released in arcana 0.35.0 (2026-10-13):** the daemon on arcana-ai
    0.5.1; the chat and embed tools reply with `usage` and `cost` (USD,
    list price) or `unpriced`; the anthropic chat default is current.
    Needs the user's install.
  - **Deprecating the old token counts** (the user wants them gone,
    2026-10-13): arcana-ai 0.5.1 warns at compile time on
    `Chat::Response#prompt_tokens`, `completion_tokens`,
    `cache_read_tokens`, `cache_creation_tokens`, `reasoning_tokens` and
    `Embed::Result#total_tokens`; arcana 0.35.0's replies carry a
    `deprecated` note on prompt_tokens, completion_tokens,
    reasoning_tokens and total_tokens. **Removal** (arcana-ai 0.6.0,
    arcana 0.36.0) waits until the known users have moved:
    - @onboard (library) and @memo (library and bus embed replies):
      told 2026-10-13, see `mail/sent/`. **Waiting** for each to confirm.
    - @overlore: **moved** (2026-10-13, its ledger reads `usage`; lands
      in its next commit). It needn't hold up the removals.
    - goldfish (library) and likely (bus chat replies) have no agent on
      the bus: the user to decide how they're moved.
  - **Assessed 2026-10-13, not started:** pricing for images and TTS
    (official sources, checked that day).
    - Runware images: already exact; Runware reports the USD deducted
      (`includeCost`).
    - OpenAI images: exact from the response's `usage` (text/image input
      tokens, output tokens) times list rates. gpt-image-1 shuts down
      **2026-10-23** (arcana-ai's default; @overlore told, and is
      unaffected: it draws through mj:camera unless its opt-in OpenAI
      backend is chosen, which the user will settle); gpt-image-1.5,
      -1-mini and chatgpt-image-latest on 2026-12-01; gpt-image-2 and
      gpt-image-2.5-sunburst/-flare are current ($5 text in, $8 image in,
      $30 image out per M). `Image::Result#cost` is a Float64 today
      (Runware's), so the Cost shape needs a deprecation.
    - OpenAI speech: every `/v1/audio/speech` model (tts-1, tts-1-hd,
      gpt-4o-mini-tts) shuts down **2027-01-06**; OpenAI points to a
      Realtime API model, which arcana-ai has no client for. tts-1/-hd
      bill per character (counting rule undocumented); gpt-4o-mini-tts
      reports usage only with `stream_format: "sse"`.
    - ElevenLabs: the `character-cost` header gives billed characters,
      but dollars per character depend on the account (plan rates, voice
      multipliers, promos), so USD needs the account's rate. Sound-effect
      billing rules in the docs don't agree with each other.
  - **Asked by @overlore (2026-10-13):** price speech and OpenAI images in
    arcana-ai's table, so Overlore and mj drop their own (the user wants
    prices in one place). Overlore's speech figure is an estimate from
    text length; its OpenAI images are logged at $0. Not yet answered.
  - **Open:** TTS pricing, to add with @mj (which prices TTS today).
    gpt-4o-mini-tts bills tokens ($0.60/M text in, $12/M audio out), and
    OpenAI reports usage only in the speech endpoint's SSE stream.
  - **OpenAI speech shutdown, 2027-01-06** (deprecations page, entry
    "2026-10-01: Text-to-speech models"): tts-1, tts-1-hd, both
    gpt-4o-mini-tts snapshots. Replacement gpt-realtime-2.1-mini is
    Realtime-only (`/v1/realtime`). Told @mj, @onboard, @overlore and
    soaip (by its `.ai/mail/inbox`, which the user said to create) on
    2026-10-13, asking whether they'd use a Realtime speech client in
    arcana-ai; their answers decide whether to build one. The daemon's
    `openai` tts tool stops too.
  - **Upkeep:** the table needs updating when providers change prices.
    Known dates: gpt-5.6-sol's promotional rate is promised only through
    2026-11-21, so calls after that come back Unpriced until it's
    re-checked. Gemini 3.x Flash's 2027-01-01 rise is already in.
- **Open:** TTS defaults to `gpt-4o-mini-tts`, which OpenAI's model list
  shows as deprecated (reported by @mj, 2026-10-06). It still works; no
  replacement chosen.
- **Open (the user is deciding):** retiring the `runware` toolset and the
  `tts` tool on `openai` from `bin/arcana.cr`; see
  [services.md](services.md).

Next feature: shared-log topics, see [topics.md](topics.md).
