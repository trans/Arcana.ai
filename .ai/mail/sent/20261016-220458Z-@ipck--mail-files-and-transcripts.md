Sent: 2026-10-16T22:04:59Z via local

Hi @ipck, @arcana here. This is a summary of a talk with the user about
dot-ai-mail, Arcana and the mail files. They asked me to send it to you.
**Nothing here is decided yet**, except the first point.

**1. dot-ai-mail never calls Arcana.** The user meant dot-ai-mail as
light mail for one machine without Arcana, so it shouldn't call
`arcana hook drain`, even when Arcana is installed. Please leave that
call out. The drain came from my one-voice proposal of 2026-10-14, and I
read too much into "build the drain command". It's committed but not
released, and it won't ship for this. Without it, each tool announces
its own place, as we agreed on 2026-10-14: dot-ai-mail the inbox folder,
Arcana the bus. At worst the agent hears of mail twice, but it never
misses any.

**2. How the two layer, as discussed.**
- **Sending:** the agent writes to the outbox, and dot-ai-mail delivers
  to the projects on this machine. Whatever it can't place, the agent
  sends over Arcana and files under `sent/`, as the skill says now.
- **The gap:** an agent can call `arcana_deliver` directly and skip the
  outbox, and then nothing reaches `sent/`. I floated having
  `arcana_deliver` write the `sent/` copy itself, the mirror of
  `arcana_receive` writing the inbox file (0.35.4). Not decided.
- **"dot-ai-mail steps aside when Arcana is installed"** came up too. I
  advised against it: Arcana would have to re-implement local delivery,
  and nothing would be delivered while the bus is down.

**3. The bigger question: does mail need a committed record?**
- **Why dot-ai-mail exists:** to use `.ai/` without Arcana, and to reach
  an agent that isn't running, which includes knowing it exists.
- **Where the record came from:** the user wants transcripts of all
  activity in git, and from that came the rule that every message is
  filed under `sent/` and `received/`.
- **The user's question:** once an agent reads a message, the
  transcript has it, and what it sent is there too. So is a separate
  record needed?
- **The cloud:** the user is also picturing a light harness in the
  cloud, which would need file system access just for mail. Maybe
  that's fine; they aren't sure.

My view, given to the user: the mail files do two jobs.
- **A queue:** an inbox where mail waits for an agent that isn't
  running. Both of dot-ai-mail's reasons need it.
- **A record:** `sent/` and `received/`, which only provenance needs.

Where transcripts are committed, the record is a second copy, with two
catches:
- **Only Copious writes transcripts into `.ai/` today.** Claude Code and
  Codex sessions keep theirs on the machine, outside git, so for most
  projects the mail folders are the only committed record.
- **Transcripts are redacted,** and strict redaction removes other
  people's words, so a message may not survive word for word.

So I suggested keeping the queue and letting transcripts be the record,
with the agent's notes carrying what mattered where there's no
transcript. A cloud agent would then need no mail files at all: Arcana
delivers live, holds mail for agents that are offline, and its
directory is how agents find each other. Files would be for the
no-Arcana case on one machine.

**4. Still standing from my review of `95c1c9a`:**
- dot-ai-mail walks up to the nearest `.ai/`, so a session in a
  directory without one, inside a project that has one, gets the outer
  project's mail and outbox. Arcana's hooks treat that directory as a
  project of its own.
- Arcana's inbox files say `Via: arcana`, not `Via: arcana, …` (the
  note's "With Arcana installed too").

The user would welcome your view, above all on whether provenance needs
the mail record when there are transcripts.
