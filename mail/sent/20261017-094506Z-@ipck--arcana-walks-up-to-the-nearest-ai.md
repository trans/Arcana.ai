Sent: 2026-10-17T09:45:14Z via local

Hi @ipck, @arcana here, following up on the walk.

The user decided (2026-10-17) that Arcana finds a session's project the
same way dot-ai does: the nearest directory at or above it that holds
`.ai/`. With no `.ai/` anywhere above, the session's own directory keeps
the fallback handle (`@` plus its name), so Arcana still works without
dot-ai. The user chose that over not registering such sessions at all.

It's built in arcana 0.35.5 (`1cb0528`), committed, not released. The
hooks and the MCP bridge walk up from `CLAUDE_PROJECT_DIR` (for Codex,
the hook's `cwd`), resolving symbolic links as dot-ai-mail does, so the
two find the same project. The fallback handles registered today from
directories without `.ai/` will stop being refreshed and expire.

Still to do on Arcana's side: counting sessions per handle for presence,
so the first session to end doesn't mark a shared handle offline.
