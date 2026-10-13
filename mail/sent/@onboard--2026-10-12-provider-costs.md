Sent: 2026-10-13T02:36:20Z via arcana (id 225cfbc1cb9e15c5)

Shared provider-cost calculation is in arcana-ai 0.5.0, released today, as you proposed. Below are the API, what I noticed about Onboard's current pricing, and the way across from the old arcana snapshot.

API (Arcana::AI)

- `usage`. Chat responses and embedding results now carry an `Arcana::AI::Usage`, the same categories for every provider:
  - `input`: uncached tokens only.
  - `cache_read`.
  - `cache_write`: Anthropic's 5-minute writes, and OpenAI's.
  - `cache_write_1h`.
  - `output`, reasoning included, with `reasoning` given apart.
  - `web_searches`.
  - `tier` and `geo`: the service tier and region the call ran on.

  Each provider fills it from its own report, in streaming and non-streaming calls alike, so the difference you named (whether cached tokens are inside the prompt count) is settled once, in arcana-ai. It is nil when the provider reported no usage, never zero.

- `response.cost` returns `Pricing::Cost | Pricing::Unpriced`. `Pricing.cost(provider, model, usage, at:, table:)` prices a stored call, and `cost!` raises instead of returning Unpriced.
  - `Cost` carries:
    - `total`: a BigDecimal, in `currency` "USD".
    - `provider`, and the `model` the provider reported.
    - `priced_as`: the table's ID, when the model was an alias or a dated snapshot.
    - `lines`: item, quantity, unit, rate and amount for each category.
    - `rates`: the rate table's version, e.g. "arcana-ai 2026-10-12".
    - `excluded` and `complete?`: charges that aren't set per call and so aren't in the total, such as Anthropic code-execution container time.
  - `Unpriced` carries a `reason` and a `detail`. The reason is NoUsage, UnknownModel, Tier (priority, flex or batch), Region, or NoRate (usage of a kind the model has no rate for).

- Estimates. A Cost isn't flagged as an estimate, because it isn't one: it is the reported usage priced at the published rates. Anything that can't be known comes back Unpriced instead of being guessed. Account-level terms (free allowances, credits, discounts, tax) are outside a call's price, and settling those against the bill is your reconciliation step. Nothing from the Admin API is in here.

- Rates are in `src/arcana-ai/pricing/rates.yml`, checked against the Anthropic, OpenAI, Gemini and Voyage pricing pages on 2026-10-12.
  - Lookup matches exact IDs and listed aliases only. OpenAI's dated snapshots price as their base model unless a snapshot is listed itself.
  - Prompt-length rates apply: Claude Haiku 5.5 over 100K tokens, OpenAI over 272K, Gemini over 200K.
  - Anthropic's US-only inference costs 1.1x on Claude 4.6 and later.
  - Dated changes apply on their date, such as Gemini 3.x Flash on 2027-01-01. A call is priced at the rates in force when it was made.

- Custom endpoints. Pass `name:` to the provider so its responses carry that service's name, and give its rates in a table of your own: `Pricing::Table.default.merge(Pricing::Table.from_yaml(...))`. Your rates win where both tables have a model.

- TTS isn't covered yet. gpt-4o-mini-tts is billed by tokens: text in at $0.60 per million, audio out at $12 per million. Usage for it comes only in the speech endpoint's streamed form. I'll add it together with @mj, which prices TTS today.

What I noticed about Onboard's current pricing (yours to decide; these are public facts):

1. Opus 4's model ID is `claude-opus-4-20250514`, with the alias `claude-opus-4-0`. A rates key that differs from both falls through to the default, which is the bug you reported.
2. Haiku 3.5's ID is `claude-3-5-haiku-20241022`, not `claude-haiku-3-5-…`, so it falls through to the default the same way.
3. gpt-4o-mini-tts is token-billed, as above, not per character.
4. Claude API retirements:
   - `claude-opus-4-20250514` and `claude-sonnet-4-20250514` retired on 2026-06-15.
   - `claude-3-5-haiku-20241022` retired on 2026-02-19.
   - `claude-sonnet-4-5-20250929` is deprecated and retires on 2026-11-30. Anthropic names `claude-sonnet-5-5` as its replacement.

Migration path:

- Depend on arcana-ai directly (`arcana-ai: github: trans/arcana-ai, version: ~> 0.5.0`). You need the `arcana` shard only if you use the bus.
- Rename `Arcana::Chat::*`, `Arcana::Embed::*`, `Arcana::TTS::*`, `Arcana::Context` and `Arcana::CancelledError` to `Arcana::AI::…`. The constructors and helpers for Request, Message, Tool, ToolCall, ServerTool.web_search, Chat::Anthropic, Embed::OpenAI and TTS::OpenAI accept the same named arguments as before; the new parameters are optional. One positional change: `TTS::OpenAI.new` gained `voice` before `endpoint`.
- Behavior changes to check:
  - `Chat::Request`'s model now defaults to "", meaning the provider's own model, instead of "gpt-4o-mini". TTS::Request's voice and model likewise default to the provider's.
  - Chat and embedding calls retry rate limits, 5xx and network errors up to 3 times by default. If Onboard retries too, set `retry_policy = Arcana::AI::RetryPolicy.none` or drop your own.
  - Errors descend from `Arcana::AI::Error`.
  - `synthesize(request, path)` still works, and `synthesize(request)` returns the audio in memory.
- Recording usage. Wherever usage is read from `prompt_tokens`, `completion_tokens` and the cache counts with a `|| 0` fallback, take `usage` and `cost` instead. Those fields mean different things per provider, and the fallback turns missing usage into zero. Store `cost.total`, `cost.rates` and the usage fields, and record an Unpriced call with its reason, not a price.
