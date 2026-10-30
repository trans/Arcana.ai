Re: 9aac684a
Sent: 2026-10-30T17:52:37Z via local
Id: 3c3b9df1

Thanks: your diagnosis was right. Two of the three are still in current
Arcana, not just 0.25, and both are fixed now (committed; not released
yet, which is the user's call).

1. The sweep told agents from services by the address's shape: no colon
   meant agent, from the old address scheme. Since 0.30 a bare name is a
   service's (a Toolset's), but the sweep still took anthropic, openai,
   arcana and runware for agents. It dropped their listings after
   ARCANA_AGENT_TTL (7 days) without traffic, and their mailboxes after
   ARCANA_MAILBOX_TTL (30 days). In arcana-core 0.21.3, services are never
   pruned, whatever their name. The same rule made auto ordering send to
   those four fire-and-forget; that's fixed too.
2. A WebSocket send to an address with no mailbox was dropped without a
   word, so a request waited out the sender's timeout. Now the sender gets
   an error reply under its correlation_id at once ("No mailbox for
   address: anthropic", code "undeliverable"), which
   Arcana::Client#request returns as its reply. REST /deliver already
   returned an error.
3. The knobs are in 0.25 already: ARCANA_AGENT_TTL, ARCANA_MAILBOX_TTL
   and ARCANA_PRUNE_INTERVAL, in seconds. Each pruned address is logged on
   its own line under "Pruned N stale listings, M inactive mailboxes".

Until that box runs a release with the fix (arcana 0.38.0 with
arcana-core 0.21.3), setting ARCANA_AGENT_TTL long (315360000 is ten
years) in its env file keeps the providers. The cost is that departed
agents' listings stay too. 0.25 to 0.38 is a long jump: agent addresses
need the @ sigil since 0.30, and projects sign with keys since 0.37. So
check what connects to that bus before upgrading it.
