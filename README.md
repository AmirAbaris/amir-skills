# amir-skills

My working set of [Claude Code agent skills](https://code.claude.com/docs/en/skills) and the workflow that ties them together. Curated (not authored from scratch) from a few public skill collections — see [Attribution & licensing](#attribution--licensing) below — with one thing removed on purpose: nothing here depends on a paid third-party review service. The review gate is a skill in this repo (`code-review`), not a subscription.

The problem this setup solves: agent-driven "do everything, hand me one PR" workflows tend to produce a single large, hard-to-review, still-buggy change. Every skill below exists to keep a human decision point between "idea" and "merged code" — smaller slices, an approval step before building, and a review pass before every commit.

## Two kinds of skills

Every `SKILL.md` has a `description` Claude Code matches against the current task. Most skills can fire on their own the moment the task looks right — you never type their name. A few are marked `disable-model-invocation: true`: those only run when you explicitly type `/skill-name`, because they make a decision (start a spec, start implementing) that shouldn't happen without you asking for it.

**Call these yourself — they don't fire on their own:**

| Skill | Call it when |
|---|---|
| [`grill-with-docs`](skills/grill-with-docs/SKILL.md) | You have a rough idea and want it interrogated before anything gets built. Sharpens the plan and writes `CONTEXT.md`/ADRs as decisions land. |
| [`to-spec`](skills/to-spec/SKILL.md) | The grilling conversation has settled and you want it turned into a spec, published as a GitHub issue. |
| [`to-tickets`](skills/to-tickets/SKILL.md) | You have a spec (or a big enough conversation) and want it split into small, independently buildable tickets. |
| [`implement`](skills/implement/SKILL.md) | You have a spec or ticket and want it built: TDD where it fits, `code-review` before commit. |
| [`setup-matt-pocock-skills`](skills/setup-matt-pocock-skills/SKILL.md) | Once, per repo, before first use of the above — picks the issue tracker (GitHub/GitLab/local files) and domain-doc layout the rest assume. |

**Leave alone — Claude reaches for these itself when the task matches:**

| Skill | Fires when |
|---|---|
| [`new-feature`](skills/new-feature/SKILL.md) | The start of any new task — puts it on its own worktree/branch off `origin/main` instead of touching `main`. |
| [`code-review`](skills/code-review/SKILL.md) | You ask to review a branch, a PR, or work-in-progress — checks Standards (repo conventions) and Spec (does it match the issue) as two parallel, separately-reported passes. |
| [`codebase-design`](skills/codebase-design/SKILL.md) | A module's interface is being designed or reworked — vocabulary for "deep modules" (small interface, real behavior behind it). |
| [`domain-modeling`](skills/domain-modeling/SKILL.md) | Project terminology comes up, or `CONTEXT.md`/an ADR needs writing or challenging. |
| [`before-and-after`](skills/before-and-after/SKILL.md) | A UI change needs a quick before/after screenshot pair for the PR. |
| [`evidence-driven-testing`](skills/evidence-driven-testing/SKILL.md) | A change needs proof it actually works — a recorded, narrated walkthrough with pass/fail assertions, not a prose claim. |
| [`unslop`](skills/unslop/SKILL.md) | Right before a commit message, PR body, or doc edit gets written — strips AI-tell phrasing. |
| [`shadcn`](skills/shadcn/SKILL.md) | Working with shadcn/ui components, registries, or `components.json`. |
| [`emil-design-eng`](skills/emil-design-eng/SKILL.md) | Polishing UI feel — animation timing, interaction detail, the stuff that separates "functional" from "good". |

Everything in the second table can also be called explicitly (`/code-review`, `/unslop`, …) — "leave alone" means you don't have to, not that you can't.

## The workflow

```
/new-feature ──▶ /grill-with-docs ──▶ /to-spec ──▶ /to-tickets ──▶ /implement (per ticket)
   (auto)          idea → decisions      spec on         N small          TDD + code-review
                    + CONTEXT.md/ADR     the tracker      tickets          + before/after on PR
```

### Worked example: "add a newsletter signup form to the landing page"

1. **Start the task.** You just start talking about the feature; `new-feature` fires on its own and puts you on a fresh branch off `origin/main`. You never touch `main` directly.
2. **`/grill-with-docs`** — a real conversation, not a form. It asks what "signup" means here (just an email? confirmation flow? where does it get stored?), what happens on duplicate emails, what the empty/error/success states look like. As terms get pinned down ("subscriber" vs "lead"), it writes them into `CONTEXT.md` inline. If a decision is hard to reverse (e.g. "store emails in the existing `contacts` table, not a new one"), it offers an ADR.
3. **`/to-spec`** — once the conversation feels settled, this turns it into a spec (problem, solution, user stories, implementation + testing decisions) and files it as a GitHub issue. No re-interview, just synthesis.
4. **`/to-tickets`** — splits the spec into tracer-bullet tickets, each a complete vertical slice you could demo on its own, e.g.:
   - `01: signup form renders and validates client-side` (blocked by nothing)
   - `02: submit persists a subscriber row + returns success/duplicate state` (blocked by nothing)
   - `03: form wires to the endpoint end-to-end, success/error states visible` (blocked by 01, 02)

   You see this breakdown before anything gets built and can merge, split, or reorder tickets. **This is the step that replaces "one giant PR" with a plan you actually signed off on.**
5. **`/implement`**, once per ticket, in a fresh context each time. Each run drives TDD where it fits, runs `code-review` (Standards + Spec, two separate reports) before committing, and stops — it doesn't cascade into the next ticket unattended.
6. **Shipping.** `unslop` cleans up the commit message and PR body before they're written. `before-and-after` grabs a screenshot pair of the form for the PR description; for the submit flow specifically (state that's easy to get subtly wrong — duplicate handling, error states) `evidence-driven-testing` records an actual click-through with pass/fail assertions instead of a prose "tested it" claim.

