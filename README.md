# amir-skills

My working set of [Claude Code agent skills](https://code.claude.com/docs/en/skills) and the workflow that ties them together. Curated (not authored from scratch) from a few public skill collections — see [Attribution & licensing](#attribution--licensing) below — with one thing removed on purpose: nothing here depends on a paid third-party review service. The review gate is a skill in this repo (`code-review`), not a subscription.

The problem this setup solves: agent-driven "do everything, hand me one PR" workflows tend to produce a single large, hard-to-review, still-buggy change. This isn't solved with a ticket/tracker pipeline here — no tickets, no issues, GitHub or otherwise, that's not how I work. It's solved with two narrower habits: interrogate the idea *before* any code gets written, and review *every* commit, not just the final diff.

## Two kinds of skills

Every `SKILL.md` has a `description` Claude Code matches against the current task. Most skills can fire on their own the moment the task looks right — you never type their name. A couple are marked `disable-model-invocation: true`: those only run when you explicitly type `/skill-name`, because they make a decision (start implementing) that shouldn't happen without you asking for it.

**Call these yourself — they don't fire on their own:**

| Skill | Call it when |
|---|---|
| [`grill-with-docs`](skills/grill-with-docs/SKILL.md) | You have a rough idea and want it interrogated before anything gets built. Sharpens the plan and writes `CONTEXT.md`/ADRs as decisions land. Start any feature-sized request here. |
| [`implement`](skills/implement/SKILL.md) | The grilling conversation has settled (or the change is small enough to skip grilling) and you want it built: TDD where it fits, `code-review` before commit. |

**Leave alone — Claude reaches for these itself when the task matches:**

| Skill | Fires when |
|---|---|
| [`new-feature`](skills/new-feature/SKILL.md) | The start of any new task — puts it on its own branch off `origin/main` instead of touching `main`. A plain branch, not a git worktree — that only happens if you explicitly ask for one. |
| [`code-review`](skills/code-review/SKILL.md) | You ask to review a branch, a PR, or work-in-progress — checks Standards (repo conventions) and Spec (does it match the issue) as two parallel, separately-reported passes. |
| [`tdd`](skills/tdd/SKILL.md) | Building or fixing something test-first — what makes a test worth keeping, where it should live, red-green-refactor discipline. `implement` calls this internally. |
| [`codebase-design`](skills/codebase-design/SKILL.md) | A module's interface is being designed or reworked — vocabulary for "deep modules" (small interface, real behavior behind it). |
| [`domain-modeling`](skills/domain-modeling/SKILL.md) | Project terminology comes up, or `CONTEXT.md`/an ADR needs writing or challenging. |
| [`before-and-after`](skills/before-and-after/SKILL.md) | A UI change needs a quick before/after screenshot pair for the PR. |
| [`evidence-driven-testing`](skills/evidence-driven-testing/SKILL.md) | A change needs proof it actually works — a recorded, narrated walkthrough with pass/fail assertions, not a prose claim. |
| [`unslop`](skills/unslop/SKILL.md) | Right before a commit message, PR body, or doc edit gets written — strips AI-tell phrasing. |
| [`shadcn`](skills/shadcn/SKILL.md) | Working with shadcn/ui components, registries, or `components.json`. |
| [`emil-design-eng`](skills/emil-design-eng/SKILL.md) | Polishing UI feel — animation timing, interaction detail, the stuff that separates "functional" from "good". |

Everything in the second table except `shadcn` can also be called explicitly (`/code-review`, `/unslop`, …) — "leave alone" means you don't have to, not that you can't. `shadcn` is marked `user-invocable: false` in its own frontmatter: it only ever fires on its own, you can't type `/shadcn`.

## The workflow

```
/grill-with-docs ──▶ /implement
  idea → decisions       TDD + code-review before commit
  + CONTEXT.md/ADR       + before/after evidence on the PR
```

`new-feature` runs underneath both steps automatically — it's not something you type, it's what puts the work on its own branch the instant a new task starts, before either command above runs.

### Worked example: "add a newsletter signup form to the landing page"

1. **You type:** `/grill-with-docs let's add a newsletter signup form to the landing page`. `new-feature` fires invisibly in the same turn, putting you on a fresh branch off `origin/main` — you never touch `main` directly.
2. **The interview.** A real conversation, not a form. It asks what "signup" means here (just an email? confirmation flow? where does it get stored?), what happens on duplicate emails, what the empty/error/success states look like. As terms get pinned down ("subscriber" vs "lead"), it writes them into `CONTEXT.md` inline. If a decision is hard to reverse (e.g. "store emails in the existing `contacts` table, not a new one"), it offers an ADR.
3. **You type:** `/implement`, once the conversation feels settled. It builds the whole thing in this session: `tdd` where it fits, then runs `code-review` (Standards + Spec, two separate reports) before committing. If the feature is genuinely too big for one sitting, say so and agree a smaller first slice to build instead of pushing through — there's no ticket system to lean on here, so that judgment call is yours and mine to make in the conversation, not something a tool decides.
4. **Shipping.** `unslop` cleans up the commit message and PR body before they're written. `before-and-after` grabs a screenshot pair of the form for the PR description; for the submit flow specifically (state that's easy to get subtly wrong — duplicate handling, error states) `evidence-driven-testing` records an actual click-through with pass/fail assertions instead of a prose "tested it" claim.

