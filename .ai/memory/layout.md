# Layout: one project, four repositories

*Updated 2026-10-17*

arcana, arcana-core and arcana-ai are separate repositories, since a
Crystal shard needs one of its own, but one project, `@arcana` (the user,
2026-10-17). The project has a fourth repository, `trans/Arcana.ai`, that
holds none of the code (the user, 2026-10-17; the capital sets it apart
from a code repository). It holds `.ai/`, committed on its `main`, a
README, and a `.gitignore` naming the other three. Its clone is the
project folder, and the three code repositories are clones inside it:

```
Arcana/              clone of trans/Arcana.ai (on this machine, ~/Projects/Arcana)
  .ai/               notes, mail, config.yml
  arcana/            its own clone, ignored by Arcana.ai
  arcana-core/
  arcana-ai/
```

A session in any of the three, or in the folder itself, walks up to the
nearest `.ai/` and finds `@arcana`'s. Arcana does this from 0.35.5
(`1cb0528`), and dot-ai-mail walks the same way. Git for the notes works
from anywhere in the folder: `git -C .ai ...` from the folder,
`git -C ../.ai ...` from a code repository. Push with `git -C .ai push`.

**History.** Until 2026-10-17 the notes were the orphan `ai` branch of
`trans/arcana`, checked out as a worktree at `arcana/.ai`. Its 42 commits
are `Arcana.ai`'s history, followed by one that moves them into `.ai/`, so
`git log --follow` still traces each note. For a few hours that day the
folder was plain and its `.ai` linked to `arcana/.ai`. The user didn't
want one code repository holding the notes of all three, so the link gave
way to this. The `ai` branch was deleted from `trans/arcana` the same day
(the user's go-ahead), after checking both its copies were in `Arcana.ai`.

**Why not git submodules:** a submodule fixes a child at one commit, and
the parent needs a commit each time the child moves. These three move on
their own `main`s, and `shard.lock` already fixes the versions they build
against.

## Setting up on a machine

```sh
git clone git@github.com:trans/Arcana.ai.git Arcana
cd Arcana
git clone git@github.com:trans/arcana.git
git clone git@github.com:trans/arcana-core.git
git clone git@github.com:trans/arcana-ai.git
```

The three are listed under `repos:` in `../config.yml`. **Decided (the
user, 2026-10-17): cloning isn't automatic.** An agent that finds one
missing offers to clone it, as the dot-ai skill has it offer to check out
the `ai` branch; no hook or code. It happens once a machine, and a hook
that clones would hit the network and ssh at every session start. Cloud
sessions, where it would matter, have setup scripts of their own. A line
for the skill goes to @ipck once `repos:` has been used a while.

Claude Code files a session's memory under the folder it starts in, so
sessions started in the folder and in `arcana/` keep separate memory unless
the machine links them. Start in `arcana/` to have its
`.claude/settings.local.json` apply.

## Off the dot-ai spec

- The spec has `.ai/` as a worktree of the project repository's orphan
  `ai` branch. Here `.ai/` is committed on `main` of a repository of its
  own, and `repos:` is a key the spec doesn't have. Both are for the
  spec (the user's, with @ipck), once they've been used a while. Sent to
  @ipck 2026-10-17
  (`mail/sent/20261017-202708Z-@ipck--the-folder-is-its-own-repository.md`):
  this morning's request to spec the link is withdrawn, both are
  described, and who clones is left open. It may also answer the open
  question of overseer folders like `~/Projects` (see
  [status.md](status.md)).
- dot-ai-mail finds `@arcana` once: it searches for directories named
  `.ai`, and there is one.

## Portability

- **A code repository on its own**, outside the folder or in a cloud
  session that clones only arcana-core, has no `.ai/` above it. Its
  sessions land in the project above (`@projects` here) or get a fallback
  handle. Cloning `Arcana.ai` first avoids it.
- **Windows** needs no link any more. dot-ai-mail is still POSIX sh.
- **Sessions sharing `@arcana`** are more likely with four places to start,
  so presence should count sessions (see [status.md](status.md)).
