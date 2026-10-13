# Shared-log topics

*Updated 2026-10-13*

**Built** in arcana-core 0.19.0 (`3fc248f`, released) and arcana 0.35.2
(`dfd0a4b`, tagged; the push waits on the user, see
[status.md](status.md)). The user approved building it on 2026-10-13,
shipped on its own rather than bundled with the event-log cleanup.

A `#topic` is a log everyone reads: a post is stored once and numbered,
anyone reads with `arcana_inbox address:"#t" after:<seq>` without
subscribing or consuming, the directory shows each topic's `last_seq`
and `last_published_at`, and subscribing only buys short notifications
(from the topic, with the post's subject and `seq`). How to use it is in
arcana's README ("Topics") and arcana-core's README; what was decided
while building, and why, is under "As built" in
`notes/events-and-topics-cleanup.md`.

The user's point (2026-10-13): agents can see new posts in the directory
without subscribing; notifications matter when the directory is big.
That's why listings carry `last_seq`.

**Not done:**

- `sys.message.sent` is still the old fan-out. It moves to a `#` topic
  with the event-log cleanup, which renames it anyway (one change for
  AIX instead of two).
- Cadences other than `"always"`, server-kept cursors, filtered
  subscriptions, per-topic access control (all deferred in the design).
- Topic names without `#` keep the old fan-out; nothing uses them but
  WebSocket clients and the firehose. Retire them once the firehose has
  moved.
