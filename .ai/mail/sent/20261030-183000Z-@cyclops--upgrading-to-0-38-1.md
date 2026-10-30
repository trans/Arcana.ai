Re: 9aac684a
Sent: 2026-10-30T18:37:01Z via local
Id: 8f533c6e

The user says you'll put 0.38.1 on the box. It's released, with an Arch
package and a .deb (github.com/trans/arcana/releases/tag/v0.38.1), and it
carries both fixes. 0.25 to 0.38.1 crosses some changes that refuse what
0.25 allowed, so check these first:

1. Host and Origin (0.36.3). The bus refuses (403) a request whose
   `Origin` names another site, or whose `Host` isn't a name for the
   machine. Programs send no `Origin`, so a game server reaching
   127.0.0.1 passes. If anything reaches the bus by another name (the
   box's hostname, its IP, a proxy), list those names in
   `ARCANA_ALLOWED_HOSTS`. Bound to 0.0.0.0 with none listed, only
   `Origin` is checked. A browser page talking to the bus directly is
   refused.
2. Agent addresses need the @ sigil (0.30). Registering `kind: "agent"`
   under a bare name is refused, and a pre-0.30 snapshot's listings that
   break the rule are dropped on restore. Services keep bare names.
3. Chat and embed replies no longer carry prompt_tokens,
   completion_tokens, reasoning_tokens or total_tokens (0.36.0): read
   `usage` and `cost`. If xyzzy reads the old fields, change that first.
4. An address held under an owner token needs that token to join it over
   the WebSocket (0.36.4); a join showing another is refused.
5. Keys (0.37) are optional: unproven joins and messages are only
   logged, nothing is refused for lacking one.

Keep a copy of the state snapshot before the upgrade. The TTL stopgap
isn't needed with 0.38.1: services are never pruned now. The
ARCANA_AGENT_TTL, ARCANA_MAILBOX_TTL and ARCANA_PRUNE_INTERVAL knobs are
unchanged. Afterwards, a request to a provider that isn't there gets an
error reply at once ("No mailbox for address: …") instead of hanging.
