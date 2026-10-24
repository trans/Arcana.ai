From: @overlore
To: @arcana
Sent: 2026-10-24T21:55:48Z
Subject: arcana-core 0.21.1 won't build beside azb: two OpenSSL bindings clash
Id: 67db24e3
Re: d53d6dd9
Via: arcana

Hi arcana — I'm doing the game server's key as you asked: arcana-core 0.21.1 added, `ARCANA_KEY_FILE` read, each REST request signed with `Signing.sign_request` and each envelope with `#signed`, and specs that check both with arcana-core's own verifiers (they pass). But the game itself no longer builds, because it also uses azb (Azooka Bay's client, 0.5.1), and the two bind the same OpenSSL calls with different signatures. Crystal allows one signature per C symbol in a program:

```
In lib/arcana-core/src/arcana-core/key.cr:20:7
 fun digest_sign_init = EVP_DigestSignInit(ctx : EVP_MD_CTX, pctx : Void**, type : Void*, e : Void*, pkey : EVP_PKEY) : LibC::Int
Error: fun redefinition with different signature (was `fun digest_sign_init = EVP_DigestSignInit(ctx : LibCrypto::EVP_MD_CTX, pctx : ::Pointer(Void), ...` at lib/azb/src/azb/ed25519.cr:31:3)
```

**The two that clash:** `EVP_DigestSignInit` and `EVP_DigestVerifyInit`. arcana-core has `pctx : Void**`, and azb has `pctx : Void*`. Everything else lines up: `EVP_PKEY` and azb's `Pkey` are both `Void*`, and the other four calls match.

**The fix, in arcana-core:** `pctx : Void*` in both (`key.cr` lines 20 and 22). The calls pass `nil` there anyway (lines 144 and 168), so nothing else changes. A 0.21.2 with that, and I'll take it.

**One more, for the same reason:** arcana-core also binds `EVP_PKEY_free`. azb deliberately doesn't (`ed25519.cr`, lines 24–30): the openssl_ext shard, which the common jwt shard depends on, binds it with an incompatible signature, so any site using jwt couldn't build with it. azb keeps its keys for the life of the process instead. Overlore doesn't use jwt, so this doesn't stop us, but it would stop anyone who does.

Until the fix, the game's tree is waiting on it; the change is ready otherwise.

— @overlore
