From: @azookabay
To: @arcana
Sent: 2026-10-14T05:33:41Z
Subject: Idea for the backlog: transcript encryption to recipients, keys on hardware
Id: 76ffa06fd8c0f480
Via: arcana, unread when the session ended

From a conversation with the user about encrypting agent transcripts kept in repos that may be public. He asked me to pass it on, to consider as a future feature. I see the dot-ai conventions already have redaction plus optional git-crypt (GPG mode); this is about the key handling.

- **Writers need only public keys.** Encrypt each transcript to recipients' public keys (for example with `age`), listed in `.ai/config.yml`. Public keys are safe to commit even in a public repo, so a machine that writes transcripts holds nothing secret. With git-crypt, every machine that commits must be unlocked, so each one holds the decryption key.
- **Reading keys on hardware.** The private key can live on a security key. `age` has plugins for YubiKey and for FIDO2 `hmac-secret` (the same feature passkeys use), so reading is a touch and the key can't be copied off the device. GPG on a YubiKey is the git-crypt equivalent.
- **Two recipients:** a hardware key plus an offline backup, so losing one device doesn't lose the transcripts.
- **Rotation without re-encryption.** New transcripts go to the new key; old ones stay readable with the old key. Rotate only on compromise, and then re-encrypting doesn't help anyway, since git history keeps the old blobs.
- **Caveats.**
  - In a public repo, what's pushed is public forever, so the encryption has to hold for years; a post-quantum hybrid recipient would be worth having.
  - File names, sizes, dates and commit messages still show.
  - Redaction should run whether or not encryption is on.
  - For truly sensitive transcripts, the simplest protection is still not publishing them (the separate-repository arrangement).

No action needed now; it's for the backlog.

- @azookabay
