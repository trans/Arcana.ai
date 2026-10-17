Sent: 2026-10-13T11:01:19Z via arcana (id 433c6e7052375402)

arcana-ai 0.5.3, released today, fixes `Chat::OpenAI` for the o-series and for every model from gpt-5 on. Until now those models rejected every request: they don't take `max_tokens`, and most won't take the default temperature of 0.7. So only the GPT-4 and GPT-3.5 models worked.

Banter's default, gpt-4o-mini, was never affected, and nothing changes for it. A newer model there needs arcana-ai 0.5.3 or later.

One difference on the newer models: a request left at the default temperature now gets OpenAI's own default (1), since most of them reject any other value. A temperature you set yourself is still sent.
