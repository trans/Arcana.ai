# Later

*Updated 2026-10-14*

Planned or parked work beyond [status.md](status.md) and
[topics.md](topics.md). None of it is scheduled.

- **SaaS auth stage 3, per-org isolation.** Stages 1 (identities, orgs, API
  keys in Postgres, `arcana-admin`) and 2 (bearer tokens enforced with
  `ARCANA_AUTH_REQUIRED=1`) shipped in 0.18 and 0.19. Stage 3 splits into
  3a (bind a process to one org with `ARCANA_ORG`) and 3b (a dispatcher in
  front). Keep `ARCANA_AUTH_REQUIRED` separate from `ARCANA_DATABASE_URL`,
  and keep `/health` open. Operator walkthrough: `doc/AUTH.md`.
- **Durability past snapshots.** Mail survives only a clean shutdown; an
  append-only event log, or a Postgres `StateBackend`, is the next tier.
- **Audio over the wire.** Inline TTS is base64 in JSON (+33%). Real fixes
  are transport-level: multipart responses, a WebSocket binary frame, or
  binary references.
- **Mailbox privacy.** Mail is readable by any local client today; see
  [registration.md](registration.md).
- **Chain-of-thought** shipped (arcana-ai 0.2.0, arcana 0.31.0); a separate
  `anthropic:research` service for web search waits for a second consumer.
- **Transcript encryption to recipients** (an idea the user had
  @azookabay pass on, 2026-10-14; `mail/received/`): encrypt each
  transcript to public keys listed in the config (with `age`, say), so
  machines that write transcripts hold no secret, with the reading key on
  a hardware token plus an offline backup. Not Arcana's to build:
  transcripts are written by the runtime (Copious) to the dot-ai spec.
  Noted here until the user says where it goes.
