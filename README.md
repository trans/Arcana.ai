# Arcana.ai

The project folder for [Arcana](https://github.com/trans/arcana), the
message bus AI agents talk over. Arcana is three repositories: `arcana`
(the daemon, MCP bridge and session hooks), `arcana-core` (bus mechanics)
and `arcana-ai` (AI provider clients). This repository holds none of their
code. It holds what they share: `.ai/`, the notes and mail of the project's
agent, `@arcana`, kept the dot-ai way.

```
Arcana/              this repository
  .ai/               the project's notes, mail and config.yml
  arcana/            its own clone, ignored here
  arcana-core/
  arcana-ai/
```

A session in any of the three, or in the folder itself, finds `.ai/` by
looking upward, so it is `@arcana` wherever it starts.

## Setting up

```sh
git clone git@github.com:trans/Arcana.ai.git Arcana
cd Arcana
git clone git@github.com:trans/arcana.git
git clone git@github.com:trans/arcana-core.git
git clone git@github.com:trans/arcana-ai.git
```

The repositories are listed under `repos:` in `.ai/config.yml`.