### Smaller changes skip the interview

A one-line copy fix or a small bug doesn't need `grill-with-docs` — ask for it directly and go straight to `/implement`. `new-feature` still isolates it on its own branch either way.

### Fixing something broken

Not covered by the skills in this repo — `diagnosing-bugs` from [mattpocock/skills](https://github.com/mattpocock/skills) is the on-ramp for that (reproduce, get a tight red-going-green feedback loop, then fix with a regression test) and hands off to `implement` the same way.

### If you do want tickets

I deliberately don't run a ticket/tracker pipeline — see the note at the top. If that's not true for you, mattpocock/skills has `to-spec` (spec from conversation) and `to-tickets` (spec → small tracer-bullet tickets, published to GitHub, GitLab, or local files) as a drop-in extension of the same `grill-with-docs`/`implement` flow. Not included here on purpose.

## Installation

Each skill is a plain folder with a `SKILL.md` — no build step. Either:

```bash
# one skill, into the current project
npx skills@latest add AmirAbaris/amir-skills --skill grill-with-docs

# everything in this repo
npx skills@latest add AmirAbaris/amir-skills --all
```

or copy the folder directly into `.claude/skills/` (or wherever your agent looks for skills).

Before first using `grill-with-docs` / `implement` in a new repo, tell Claude where domain docs go there (usually a single `CONTEXT.md` + `docs/adr/` at the root) and have it write that down as `docs/agents/domain.md` — a one-time, two-minute conversation, not a skill you need installed.

## Attribution & licensing

This repo is a personal curation, not original work for most of it. Every skill folder is reproduced from one of these sources; nothing here is relicensed beyond what its author already granted.

| Skill(s) | Source | License |
|---|---|---|
| `code-review`, `codebase-design`, `domain-modeling`, `grill-with-docs`, `implement`, `tdd` | [mattpocock/skills](https://github.com/mattpocock/skills) | MIT |
| `emil-design-eng` | [emilkowalski/skills](https://github.com/emilkowalski/skills) | MIT |
| `shadcn` | [shadcn-ui/ui](https://github.com/shadcn-ui/ui) | MIT |
| `unslop` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills), itself vendored from [cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop) | MIT (see [skills/unslop/LICENSE](skills/unslop/LICENSE)) |
| `before-and-after` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills), itself vendored from [vercel-labs/before-and-after](https://github.com/vercel-labs/before-and-after) | PolyForm Shield 1.0.0 (see [skills/before-and-after/LICENSE](skills/before-and-after/LICENSE)) |
| `new-feature`, `evidence-driven-testing` | [michaelshimeles/skills](https://github.com/michaelshimeles/skills) | No license file published upstream at time of writing — reproduced here with attribution; check the source repo before reusing outside a personal workflow |

Deliberately excluded from this collection: `greploop` and `greploop-apps` (also from michaelshimeles/skills) — both require a paid Greptile subscription to do anything. `code-review` above is the substitute: a review gate that doesn't need one.

My own contribution here is the curation, the README, and the workflow description above — not the skill contents themselves. That part is [MIT](LICENSE).
