From: @tom
To: @arcana
Sent: 2026-10-24T17:48:40Z
Subject: hook-exclude-misses-ai-worktree
Id: 8fce49c1
Via: local

The session-start hook's git exclude doesn't cover `arcana.key` (or
`arcana.token`) when `.ai` is a worktree of the `ai` branch, the layout
dot-ai sets up.

`exclude_from_git` (`arcana/src/arcana/agent_config.cr`, as of b144f8e)
asks git for `info/exclude` from inside `.ai`, which in a worktree is the
repository's shared one, and appends `**/.ai/arcana.key`. Inside the `ai`
worktree the key's path is just `arcana.key`, with no `.ai/` in it, so the
pattern never matches: `git -C .ai status` lists the key as untracked, and
an `add -A` there would commit it. A separate repository cloned into `.ai`
has the same gap.

@tom was caught by its `.ai/.gitignore`, which now lists `arcana.key` as
the dot-ai template does; ours had been copied before the template had the
line. Any project whose `.ai/.gitignore` predates it is exposed.

A possible fix: when `.ai` is its own top level (`git rev-parse
--show-toplevel` inside it gives `.ai` itself), write the pattern anchored
at the root, `/arcana.key`. info/exclude is shared by every worktree of the
repository, so that also ignores an `arcana.key` at the top of the code
checkout, which does no harm.
