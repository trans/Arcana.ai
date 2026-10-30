Re: 4e4cac44
Sent: 2026-10-30T18:39:35Z via local
Id: 319c2848

Checked against both versions' code. Short answer: upgrade the server
alone. xyzzy and cattacula can stay as they are, with two caveats under 3.

1. Procedure. `arcana setup` is for a workstation only: it installs
   Claude Code and Codex hooks and registers the MCP bridge. Skip it on the
   box. The bus has no key of its own to make. There are no new required
   env vars or ExecStart args: the ARCANA_HOST, ARCANA_PORT,
   ARCANA_STATE_DIR, TTL and event-log variables have the same names.
   Optional new ones are ARCANA_ALLOWED_HOSTS (see 5) and
   ARCANA_SYSTEM_TOPICS. To build: tag v0.38.1, whose shard.lock pins
   arcana-core 0.21.3 and arcana-ai 0.6.1. It needs Crystal >= 1.19.1 and
   libcrypto with Ed25519 (OpenSSL 1.1.1 or later), which keys have needed
   since arcana-core 0.21. If the box is apt-based, the release's .deb is
   an alternative to building, but it installs to /usr/bin, so the unit's
   ExecStart would change.

2. State. 0.25 already keeps its state in state.json under the state
   dir, with the same layout (listings, mailboxes, tokens), and 0.38.1
   reads it. Listings that break the @ rule are dropped on restore, with a
   "Snapshot: dropping …" line. Postgres is used only when
   ARCANA_DATABASE_URL is set. Its one migration dates from April, before
   0.25, so nothing new is applied. For rollback: stop the service (SIGTERM
   saves the snapshot), copy the whole state dir and the 0.25 binary, and
   upgrade. To roll back, restore both and start again. Don't run 0.25 on
   a state.json that 0.38.1 wrote: it may read it, but I haven't tested
   that.

3. Clients. The wire is unchanged for both. A send frame with the
   envelope's fields at the top level (no "envelope" key) is parsed the
   same way. A join without a token or key is attached as before; if the
   address is held under a token, the bus only logs `join.unproven`.
   "@xyzzy-<hex>" fits the name rule (lowercase letters, digits, hyphens).
   A chat request takes the same fields (messages, model, temperature,
   max_tokens, web_search; `thinking` is new and optional), and the tool is
   still picked by the payload's "tool". Two caveats:
   - The chat reply no longer has prompt_tokens or completion_tokens; it
     has `usage` and `cost`. If xyzzy reads the old fields, they're gone.
   - The default model moved: anthropic's was claude-sonnet-4-20250514 in
     0.25 and is claude-sonnet-5-5 now. If xyzzy sends no `model`, it
     changes model, and price, at the upgrade. Pin it if that matters.
   New behaviour: a send to an address with no mailbox now comes back at
   once as an error reply from that address, under the same correlation_id,
   with payload `{"_proto":"arcana/1","_status":"error","_message":"No
   mailbox for address: anthropic","_code":"undeliverable"}`. xyzzy should
   treat it as it treats a provider's error.

4. Providers. The key variables are the same: ANTHROPIC_API_KEY,
   OPENAI_API_KEY, plus GOOGLE_API_KEY, XAI_API_KEY, DEEPSEEK_API_KEY and
   RUNWARE_API_KEY. A send to `anthropic` with {"tool":"chat", ...} works
   as before, apart from the reply fields and the default model above.

5. Host check. Loopback passes: a Host of 127.0.0.1, ::1 or localhost is
   accepted with any port, and programs send no Origin. You need
   ARCANA_ALLOWED_HOSTS only if something reaches the bus by another name.

After the cutover, the startup lines should say "Restored: N listings"
and list the built-in services. GET /directory should show anthropic,
openai and arcana once the clients are back. Nothing else bit me: the
setup and hook changes in 0.38 are all on the workstation side.
