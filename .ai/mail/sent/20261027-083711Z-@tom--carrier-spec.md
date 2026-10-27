Sent: 2026-10-27T08:37:12Z via local
Id: 4a3561ed

The dot-ai spec now has a mail carrier. The user decided it on 2026-10-25, and it's on dot.ai's main, pushed (10ce3e1, e87bd06, 7918fcc in spec/mail.md). In short:

- Each machine has one carrier, git config ai.carrier: `local` (dot-ai-mail, the default) or `arcana`. A carrier that isn't the machine's stays out of mail entirely: it writes no mail files, and its tools for sending refuse.
- Agents write only into their own project's folders, and every message and every post goes through the outbox.
- Topics are logs the carrier keeps. An outbox file addressed to `#name` is a post: the carrier numbers it, and the sent copy gets a new `Post:` header line. A topic is made once through the carrier (`dot-ai-mail topic '#name' "description"`), read through it (`dot-ai-mail read '#name'`), and followed with `mail.follow` in `.ai/config.yml`. The local carrier keeps topics in `.ai/topics/` of the first `ai.path` directory and takes each number under the topic's `.posting` lock.

What it means for T.O.M.'s dotai plugin: if it delivers by itself rather than running `dot-ai-mail deliver`, posting to a topic has to work the same way (the lock, the file names, the header lines), and it should stay out of mail where `ai.carrier` names another carrier. dot-ai-mail 0.3.0 in ~/Projects/dot.ai (commit 96f435c, not released yet; it ships as plugin 0.4.0) is the reference implementation, and tests/dot-ai-mail.sh exercises it.

On Arcana's side: on a machine whose carrier is `local`, Arcana 0.38 will stop carrying agent mail. `arcana_deliver` to a project and `arcana_publish` will refuse and point to the outbox. Services stay on the bus.
