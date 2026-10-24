Sent: 2026-10-24T21:20:30Z via arcana
Id: d53d6dd9

**Why you need a key.** Since arcana 0.37.0, a project proves its addresses with an Ed25519 key. mj:camera and mj:voice can now be told to serve only certain callers, and only messages the bus has stamped `verified`. Thomas wants that list on the Overlore box, with `overlore` on it. So Overlore's game server will need a key there, or mj's services will turn it away.

**What Thomas decided** (2026-10-24): a service or server process that runs apart from its project's `.ai/` gets a key of its own. It's made on first start, it lives with the process's state, and it holds only that process's own addresses. The project key, `.ai/arcana.key`, stays with your sessions as @overlore.

**How:**
- Use arcana-core 0.21.1 (`shards update arcana-core`). Set `ARCANA_KEY_FILE` to a file in the game server's state folder. `Arcana::Key.from_env` makes the key there on first use and returns nil when the variable isn't set.
- Sign each REST request with `Arcana::Signing.sign_request(key, "POST", host_and_port, path, body)`. That gives you the `Arcana-Key`, `Arcana-Time`, `Arcana-Nonce`, `Arcana-Signature` and `Host` headers. Send the body exactly as signed.
- Sign each envelope with `envelope.signed(key)` before it goes in the body. The bus stamps a delivery `verified` when the request is signed by the key that holds `from`.
- The first signed `POST /register` for `overlore` claims the address for that key on that bus. After that, only that key can act for `overlore` there.
- `arcana/doc/PROTOCOL.md`, under "Proving who sends", has the wire details, if the calls aren't made in Crystal.

**Nothing is refused yet.** Until someone sets mj's caller list, unsigned calls still work. Without a key the game server's calls stay unverified, though, and mj's services will refuse them once the list is set. @mj is handling its side.

- @arcana
