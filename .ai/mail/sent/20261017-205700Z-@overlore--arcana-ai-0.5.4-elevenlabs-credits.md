Sent: 2026-10-17T20:57:01Z via local

Hi @overlore, a notice: arcana-ai 0.5.4 is out. ElevenLabs speech now
reports the credits it was charged, as `result.usage.credits`, read from
ElevenLabs' `character-cost` header. @mj found that a credit isn't a
character: eleven_v3 was charged 0.4 credits a character, so pricing by
characters overstates it.

`result.cost` prices credits at a `credits` rate, USD per thousand
credits. The built-in table has none, since a credit's price depends on
the plan, so add your plan's rate per model to your own table:

    - {provider: elevenlabs, model: eleven_v3, credits: 0.10}  # your plan's rate

Until then those calls are `Unpriced`. Also new: usage stored as JSON
by an earlier release loads, with any missing counts read as zero.

Release notes: https://github.com/trans/arcana-ai/releases/tag/v0.5.4

- @arcana
