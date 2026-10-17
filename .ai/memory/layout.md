# Layout: one project, three repositories

*Updated 2026-10-17*

arcana, arcana-core and arcana-ai are separate repositories but one
project, `@arcana` (the user, 2026-10-17). On a machine that has all
three, they sit in a plain folder, not itself a repository. The folder's
`.ai` is a symbolic link to the arcana repository's `.ai/`:

```
Arcana/              plain folder (on this machine, ~/Projects/Arcana)
  .ai -> arcana/.ai  relative link, made by hand on each machine
  arcana/            the arcana repository; its .ai/ is the `ai` branch's worktree
  arcana-ai/
  arcana-core/
```

A session in any of the three, or in the folder itself, walks up to the
nearest `.ai` and finds `@arcana`'s. Arcana does this from 0.35.5
(`1cb0528`, `93c2140`), and dot-ai-mail walks the same way. The arcana
repository stays self-contained: a lone clone anywhere works as before,
with `git worktree add .ai ai`. The folder depends on the repository,
never the other way round.

**Why a link and not a setting in a real `.ai/`:** with a link, everything
that reads `.ai/` sees the real folder with no new code: agents reading
notes, git, mail, the owner token, transcripts. A setting would need every
tool to learn it first. Until then, dot-ai-mail would see a second
`@arcana` and stop delivering to either.

**Checked 2026-10-17** on a copy of the layout:
- dot-ai-mail finds `@arcana` once. Its search lists directories named
  `.ai`, not links.
- It sends from a sub-repository through the link and files the message
  in the real `sent/`.
- Arcana resolves every session in the folder to `@arcana`, with the same
  token and inbox.

## Setting it up

The worktree already uses relative paths (done 2026-10-17:
`git config worktree.useRelativePaths true`, then
`git worktree repair --relative-paths`). The repository can therefore move
with its `.ai/` inside it, and nothing needs repairing.

1. Close every Claude Code and Codex session in the three repositories.
2. Move them into the folder and make the link:
   ```sh
   mkdir ~/Projects/Arcana
   mv ~/Projects/arcana ~/Projects/arcana-core ~/Projects/arcana-ai ~/Projects/Arcana/
   cd ~/Projects/Arcana
   ln -s arcana/.ai .ai
   ```
   On a filesystem that ignores case (macOS, Windows), `Arcana` and
   `arcana` clash, so the `mkdir` fails while the repository is still in
   `~/Projects`. Move the repository to a temporary name first.
3. Check:
   ```sh
   git -C ~/Projects/Arcana/arcana worktree list  # .ai at its new place, not "prunable"
   git -C ~/Projects/Arcana/.ai status            # works through the link
   ```
   From `~/Projects/Arcana/arcana-core`, the plugin's
   `dot-ai-mail find @arcana` should print `~/Projects/Arcana/arcana`.
4. Claude Code files a session's memory and history under the folder it
   starts in: `~/.claude/projects/`, with `/` turned into `-` in the path.
   Most of this project's memory is here in `.ai/memory/`, so little
   depends on it. To keep the rest (how the user likes to work, private
   notes, session history), rename the old folder (ending
   `-Projects-arcana`) to end `-Projects-Arcana-arcana`. Then keep starting
   Arcana's sessions in the repository, where its
   `.claude/settings.local.json` applies. Claude Code asks once to trust
   the new path.
5. Until arcana 0.35.5 is installed, a session started in arcana-core or
   arcana-ai still registers a fallback handle of its own. Sessions in the
   repository or in the folder are `@arcana` either way.

## If the link breaks

If the arcana repository is renamed or moved out of the folder, the link
leads nowhere. The walk passes over it, so sessions in arcana-core land in
the project above (`@projects`). Since 0.35.5 (`93c2140`), session start
warns the agent when this happens. To fix it, remake the link in the
folder: `ln -sfn arcana/.ai .ai`.

## Portability: what may need doing later

- **Other machines and cloud sessions.** The folder and the link are made
  by hand on each machine; no repository records them. Another machine
  needs the same steps, or works from a lone arcana clone. A cloud session
  that clones only arcana-core is a project of its own, with a fallback
  handle. If that happens often, there are two fixes:
  - a setup command (a `just` recipe, or one in dot-ai) that clones the
    three and makes the link;
  - a committed pointer in arcana-core and arcana-ai naming their project.
- **Windows.** Symbolic links need Developer Mode or admin rights, and
  dot-ai-mail is POSIX sh. A directory junction (`mklink /J .ai arcana\.ai`)
  might serve; that's untested. Where no link can be made, the fallback is
  a pointer setting: a config key, or an `.ai` file naming the project's
  `.ai/`, like git's `.git` file. Every tool (Arcana, dot-ai-mail, Copious,
  the dot-ai skill) would have to learn it, so the spec would change first.
- **Relative worktrees** set `extensions.relativeWorktrees` in this clone's
  `.git/config`. Git before 2.48, and tools built on libgit2 or JGit (some
  editors and git GUIs), may refuse to open the repository. This affects
  only this clone; `.git/config` isn't pushed. To undo it, repair the
  worktree with absolute paths and remove the extension.
- **The spec.** dot-ai-mail skips the link because `find -type d` doesn't
  match links, not because the spec says it should. @ipck has been asked
  (2026-10-17) to write it into the spec, so that other implementations,
  such as Copious, do the same.
- **One project, two paths.** The folder and the repository both resolve to
  `@arcana`. Arcana and dot-ai-mail key nothing on the path, but a tool that
  did would see two projects.
- **Sessions sharing `@arcana`** become more likely, so presence should
  count sessions (see [status.md](status.md)).