### Smaller changes skip steps

A one-line copy fix or a small bug doesn't need a spec or tickets: `new-feature` still isolates it on its own branch, then go straight to `/implement` in the same conversation. `to-spec`/`to-tickets` exist for anything too big to hold in one working session, not as mandatory ceremony.

### Fixing something broken

Not covered by the skills in this repo — `diagnosing-bugs` from [mattpocock/skills](https://github.com/mattpocock/skills) is the on-ramp for that (reproduce, get a tight red-going-green feedback loop, then fix with a regression test) and hands off to `implement` the same way a ticket would.

## Installation

Each skill is a plain folder with a `SKILL.md` — no build step. Either:

```bash
# one skill, into the current project
npx skills@latest add AmirAbaris/amir-skills --skill to-tickets

# everything in this repo
npx skills@latest add AmirAbaris/amir-skills --all
```

or copy the folder directly into `.claude/skills/` (or wherever your agent looks for skills).

`setup-matt-pocock-skills` expects to run once per repo before `grill-with-docs` / `to-spec` / `to-tickets` / `implement` are used — it writes `docs/agents/issue-tracker.md` and `docs/agents/domain.md` so those skills know where issues and domain docs live in *that* repo.

## Attribution & licensing

This repo is a personal curation, not original work for most of it. Every skill folder is reproduced from one of these sources; nothing here is relicensed beyond what its author already granted.

| Skill(s) | Source | License |
|---|---|---|
| `code-review`, `codebase-design`, `domain-modeling`, `grill-with-docs`, `implement`, `to-spec`, `to-tickets`, `setup-matt-pocock-skills` | [mattpocock/skills](https://github.com/mattpocock/skills) | MIT |
| `emil-design-eng` | [emilkowalski/skills](https://github.com/emilkowalski/skills) | MIT |
| `shadcn` | [shadcn-ui/ui](https://github.com/shadcn-ui/ui) | MIT |
| `unslop` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills), itself vendored from [cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) | MIT (see [skills/unslop/LICENSE](skills/unslop/LICENSE)) |
| `before-and-after` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills), itself vendored from [vercel-labs/before-and-after](https://github.com/vercel-labs/before-and-after) | PolyForm Shield 1.0.0 (see [skills/before-and-after/LICENSE](skills/before-and-after/LICENSE)) |
| `new-feature`, `evidence-driven-testing` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills) | No license file published upstream at time of writing — reproduced here with attribution; check the source repo before reusing outside a personal workflow |

Deliberately excluded from this collection: `greploop` and `greploop-apps` (also from michaelshimeles/skills) — both require a paid Greptile subscription to do anything. `code-review` above is the substitute: a review gate that doesn't need one.

My own contribution here is the curation, the README, and the workflow description above — not the skill contents themselves. That part is [MIT](LICENSE).
