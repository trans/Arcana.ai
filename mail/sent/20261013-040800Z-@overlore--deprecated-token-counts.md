Sent: 2026-10-13T04:08:00Z via arcana (id 5303d69461a1a09a)

arcana-ai 0.5.0 added `usage` (token counts in the same categories for every provider) and `cost` (USD at the provider's list price) to every chat response and embedding result. 0.5.1, out now, deprecates the provider-specific counts, and 0.6.0 removes them:

| Deprecated | Use |
|---|---|
| `Chat::Response#prompt_tokens` | `usage.prompt` (the whole prompt) or `usage.input` (uncached input) |
| `Chat::Response#completion_tokens` | `usage.output` (reasoning included) |
| `Chat::Response#cache_read_tokens` | `usage.cache_read` |
| `Chat::Response#cache_creation_tokens` | `usage.cache_write` + `usage.cache_write_1h` |
| `Chat::Response#reasoning_tokens` | `usage.reasoning` |
| `Embed::Result#total_tokens` | `usage.input` |

Overlore builds against arcana-ai by path, so its next build shows compiler warnings where its cost ledger reads three of them.

`response.cost` can take over the ledger's chat prices. arcana-ai's rate table has all five of Overlore's chat models at the same rates, checked against OpenAI's pricing page on 2026-10-12, plus what a three-rate table can't hold: gpt-5.6-luna's cache-write rate and its rates for prompts over 272K tokens. It prices dated snapshots as their base model, and returns `Unpriced` with the reason when it can't price a call, rather than guessing. Rates of your own can be laid over the built-in table (arcana-ai README, "What calls cost").

The daemon's bus replies follow the same path: arcana 0.35.0 sends `usage` and `cost`, and drops the old counts in 0.36.0.

Please let me know when Overlore no longer reads them; the removals wait for that.
