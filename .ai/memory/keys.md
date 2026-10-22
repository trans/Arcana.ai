# Key pairs: proving who sends

*Updated 2026-10-22. A design, not built. The user likes the idea
(2026-10-22: "can't hurt local agents either"); the open questions below
are still theirs to settle.*

## Why

The bus takes `from` as written. Over REST it checks only that the
address has a mailbox, and an empty `from` skips even that. Over
WebSocket it checks nothing, and `join` asks for no proof. So a program
on the machine can send, join or read as any address. @mj raised it
(2026-10-22, `mail/received/20261022-034813Z-@mj--senders-aren-t-checked-anything-on-the-bus-can-sen-fda0f7b2.md`):
mj:voice and mj:camera spend money on every call and must serve only
callers they trust. Web pages, the outside way in, are refused since
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
  whatever a client wrote). Messages can also carry the sender's
  signature over from, to, id, time and the payload's hash, so a
  receiver like mj can check the sender itself without trusting the bus.
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

## Open questions (the user's)

- One key per project (proposed), or one per address?
- Signatures end to end, so receivers can check them (proposed), or
  checked by the bus only?
- Retire owner tokens once every client signs (proposed)?

## Order of work

arcana-core (keys, signing, `Client` signs its join), then arcana (the
server's checks, the bridge and hook make and use keys), then clients:
mj's services, Overlore's REST calls, and the others listed in
`coordination.md`.
