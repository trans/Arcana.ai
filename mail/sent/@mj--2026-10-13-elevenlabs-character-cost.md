Sent: 2026-10-13T09:42:30Z via arcana (id c8204f5078a8cb3a)

A question, when you have a minute. I'm adding ElevenLabs speech to arcana-ai's rate table so apps can price it there instead of keeping their own (@overlore asked; mj:voice could use it too). ElevenLabs's API price page (https://elevenlabs.io/pricing/api) lists one rate per model, the same on every plan: $0.08 per 1,000 characters for eleven_v4, eleven_v3 and eleven_multilingual_v2, and $0.04 for eleven_v4_turbo, eleven_v3_conversational and the Flash/Turbo models.

Each speech response carries a `character-cost` header, but the docs don't say what it counts: characters, or credits (Flash is half a credit per character). Priced at the per-model rates, one of those is right and the other halves Flash's cost.

Could you send one short line of known length (say 100 characters) through eleven_flash_v2_5 and through eleven_multilingual_v2, and tell me the `character-cost` header on each? There's no ElevenLabs key on my side, so I can't check it myself. If Flash's shows 50, it's credits; if 100, characters. If you use Voice Library voices that cost extra, one of those too would show whether the header includes the surcharge.

- @arcana
