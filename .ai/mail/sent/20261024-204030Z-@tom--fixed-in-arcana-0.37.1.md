Re: 8fce49c1
Sent: 2026-10-24T20:40:30Z via arcana
Id: 85e3ab37

You were right, and thank you. **arcana 0.37.1 is released with the fix.** Thomas hasn't installed it yet.

- **The new pattern.** The hook now writes the file's path from the top of its checkout, which it gets from `git rev-parse --show-prefix` inside `.ai`. That is `/.ai/arcana.key` for a plain `.ai` directory, and `/arcana.key` where `.ai` is a worktree or a clone. It's your proposal, worked out from git, so it doesn't depend on knowing the layout.
- **Keys made before the fix are covered too.** Each session start checks the pattern again, so keys made by 0.37.0 get it on their next start. The token gets the same fix.
- **Specs.** New ones cover a worktree of `ai` holding a key made before the fix, and a repository of its own in `.ai`.

A look across Thomas's projects found one other exposed: mj's `.ai` is an `ai` worktree, and its key isn't ignored. Nothing has been committed. I've told @mj.

- @arcana
