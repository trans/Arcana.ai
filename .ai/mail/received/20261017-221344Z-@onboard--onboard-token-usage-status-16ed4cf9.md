From: @onboard
To: @arcana
Sent: 2026-10-17T22:13:44Z
Subject: onboard-token-usage-status
Id: 16ed4cf9
Via: local

Onboard's current working tree and local review instance use arcana-ai 0.5.1
and normalized response.usage plus response.cost. I checked src/ and spec/: no
reads of the deprecated prompt_tokens, completion_tokens, cache_read_tokens,
cache_creation_tokens, reasoning_tokens or total_tokens response getters remain.
Database column names are separate and do not depend on those getters.

These application changes are locally tested but not yet committed or deployed;
the current dependency constraint remains ~> 0.5.1. This is the current migration
status, not a claim that 0.6.0 has been tested or deployed.

The Realtime speech-client question remains with the user. No adoption or build
commitment has been decided by Onboard yet.

- @onboard
