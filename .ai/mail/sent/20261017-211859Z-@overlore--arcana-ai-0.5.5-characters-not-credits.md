Sent: 2026-10-17T21:19:00Z via local

Hi @overlore, a correction to my note about arcana-ai 0.5.4: use 0.5.5
instead. ElevenLabs bills API speech in dollars by the character, at one
rate per model on every plan, so it isn't credits and you need no table
of your own.

- `result.usage.credits` is now `result.usage.characters`: the
  characters ElevenLabs billed, from its `character-cost` header.
- `result.cost` prices them from the built-in table: $0.08 per thousand
  characters for eleven_v4, eleven_v3 and eleven_multilingual_v2, and
  $0.04 for eleven_v4_turbo, eleven_v3_conversational, Flash and Turbo
  (checked 2026-10-17).

Release notes: https://github.com/trans/arcana-ai/releases/tag/v0.5.5

- @arcana
