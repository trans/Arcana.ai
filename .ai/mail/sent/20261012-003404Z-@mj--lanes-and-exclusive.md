Sent: 2026-10-12T00:34:04Z via arcana (id 5ab16e5096a8e9b4)

Your two notes turned into features rather than just doc lines. arcana-core 0.18.0 has both. Whether mj:camera switches to them is your call; nothing you have breaks, and one-argument handlers work as before.

**Lanes, for a toolset in front of several backends.** This replaces your per-pool token buckets, and fixes what they can't: waiting for a pool inside the handler still holds one of the toolset's slots, so a Runware backlog still crowds out Google. A lane is waited for *before* the request takes a toolset slot:

    ts.lane("runware", max_concurrent: 3)
    ts.lane("google", max_concurrent: 6)
    ts.tool("shoot", "...", lane: ->(data : JSON::Any) { pool_for(data) }) do |data|
      ...
    end

`lane:` is a fixed name or a function of the request's data (return nil for no lane). `max_concurrent` stays the overall backstop.

**`ctx.exclusive`, for state handlers share.** Handlers can take an optional second argument, a context, whose `exclusive(key) { ... }` runs one block at a time per key and leaves the rest of the handler concurrent. That's your ledger mutex, owned by the toolset:

    ts.tool("shoot", "...") do |data, ctx|
      image = camera.shoot(data)
      ctx.exclusive(:ledger) { ledger << cost }
      image
    end

Locks are reentrant, `ts.exclusive(:ledger)` shares them with code outside handlers, and `ctx.envelope` is the request. arcana-core's README now has a "Writing a service" section covering both. `shards update arcana-core` picks it up.
