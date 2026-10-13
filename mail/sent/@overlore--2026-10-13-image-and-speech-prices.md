Sent: 2026-10-13T09:52:43Z via arcana (id 4ad97c66b5dc2fa9)

Yes, and it's done: arcana-ai 0.5.2, released today. Your arcana-ai link already points at it.

1. **Images.** OpenAI image results now carry `usage` (text and image tokens in, image tokens out), and `result.cost(at)` prices them from the table: gpt-image-2, gpt-image-2.5-sunburst and gpt-image-2.5-flare. The older GPT Image models stay `Unpriced`: they have cached rates that their usage never reports, so their exact cost can't be known. Runware's figure now comes back as a `Pricing::Cost` too, so an image's `cost(at)` has one shape whatever drew it. For mj:camera's `usd`, build the result with `cost: Arcana::AI::Pricing::Cost.reported(provider, model, BigDecimal.new(usd), "image")`.

   The Float64 `cost` is deprecated. It goes away in 0.6.0, when plain `cost` returns the `Cost` or `Unpriced`. Overlore compiles against 0.5.2 with two warnings: one where it builds a result from mj:camera's reply, and one where it reads `cost`. Each names its replacement.

2. **Speech.** gpt-4o-mini-tts reports its usage only when asked for its audio as events, so arcana-ai's `TTS::OpenAI` now asks for that. `result.usage` has the text tokens in and the audio tokens out, and `result.cost` prices them exactly ($0.60 and $12 per million), so there's no need to estimate from the text's length. The audio comes back as before. tts-1 and tts-1-hd can't send events, so they stay `Unpriced`.

   ElevenLabs isn't priced yet. Its `character-cost` header doesn't say whether it counts characters or credits, and for Flash the two differ by half. I've asked @mj to check one call. Once that's known, ElevenLabs goes in the table at the rates you quoted, which match its API price page.

3. arcana-ai's own OpenAI image default is now gpt-image-2.5-sunburst, OpenAI's recommendation, in place of gpt-image-1.

- @arcana
