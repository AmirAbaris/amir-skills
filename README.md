# amir-skills

My working set of [Claude Code agent skills](https://code.claude.com/docs/en/skills), curated from a few public collections. One thing is removed on purpose. Nothing here depends on a paid third-party review service, because the review gate is a skill in this repo, `code-review`, rather than a subscription. Sources and licenses are at the bottom.

Each skill's `description` decides when it fires. There is no workflow to learn and nothing to configure. Install them and work normally.

| Skill | |
|---|---|
| [`grill-with-docs`](skills/grill-with-docs/SKILL.md) | Interrogates a rough idea before any code exists, writing `CONTEXT.md` and ADRs as decisions land. The one you call yourself, since it won't fire on its own. |
| [`new-feature`](skills/new-feature/SKILL.md) | Puts a new task on its own branch off `origin/main`. |
| [`code-review`](skills/code-review/SKILL.md) | Reviews a diff twice over, Standards and Spec, as separate parallel passes. |
| [`codebase-design`](skills/codebase-design/SKILL.md) | Vocabulary for deep modules, meaning a small interface with real behavior behind it. |
| [`domain-modeling`](skills/domain-modeling/SKILL.md) | Writes and challenges project terminology, `CONTEXT.md`, and ADRs. |
| [`before-and-after`](skills/before-and-after/SKILL.md) | A before/after screenshot pair for a UI change. |
| [`evidence-driven-testing`](skills/evidence-driven-testing/SKILL.md) | A recorded walkthrough with pass/fail assertions, instead of a prose "tested it" claim. |
| [`unslop`](skills/unslop/SKILL.md) | Strips AI-tell phrasing from commit messages, PR bodies, and docs. |
| [`shadcn`](skills/shadcn/SKILL.md) | shadcn/ui components, registries, `components.json`. Fires on its own only, so you can't type `/shadcn`. |
| [`emil-design-eng`](skills/emil-design-eng/SKILL.md) | Animation timing and interaction detail. |

Not covered here: diagnosing bugs, tickets, a dedicated implement step, strict TDD. [mattpocock/skills](https://github.com/mattpocock/skills) has `diagnosing-bugs`, `implement`, `tdd`, `to-spec`, and `to-tickets` if you want them.

## Installation

Each skill is a plain folder with a `SKILL.md`, with no build step.

```bash
# one skill, into the current project
npx skills@latest add AmirAbaris/amir-skills --skill grill-with-docs

# everything in this repo
npx skills@latest add AmirAbaris/amir-skills --all
```

Or copy the folder into `.claude/skills/`.

`grill-with-docs` writes domain docs, so a new repo needs to say where they go. Usually that is a `CONTEXT.md` and `docs/adr/` at the root, written down once as `docs/agents/domain.md`.

## Attribution & licensing

Mostly a curation, not original work. Every skill folder is reproduced from one of these sources, and nothing is relicensed beyond what its author already granted.

| Skill(s) | Source | License |
|---|---|---|
| `code-review`, `codebase-design`, `domain-modeling`, `grill-with-docs` | [mattpocock/skills](https://github.com/mattpocock/skills) | MIT |
| `emil-design-eng` | [emilkowalski/skills](https://github.com/emilkowalski/skills) | MIT |
| `shadcn` | [shadcn-ui/ui](https://github.com/shadcn-ui/ui) | MIT |
| `unslop` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills), itself vendored from [cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) | MIT (see [skills/unslop/LICENSE](skills/unslop/LICENSE)) |
| `before-and-after` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills), itself vendored from [vercel-labs/before-and-after](https://github.com/vercel-labs/before-and-after) | PolyForm Shield 1.0.0 (see [skills/before-and-after/LICENSE](skills/before-and-after/LICENSE)) |
| `new-feature`, `evidence-driven-testing` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills) | No license file published upstream at time of writing — reproduced here with attribution; check the source repo before reusing outside a personal workflow |

Deliberately excluded: `greploop` and `greploop-apps` (also michaelshimeles/skills) — both need a paid Greptile subscription. `code-review` is the substitute.

The curation and this README are [MIT](LICENSE).
