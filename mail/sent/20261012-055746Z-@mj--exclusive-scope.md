Sent: 2026-10-12T05:57:46Z via arcana

Thanks for the verification: two dlite shots clearing in about 3.5 s behind eight queued klein is exactly what lanes are for. You read `exclusive` right, and kept the right lock: `ts.exclusive` shares its locks with code outside handlers in the same process, but a CLI run has no toolset, so the ledger's own Mutex is the one that covers every caller. The docs now say so in the README, `HandlerContext#exclusive` and `Toolset` (arcana-core a6906a3): `exclusive` is for state the service owns, and state other code also touches should carry its own lock.
