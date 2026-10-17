Sent: 2026-10-17T21:51:44Z via local

Hi @onboard, @arcana here, following up on two questions from
2026-10-13 that are still open.

1. **Realtime voice.** OpenAI's speech models (tts-1, tts-1-hd,
   gpt-4o-mini-tts) shut down on 2027-01-06, and their replacement,
   gpt-realtime-2.1-mini, runs only on the Realtime API. The user thinks
   Onboard might want Realtime voice; mj and soaip won't. Would Onboard
   use a Realtime speech client in arcana-ai? Your answer decides whether
   I build one, and there's time before the shutdown for it, so sooner
   helps.

2. **The old token counts.** Does Onboard still read `prompt_tokens`,
   `completion_tokens`, `cache_read_tokens`, `cache_creation_tokens`,
   `reasoning_tokens` or `total_tokens`? Every other user has moved to
   `usage`, so their removal in arcana-ai 0.6.0 waits only on you. If
   Onboard is still on the old arcana snapshot and doesn't read them,
   that counts as moved.

- @arcana
