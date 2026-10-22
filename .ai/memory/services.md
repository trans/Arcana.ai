# Who supplies services

*Updated 2026-10-22*

- **Direction (the user, 2026-10-06):** arcana supplies the bus; project
  owners write their own services and register them. The daemon shouldn't
  keep growing provider integrations, which age at the speed of whoever
  last cared about them (arcana-ai's Runware identity requests were
  silently ignored by Runware for months).
- **Direction (the user, 2026-10-22):** everything goes through the bus,
  not a library: that decouples hard dependencies and lets projects be
  written in any language. Today Overlore links arcana-ai for OpenAI chat
  with tools, mj-arcana for speech, memo for embeddings, so each holds its
  own API keys and needs its own release to pick up a fix (arcana-ai 0.6.1
  took three). The aim: apps link at most a bus client, and arcana-ai
  lives only inside the services that wrap a provider. What the bus lacks
  first: proof of who calls ([keys.md](keys.md)), since one service then
  spends for every caller; chat on the bus that takes tools, tool calls
  and images (`openai`'s `chat` takes only role and text); streaming;
  and a written wire protocol for clients in other languages.
- `mj:camera` and `mj:voice` supersede the daemon's `runware` toolset and
  the `tts` tool on `openai`. `openai`'s `chat` and `embed` have no
  replacement and stay. **Open (the user is deciding):** whether and when
  to retire those two from `bin/arcana.cr`.
- **Proposed, not decided (2026-10-07):** move the provider toolsets out of
  the daemon into an optional process that registers like any other
  owner's service; admit a media integration into arcana-ai only once two
  projects use it (chat and embeddings stay: they reduce to a few wire
  formats); keep making `Toolset` services trivial to write.
- **Decided (the user, 2026-10-08):** Runware retries and the cap on
  concurrent requests belong in mj's own client, since Infocomic moves its
  images to `mj:camera` before launch. arcana-ai's `Image::Runware` is left
  as is.
- TTS pricing knowledge stays in mj until a second project needs it
  (agreed with @mj, 2026-10-06).
- **Later idea (the user, 2026-10-07):** when there are too many listings
  for agents to scan, Arcana becomes an operator/librarian that answers
  "who can do X?".
