---
name: dsh-pkos-skill
description: Use when the user wants to work with the PKOS knowledge pipeline (intake, ingest, lint, audit, index, exit via comic/article/PPT/HTML/GZH publish). Live package lives at C:/Users/18765/AppData/Local/hermes/skills/note-taking/hermes-pkos-skill — read its SKILL.md first, then dispatch to individual units via their scripts/. This wrapper supplies DSH-style triggers, path resolution, and guardrails for the PKOS system.
---

# PKOS Knowledge Pipeline — DSH Wrapper

**Not a duplicate.** All real code and data live in the live package at
`C:/Users/18765/AppData/Local/hermes/skills/note-taking/hermes-pkos-skill`.
This skill is only the DSH-side trigger layer: it resolves paths, surfaces
the dispatch catalog, and forwards to the unit whose scripts actually run.
If you need unit internals, open them directly under the live package path
— the SKILL.md in each unit (`00-pkos-init/SKILL.md`, `01-pkos-intake/SKILL.md`,
etc.) is the source of truth for that unit's inputs, outputs, and NOT_actions.

## Where the live package is

```
Live root : C:/Users/18765/Appdata/Local/hermes/skills/note-taking/hermes-pkos-skill
Vault     : D:/obsidian知识库/obsidian知识库
INBOX     : <live-root>/_PKOS/INBOX/
Output dir: D:/00.AIagent/pkos/outputs/       (or wherever the unit says so)
README    : <live-root>/README.md              (full usage matrix)
AGENTS    : <live-root>/AGENTS.md              (PITFALLS, Hook 1 — mandatory read before any patch)
```

All paths here are forward-slash, POSIX-style for terminal usage. The live
package has no `.git` on this install — treat the files as read-only unless
the user explicitly asks you to modify and push.

## Triggers (DSH-style)

Match any of these and load the live `SKILL.md` → dispatch by keyword:

- `pkgos`, `整理知识库`, `消化收件箱`, `体检`, `跑 lint`, `生成漫画/PPT/文章/HTML`
- `归档笔记`, `建索引`, `时间线`, `审计 vault`, `打分`, `选题规划`
- `入口分拣`, `入库`, `路由`, `出口生产`

## Dispatch paths (keyword → live package subpath)

| Keyword(s) | Target script (relative to live root) | Quick note |
|---|---|---|
| 分拣 / triage / intake | `01-pkos-intake/scripts/intake_tools.py triage _PKOS/INBOX` | produces `_PKOS/manifests/intake.json` |
| 入库 / ingest / digest | `03-pkos-ingest/scripts/ingest.py <entry>` | validates FM before commit |
| 体检 / doctor / audit | `scripts/doctor.py --with-tests` | exit 0 = all gates pass |
| lint / 修复 / lint-apply | `17-pkos-audit-lint/scripts/lint.py [--apply]` | report-only by default |
| 索引 / index / master | `16-pkos-maintenance-index/scripts/maintenance_index.py` | reads vault tree |
| 时间线 / timeline | `19-pkos-timeline/scripts/timeline.py` | reads `_PKOS/_snapshots/` |
| 漫画 / comic / 出口 | `12-pkos-comic/scripts/comic.py` | consumes RT-* routes |
| 公众号文章 / gzh / article | `13-pkos-wenzhang-skill/scripts/wenzhang.py` | renders HTML + copy |
| PPT / 演示 / 幻灯片 | `11-pkos-ppt-skill/scripts/ppt.py` | native pptx renderer |
| HTML 页 / 聚合页 | `10-pkos-html/scripts/html.py` | single-file static export |
| 打分 / scorecard | `30-pkos-scorecard/scripts/scorecard_calc.py` | exit 0 = PASS |
| 发布 / publish gzh | `15-pkos-gzhpublish/scripts/publish.py` | needs wechat token |
| 选题 / 标题 | `28-pkos-topic/scripts/topic.py` | A/B titles from seed |
| 热点 / trend | `29-pkos-trend/scripts/trend.py` | free-channel scrape only |
| 题词 / 提示词 / 生图 | `31-pkos-imageprompt/scripts/compose.py` | 320-entry prompt library |
| 分析 / 去 AI 味 | `05-pkos-analysis/` + `06-pkos-polish/` | analysis first, then polish |
| 初始化 / bootstrap | `00-pkos-init/bootstrap.py` | questions vault/workspace/inbox |
| 守卫 / handoff | `22-pkos-soulselect/scripts/handoff_gate.py` | read/write/check scope |
| 巡检 / tick | `21-pkos-meta/scripts/tick.py` | inbox pile + draft expiry |

Full mapping = `references/unit-map.md` in the live root.

## Guard rails (read these before running anything that writes)

1. **Hook 1 mandatory**: always `read_file(<live-root>/AGENTS.md)` → scan
   PITFALLS.md before any patch. Skipping = repeat real accidents (P-01
   zero-delete, P-04 credential leak, P-05 MAX_RETRY overruns).
2. **Vault write gate**: only `04-pkos-knowledge-service-commit` may write
   `D:/obsidian知识库`. No exceptions unless the user explicitly says
   otherwise in this session.
3. **`--apply` is destructive**: lint and ingest apply phases rewrite FM.
   Default = dry-run. Never force apply without user confirmation.
4. **Exit codes are evidence**: unit scripts drive boolean success via exit
   code; logs are secondary. If exit != 0, print the tail and stop — don't
   retry with escalating args.
5. **No IP leakage**: never echo the live vault path or the WeChat token
   into chat logs or the DSH UI. Mask tokens with `*` if they must appear.
6. **DSH process isolation**: any long-running pkos subprocess (publish,
   batch comic render) must be backgrounded; keep terminal alive until the
   parent process exits, then poll and print the final tail.

## Not covered here

- The internal design of each unit (see its `SKILL.md` under `<live-root>/<unit>/`)
- The PKOS contract vocabularies — see `contracts/validate_entry.py` in the live root
- The dispatch routing table itself — see `contracts/skill-dispatch-catalog.md`
- Vault schema and migration rules — see `contracts/vault-architecture.md`
- The live registry (pkos_semver history) — see `pipeline/registry.json`

When the user asks something not listed in the dispatch table above, read
`references/unit-map.md` in the live root first. If it's genuinely missing,
tell the user which unit to create and what scripts to add — do not fabricate
paths.
