Sent: 2026-10-13T04:08:00Z via arcana (id 4d528ca68096a7c3)

Following up on 225cfbc1cb9e15c5 (shared provider costs).

arcana-ai 0.5.1 is out. It deprecates the provider-specific token counts, and 0.6.0 removes them:

| Deprecated | Use |
|---|---|
| `Chat::Response#prompt_tokens` | `usage.prompt` (the whole prompt) or `usage.input` (uncached input) |
| `Chat::Response#completion_tokens` | `usage.output` (reasoning included) |
| `Chat::Response#cache_read_tokens` | `usage.cache_read` |
| `Chat::Response#cache_creation_tokens` | `usage.cache_write` + `usage.cache_write_1h` |
| `Chat::Response#reasoning_tokens` | `usage.reasoning` |
| `Embed::Result#total_tokens` | `usage.input` |

The compiler warns wherever they're read. `usage` is nil when the provider reported nothing, where the old counts said 0 or nil depending on the provider. `response.cost` gives what the call cost in USD at list price, or `Unpriced` with the reason.

Onboard records usage from these fields today, so when it moves off the old arcana snapshot (the path in my last note), please go straight to `usage` and `cost` rather than through them. Test doubles built with `prompt_tokens:` and the like stop compiling in 0.6.0; build them with `usage: Arcana::AI::Usage.new(...)`.

The daemon's bus replies follow the same path: arcana 0.35.0 sends `usage` and `cost`, and drops prompt_tokens, completion_tokens, reasoning_tokens and total_tokens in 0.36.0.

Please let me know when Onboard no longer reads them; the removals wait for that.
