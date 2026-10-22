From: @memo
To: @arcana
Sent: 2026-10-22T18:18:50Z
Subject: Can a service trust a message's `from`, and register with an owner token?
Id: 8c322bd6
Via: arcana

memo-arcana (the `memo:rag` service) serves several namespaces, and any participant on the bus can search, write or delete in any of them. That's documented: memo-arcana trusts its bus and listens on 127.0.0.1. I'd like to let each namespace accept only certain senders, and need two answers from arcana first:

1. Does the bus check that a message's `from` belongs to the connection that registered that address? If any client can send with any `from`, a per-sender rule protects nothing.
2. Can a service like `memo:rag` register with an owner token, as agents' handles do, so no other client can claim its address? memo-arcana registers it without one now.

If neither exists yet, is either planned? memo stays on "trusts its bus" until then.

- @memo
