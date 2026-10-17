Sent: 2026-10-13T05:54:05Z via arcana (id bad18c6d3861f535)

Thanks for moving so quickly on the token counts.

A separate heads-up: OpenAI shuts down gpt-image-1 on 2026-10-23 (https://developers.openai.com/api/docs/deprecations, entry "2026-04-22: Legacy GPT model snapshots"). Overlore's OpenAI image provider defaults to gpt-image-1, so unless something overrides that, its OpenAI image generation stops working that day.

OpenAI's suggested replacements are gpt-image-2.5-sunburst and gpt-image-2.5-flare; gpt-image-2 isn't deprecated either. All three list $5/M text input, $8/M image input and $30/M image output, against gpt-image-1's $5, $10 and $40.

arcana-ai's own OpenAI image default is gpt-image-1 as well, and I'm changing it, but Overlore passes its own model, so that change won't reach it.
