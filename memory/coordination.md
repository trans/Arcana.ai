# Coordination

*Updated 2026-10-11*

- This project's agent is `@arcana` (`.ai/config.yml`). The bare `arcana`
  address is the daemon's own toolset (echo, help, ...), a different entity
  with its own mailbox.
- Outbound mail follows the dot-ai rules (see the README): routine mail
  goes out; anything that commits the user or the project waits for them.
- arcana-ai has several consumers, so its breaking changes reach all of
  them: arcana, minanime (@mj), soaip, goldfish (tracks `main`) and
  Overlore, formerly infocomic (a path dependency, so it builds against the
  working tree).

## Peers

- **@mj** (minanime): runs its own bus services, `mj:camera` (Runware
  images) and `mj:voice` (TTS through arcana-ai), built on arcana-core and
  arcana-ai. Found the arcana-ai Runware identity bugs, the
  `Toolset#start` pitfall that led to `Toolset#run` (arcana-core 0.15.0),
  and `Toolset`'s one-request-at-a-time dispatch (2026-10-11).
- **@memo** (the Memo search library): asked for the retry policy that
  shipped in arcana-ai 0.4.0, for Overlore's launch.
- **@overlore** (Overlore, the game formerly called infocomic; its handle
  was `@infocomics` until 2026-10-09, still checked for a while): calls
  arcana-ai chat, image and TTS directly; plans to move images and voices
  to mj's services before launch. Asked for chat image input (arcana-ai
  0.4.0), passing URLs for S3-stored pictures and bytes for local ones.
  Its service calls now come from `overlore` (was `infocomic`).
- **@cyclops**: investigated provider keys leaking into a workstation's
  user environment (2026-10-08). Told: arcana writes none of
  `environment.d`, `import-environment` or shell universal variables, and
  isn't BYOK today (the daemon's provider services use keys from its own
  environment).
- Others seen on the bus: @datadungeon, @oyl, @curio, @owl, @march.
