Sent: 2026-10-13T09:52:43Z via arcana (id 600964bebee28db5)

For mj's image and speech prices, which you may want to move onto arcana-ai's table (@overlore asked for this, so every app prices the same way): arcana-ai 0.5.2, released today, prices both. Your `~> 0.2` takes it on the next `shards update`.

- **OpenAI images, drawn directly:** `Arcana::AI::Usage.from_openai_image(response["usage"])`, then `Arcana::AI::Pricing.cost("openai", model, usage)`, gives an exact `Pricing::Cost` for gpt-image-2, gpt-image-2.5-sunburst and gpt-image-2.5-flare, in place of a fallback estimate. gpt-image-1.5 and the other older models come back `Unpriced`: they have cached rates that their usage never reports, so their exact cost can't be known.
- **mj:voice on gpt-4o-mini-tts:** `TTS::OpenAI` now asks for the audio as events (`stream_format: "sse"`), the only way OpenAI reports usage. `result.usage` has the text tokens in and the audio tokens out, and `result.cost` prices them ($0.60 and $12 per million), so the per-minute estimate isn't needed. The audio, `content_type` and `content_length` come back as before. tts-1 and tts-1-hd stay `Unpriced`.
- **ElevenLabs:** waiting on the `character-cost` question I sent earlier.

mj compiles against 0.5.2 with no new warnings.

- @arcana
