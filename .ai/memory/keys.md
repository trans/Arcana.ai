# Key pairs: proving who sends

*Updated 2026-10-22. Built and released 2026-10-22: arcana-core 0.21.0,
arcana 0.37.0 (not yet installed). The user liked the idea ("can't hurt
local agents either") and settled the open questions the same day. Wire
details: `arcana/doc/PROTOCOL.md`, "Proving who sends".*

## Why

The bus takes `from` as written. Over REST it checks only that the
address has a mailbox, and an empty `from` skips even that. Over
WebSocket it checks nothing, and `join` asks for no proof, not even an
address's owner token: a join to an address already listed attaches to
its mailbox, and when that connection closes the bus unregisters the
address (dropping its owner token) and its mailbox. So a program on the
machine can send, join or read as any address. Since 0.36.4 a join can
show the owner token and a wrong one is refused, and a disconnect leaves a
held address alone; a join without the token still goes through (logged
`join.unproven`). @mj raised it
(2026-10-22, `mail/received/20261022-034813Z-@mj--senders-aren-t-checked-anything-on-the-bus-can-sen-fda0f7b2.md`):
mj:voice and mj:camera spend money on every call and must serve only
callers they trust. @memo asked the same (2026-10-22): memo:rag serves
several namespaces and wants each to accept only certain senders. Web pages, the outside way in, are refused since
0.36.3 (`LocalGuard`). The bus listens on 127.0.0.1 only, so what's
left needs a program already running on the machine.

## The design

Each project gets an Ed25519 key pair; the bus holds only public keys.

- **Keys.** Made at first registration, as the owner token is now: the
  private key in `.ai/arcana.key` (0600, kept out of git), the public key
  given to the bus. One key per project holds all its addresses (`@mj`,
  `mj:voice`, `mj:camera`). Later the key can sit in a key holder
  (ssh-agent, the TPM) that signs without letting the agent read it, so
  a prompt-injected agent can misuse its identity while running but
  can't hand it over.
- **Claiming.** Trust on first use, as owner tokens: the first key to
  claim a free address holds it. An address held under an owner token
  moves to a key when the claim also shows the token, once.
- **WebSocket.** At `join` the bus sends a random challenge and the
  client signs it. The connection is then bound to that address, and its
  messages count only as from it.
- **REST.** Each request is signed (method, path, time, a random nonce,
  the body's hash). The bus checks it against the key for `from`, and
  refuses a time more than 60 s off or a nonce seen within that window
  (replay).
- **Receivers.** The bus stamps each message `verified` (it overwrites
  whatever a client wrote). Messages also carry the sender's signature
  over from, to, id, time and the payload's hash, so a receiver like mj
  can check the sender itself without trusting the bus (decided).
- **Joining and reading** as an address need its key too, which closes
  the other two holes.
- **Enforcement.** First the `verified` stamp only, so nothing breaks;
  services turn down unverified callers themselves. Then
  `ARCANA_VERIFY_SENDERS=1` refuses unverified sends outright, once the
  event log shows no one still sends without a key. Owner tokens retire
  after that.
- **Crypto.** Ed25519 through OpenSSL's libcrypto (3.6 here), which
  Crystal already links: small bindings in arcana-core, so every Crystal
  client (`Arcana::Client`, Toolsets) gets signing.

## What it doesn't do

On one machine every project runs as the same user, so a program could
read any project's key file, or ask its key holder to sign. Real
separation needs agents running as separate users or sandboxed. Keys
stop programs and agents from using the wrong address, by mistake or
carelessly, and they're the footing a bus spanning machines needs.

## Decided

- **Signatures end to end** (the user, 2026-10-22): a message carries its
  sender's signature to the receiver, which can check it without trusting
  the bus. It keeps a network of Arcanas possible, where messages pass
  through buses the receiver doesn't run (see roadmap.md, Topology).

- **One key per project** (the user, 2026-10-22): it holds all the
  project's addresses that its sessions use.
- **A service has a key of its own** (the user, 2026-10-24, asked by
  @mj): a process that runs apart from the project's `.ai/`, sandboxed
  on the workstation or deployed on a server, makes a key on first start
  where it keeps its state, and that key holds only its own addresses.
  A compromised service then can't sign as the project's agent, nothing
  is bound in from `.ai/`, and no key is copied to a server. The cost:
  a project has several keys, so tying them together across buses would
  need the project key to sign its services' keys (not built).
  `ARCANA_KEY_FILE` names the file; `Arcana::Key.from_env` (arcana-core
  0.21.1) makes it there.
- **Owner tokens retire** once every client signs (the user,
  2026-10-22). Until then, showing an address's owner token once moves it
  to a key.

## As built (2026-10-22)

- arcana-core 0.21.0: `Key` (libcrypto Ed25519, PKCS#8 PEM, 0600, never
  overwritten), `Canonical` (RFC 8785, checked against the RFC's
  vectors), `Signing` (join, request and envelope proofs; `Nonces`),
  `Envelope#signature`/`#verified`/`#signed`/`#signer`, `Directory#claim(key:)`
  with `Claim::Moved`, `Client.new(key:)`. openssl's CLI reads our keys
  and checks our signatures, and the other way round.
- arcana 0.37.0: the bus sends a challenge to a join offering a key;
  checks signed REST requests (401 when wrong or replayed); stamps
  `verified`; keeps keys in the snapshot and listings. The session-start
  hook makes `.ai/arcana.key` (excluded from git in the repository the
  file is in, which matters where `.ai` is a link) and moves the handle
  to it; the hook and the bridge sign every request; the bridge signs the
  envelopes it sends. The bridge never makes a key itself.
- What a signature covers: from, to, subject, payload, correlation id,
  time (whole Unix seconds), id, re. Not `reply_to` (the bus sets it on
  a sync request) or `ordering`.
- Not enforced yet: `join.unproven` and `read.unproven` events, and
  `verified` on each `message.sent` event, show who still goes without
  proof. Next: `ARCANA_VERIFY_SENDERS=1` once they show nobody.
- A lost key can't be replaced by a new one through the token alone:
  unregister with the token, then register with the new key. An address
  held by a key alone (a service's) has no way back at all yet: someone
  stops the bus and removes the listing from its snapshot. To build: an
  operator's release, perhaps in the arcana-ctrl service.
- Known clients without keys: Cattacula's game joins `@cattacula`
  (the handle its Claude session holds) without proof; Overlore's REST
  calls; mj's services; memo-arcana.

## Order of work

Done: arcana-core, then arcana. Next: clients (mj's services, memo,
Overlore's REST calls, and the others listed in `coordination.md`),
then enforcement.
