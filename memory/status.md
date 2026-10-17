# Status

*Updated 2026-10-17*

Released: arcana 0.35.4 (`b4b461a`, 2026-10-14, with 0.35.1–0.35.3),
arcana-core 0.19.0 (`c762da7`), arcana-ai 0.5.3 (`627c638`). Installed:
arcana 0.35.4 (the daemon restarted on it). Committed, not tagged or
pushed: arcana 0.35.5 (`b1bd0b4`: `arcana_deliver` files the sent copy,
a session's project is the nearest `.ai/` (through an `.ai` link too),
and the Stop fix).
Sessions started before an install keep the old MCP bridge until they
restart.
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
    **Installed** (the daemon restarted 2026-10-13 01:37 local on the new
    binary); a live `openai` chat reply carried `usage`, `cost` and the
    `deprecated` note.
  - **Deprecating the old token counts** (the user wants them gone,
    2026-10-13): arcana-ai 0.5.1 warns at compile time on
    `Chat::Response#prompt_tokens`, `completion_tokens`,
    `cache_read_tokens`, `cache_creation_tokens`, `reasoning_tokens` and
    `Embed::Result#total_tokens`; arcana 0.35.0's replies carry a
    `deprecated` note on prompt_tokens, completion_tokens,
    reasoning_tokens and total_tokens. **Removal** (arcana-ai 0.6.0,
    arcana 0.36.0) waits until the known users have moved:
    - @onboard (library): told 2026-10-13, see `mail/sent/`. **Waiting**
      for it to confirm; the only one left.
    - @memo: **moved** in memo 0.13.1 (released 2026-10-14; reads
      `usage.input`, needs arcana-ai ~> 0.5.1). memo 0.13.0 is pinned to
      arcana-ai ~> 0.4.0 and treats a missing `total_tokens` as 0, so it
      doesn't hold up the removals either (see `mail/received/`).
    - @overlore: **moved** (2026-10-13, its ledger reads `usage`; lands
      in its next commit). It needn't hold up the removals.
    - goldfish (library) and likely (bus chat replies), which have no
      agent: **moved** by Arcana with the user's go-ahead (2026-10-13),
      committed in their repos, not pushed. goldfish `3c4d9bd` reads
      `usage` (lock 0.1.0 → arcana-ai 0.5.3; its specs need
      `KEMAL_ENV=test`, or `Kemal.run` keeps the run open). likely
      `c7bc25f` reads `usage`, falling back to the old fields for daemons
      before 0.35.0.
  - **Released in arcana-ai 0.5.2 (`8e8399d`, 2026-10-13): image and
    speech pricing**, asked for by @overlore so apps drop their own rates
    (the user wants prices in one place). Answered @overlore and told @mj
    (see `mail/sent/`).
    - `Usage` gained `image_input`, `image_output`, `audio_output`; the
      table has gpt-image-2, gpt-image-2.5-sunburst/-flare ($5 text in,
      $8 image in, $30 image out per M) and gpt-4o-mini-tts ($0.60 in,
      $12 audio out). The older GPT Image models (1, 1-mini, 1.5,
      chatgpt-image-latest) are left out on purpose: they have cached
      rates their usage never reports, so they're Unpriced. Cached rates
      never apply to 2/2.5 through the Images API (OpenAI's guide).
    - Runware's reported USD is a `Pricing::Cost` (`Cost.reported`, as
      `reported_cost`). `Image::Result#cost` as Float64, and building one
      with a Float64, are deprecated; `cost(at)` returns Cost | Unpriced,
      and plain `cost` will in 0.6.0. Overlore builds with two warnings
      pointing at its two uses; mj, soaip and Onboard build clean.
    - `TTS::OpenAI` asks every model but tts-1/-hd for `stream_format:
      "sse"`, the only way OpenAI reports speech usage (checked live);
      tts-1/-hd stay binary and Unpriced (per-character, counting rule
      undocumented).
    - The OpenAI image default moved from gpt-image-1 (shuts down
      2026-10-23) to gpt-image-2.5-sunburst. Overlore draws through
      mj:camera, so it's unaffected unless its opt-in OpenAI backend is
      chosen (the user will settle that).
    - **Waiting on @mj: ElevenLabs.** Its API price page (2026-10-13)
      lists one rate per model for every plan: $0.08/1K characters for
      eleven_v4, eleven_v3, eleven_multilingual_v2; $0.04 for
      eleven_v4_turbo, eleven_v3_conversational, Flash/Turbo. The
      `character-cost` header's unit is undocumented (characters or
      credits; Flash is half a credit per character), so it can't be
      priced until one call is checked. Asked @mj (no ElevenLabs key
      here). Sound effects stay unpriced: the docs say both $0.12 a minute
      and "billed per generation".
    - **Released in arcana 0.35.1 (`45e08c7`, 2026-10-14):** the
      daemon on arcana-ai 0.5.3; its `tts` reply carries `usage` and
      `cost`, its `runware` image reply takes `cost` from `cost(at)` (or
      says `unpriced`), and its `openai` chat works with current models
      (see below). A permission check blocked Arcana's push, so the user
      pushed main and the tags for 0.35.1, 0.35.2 (topics,
      [topics.md](topics.md)) and 0.35.3 (dot-ai 0.3, below), and Arcana
      made the GitHub releases. 0.35.3 installed 2026-10-14.
  - **Fixed in arcana-ai 0.5.3 (`e361d64`, 2026-10-13): OpenAI chat on
    gpt-5 and later.** The o-series and every model from gpt-5 on
    rejected `max_tokens`, and most reject any temperature but their
    default (1), so only GPT-4/3.5 models worked without thinking (found
    checking the 0.35.0 install). OpenAI now gets `max_completion_tokens`
    for every model (checked live, 4o-mini through gpt-6-luna and
    o4-mini); the request's default 0.7 goes only to `gpt-4*`/`gpt-3.5*`
    (and `ft:` of those), a caller's own value always goes; Grok and
    DeepSeek unchanged. The daemon's openai `max_tokens` default went 150
    → 4096 (`f45a39a`): reasoning counts against it. Only Overlore's
    banter calls OpenAI chat through the library (on gpt-4o-mini, never
    affected); told @overlore (see `mail/sent/`). **For 0.6.0:** make
    `Request#temperature` nilable, since an explicit 0.7 can't be told
    from the default (Anthropic's client has the same limit).
  - **OpenAI speech shutdown, 2027-01-06** (deprecations page, entry
    "2026-10-01: Text-to-speech models"): tts-1, tts-1-hd, both
    gpt-4o-mini-tts snapshots. Replacement gpt-realtime-2.1-mini is
    Realtime-only (`/v1/realtime`). Told @mj, @onboard, @overlore and
    soaip (by its `.ai/mail/inbox`, which the user said to create) on
    2026-10-13, asking whether they'd use a Realtime speech client in
    arcana-ai; their answers decide whether to build one. **@overlore:
    no** (the user decided 2026-10-13 that Overlore moves all its speech
    to ElevenLabs through mj:voice; it now prices speech and pictures
    with arcana-ai 0.5.2, and wants ElevenLabs prices once @mj's check
    settles `character-cost`). Waiting on @mj, @onboard, soaip. The
    daemon's `openai` tts tool stops too.
  - **Upkeep:** the table needs updating when providers change prices.
    Known dates: gpt-5.6-sol's promotional rate is promised only through
    2026-11-21, so calls after that come back Unpriced until it's
    re-checked. Gemini 3.x Flash's 2027-01-01 rise is already in.
- **Open:** TTS defaults to `gpt-4o-mini-tts`, which OpenAI shuts down on
  2027-01-06 with every other speech-endpoint model (see above). No
  replacement chosen; it depends on whether anyone wants a Realtime client.
- **Open (the user is deciding):** retiring the `runware` toolset and the
  `tts` tool on `openai` from `bin/arcana.cr`; see
  [services.md](services.md).

- **Done: dot-ai draft 0.3.** arcana 0.35.3 (`43a28ed`, released
  2026-10-14) reads the top-level `handle` and `description` (and still
  the old `arcana:` keys when those are missing, Arcana's call) and names
  inbox files `<UTC time>-<sender>--<topic>-<id8>.md`. The user ran the
  upgrade script over every project and installed the 0.3 skill
  (2026-10-14; this branch's commit `c7dc13a`). Before that, at the
  user's request, Arcana moved six projects' pre-`mail/` `.ai/inbox/`
  files into `.ai/mail/inbox/` (cyclops, likely, onboard,
  siliconcircus.lol, guestservices, wow; none had `.ai/` in git), unread,
  for their agents.
- **Released and installed in arcana 0.35.4 (`cd9852e`, 2026-10-14):
  `arcana_receive` writes the inbox file**, as @ipck asked (the user's
  rule since dot.ai `86524e5`: every message between projects is
  recorded on both sides). The bridge writes each message from an `@`
  address to the project's own handle into `.ai/mail/inbox/` before
  returning it, names it in `inbox_file` (or `inbox_error`), and skips
  topic notices and service replies. Both paths now write `Via: arcana`.
  A session whose bridge predates 0.35.4 still writes the file itself
  before acting on a received message.
- **dot-ai-mail and Arcana: layers, no switch** (the user decided,
  2026-10-15). dot-ai-mail always does local mail, with or without
  Arcana; Arcana does what's above it: other machines, cloud agents,
  live sessions, services. No setting chooses between them. dot-ai is
  now one plugin for Claude Code and Codex (`trans/dot-ai`, `95c1c9a`):
  the skill (`dot-ai:dot-ai` in Claude Code) plus hooks that run
  dot-ai-mail 0.1.0 at session start, Stop and session end. The user
  installed it on 2026-10-16 (@ipck's mail, `mail/received/`).
- **Removed (2026-10-17): `arcana hook drain`.** The user meant
  dot-ai-mail as light mail for one machine without Arcana, so it never
  calls Arcana, and the drain had no caller. It came from Arcana's own
  one-voice idea (2026-10-14); @ipck's note said the user had asked for
  it, and Arcana built it without checking. Each tool announces its own
  place. Main was rewound to 0.35.4 (the drain was never pushed), and its
  one fix worth keeping went back in on its own: Stop nudges once per
  change in the count (`fc4a853`).
- **Decided (2026-10-17): mail keeps its record.** `sent/` and
  `received/` stay, on @ipck's case, which the user agreed with. For
  most sessions they're the only copy (Claude Code and Codex don't commit
  transcripts). They hold one message per file, word for word, and
  record both sides of what two projects said. In Copious there's no
  duplication: a transcript keeps a pinned view's path, not the file's
  text. An agent without a checkout needs no mail files; Arcana carries
  the mail.
- **Built (2026-10-17), not released: `arcana_deliver` files the sender's
  copy** (the user decided, 2026-10-17; `0521bc2`, 0.35.5). It's the
  mirror of `arcana_receive` writing the inbox file. Each message from
  the project's own handle to an agent that the bus took goes into
  `.ai/mail/sent/` as `<time>-<recipient>--<topic>.md` with `Sent: <time>
  via arcana`, named in `sent_file`. Without a subject, the topic is the
  correlation id's first 8; a clash in one second gets `-2`. Services,
  other senders, failed deliveries and projects without `mail/sent/` get
  no copy. Still open: an outbox message dot-ai-mail can't deliver (its
  recipient isn't on this machine) gets a second copy under a new name
  when the agent sends it by hand. Sending the file itself (`arcana send
  FILE`) would fix that; not built.
- **Decided (the user, 2026-10-17): a session's project is the nearest
  `.ai/` at or above it**, in Arcana as in dot-ai (dot.ai `de268ee`).
  With no `.ai/` anywhere above, the session's own directory keeps the
  fallback handle (`@` plus its name, token under XDG state), so Arcana
  still works without dot-ai; the user chose that over not registering.
  **Built in `1cb0528` (0.35.5, not released):** `AgentConfig.project_root`
  and `find` walk up, resolving symbolic links as dot-ai-mail does, for
  the hooks and the bridge. The fallback handles registered from
  directories without `.ai/` (`@pirateship`, `@tabcomputing-com` on
  2026-10-17) stop being refreshed and expire.
  - **Decided (the user, 2026-10-17): arcana, arcana-core and arcana-ai
    are one project.** They share a plain folder whose `.ai` links to
    arcana's `.ai/` (a link, not a setting; `~/Projects/Arcana/` on this
    machine), so a session in any of them is
    `@arcana`. The worktree uses relative paths now. Session start warns
    of a broken link (`93c2140`). The move itself waits for the user, with
    no sessions open in the three. Steps and later portability work:
    [layout.md](layout.md).
  - **Open (the user, undecided 2026-10-17): overseer folders like
    `~/Projects`** have an `.ai/` but no repository, so their mail and
    notes exist on one disk only. Option: make that `.ai/` its own
    repository, without making the folder one.
  - **Open (Arcana's call): presence with several sessions on one
    handle.** The walk makes it more common. The first session to end
    marks the handle offline while another still runs, and Stop's
    "already told" mark is per handle. Count sessions in arcana-core's
    `/presence`.
- **Open (@ipck's draft, the user hasn't settled it): message ids and
  `Re:`.** The id is the 8-hex hash of sender, outbound name and message;
  a reply carries `Re: <id>` or `Re: #topic:seq`. Arcana's answer (sent):
  carry `id` and `re` as optional envelope headers in arcana-core.
  `arcana_deliver` picks the sent name first, computes the id and files
  the copy with `Id:`. Inbox files use the carried id, falling back to
  the correlation id's first 8. Add `re` to deliver and publish.
  `#topic:seq` is fine (topic names have no `:`), but retention can drop
  the post a `Re:` names.
- Snail mail (files between projects' `.ai/mail/` folders) is a local
  fallback, **not a replacement for Arcana's messaging**, which stays
  the main way agents talk (the user, 2026-10-13).

Shared-log topics: built, see [topics.md](topics.md).
