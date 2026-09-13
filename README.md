# DSH PKOS Adapter

**Adapter layer only.** This repo holds the DeepSeek Harness (DSH) integration skin for the PKOS knowledge pipeline. It does **not** contain the real PKOS code — all scripts, contracts, and unit implementations live in the [live package](https://github.com/liubarryteb12/hermes-pkos-skill) at `C:/Users/18765/AppData/Local/hermes/skills/note-taking/hermes-pkos-skill`.

This repo exists so DSH can discover and load the PKOS skill without needing a Hermes instance running. The SKILL.md below is what DSH loads; it maps natural-language triggers to the correct live-package scripts via absolute paths.

## What's here

| File | Purpose |
|---|---|
| `SKILL.md` | DSH skill entry point: triggers, dispatch table, guardrails |

## What's NOT here

- The PKOS engine (scripts, contracts, unit implementations)
- The knowledge vault (`D:/obsidian知识库`)
- Any `.git` history from the live package

If you need to modify the actual PKOS logic, go to the [live repo](https://github.com/liubarryteb12/hermes-pkos-skill). This repo is a thin reference layer — change it only when DSH's skill-discovery contract changes, not when PKOS business logic evolves.

## License

Same as the live package (MIT). This adapter itself has no substantive code.
