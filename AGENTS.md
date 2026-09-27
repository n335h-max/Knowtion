# Instructions for coding agents

This file governs any AI coding agent (Claude Code, Copilot, or otherwise) committing
to this repository. Human contributors follow CONTRIBUTING.md instead — read that too
for the clean-room rule, dependency policy, and determinism rules, which apply to
agents exactly as they do to humans.

## Commit and push per change

- Commit each logical change as soon as it's done, with a descriptive commit message
  (what changed and why, not which files) — conventional subjects: feat, fix, refactor,
  test, docs, chore, perf. Don't batch multiple unrelated changes into one commit, and
  don't sit on committed work without pushing it.
- **One logical change per commit.** A commit should be revertable on its own.
- **Commit when green:** change, then `npm run verify`, then commit. It runs exactly what
  CI runs, minus the end-to-end smoke test, so green locally means green on push. Never
  commit a failing tree. If it objects to formatting, `npm run format` rewrites the files.
- Never use `--no-verify`. A failing hook is a bug to fix, not to skip.

## Branch and merge

- Knowtion's main build is authored solely by the project owner and AI coding agents —
  there is no external-contributor review process to protect main from, so push commits
  directly to `main`. Branches and PRs are not required.
- CI still runs on every push to `main` (typecheck, lint, format, the licence gate,
  tests). It's a safety net, not a merge gate: if it goes red after a push, the very
  next commit should fix or revert it, not leave main broken.
- **Look at the run.** `npm run verify` passing locally is not evidence that CI passed.
  CI runs on Linux and macOS as well as Windows, and with coverage; a test that depends
  on machine speed, path separators or line endings can be green on one machine for days
  while main is red. Check with `gh run list --workflow=ci.yml --limit 3` after pushing.

## Two tiers of care

Not every file earns the same effort, and pretending otherwise is why interface work has
cost as much as the merge model. Decide which tier a change is in before writing it.

**Load-bearing** — `packages/engine`, `packages/format`, `packages/sync`,
`packages/readmodel`, and anything touching the CRDT, the on-disk format, encryption, or
the two query interpreters. A mistake here corrupts data or diverges two devices, and the
log cannot be un-written. These keep the full standard: a comment that explains why, an
ADR when the decision is expensive to reverse, property tests where the input space is
large, and a case in the shared table for anything both interpreters must agree on.

**Interface** — `apps/desktop/src/renderer`, styling, wording, and the host code that only
marshals between them. A mistake here is visible, reversible and costs one commit. These
get a test for the logic worth testing, a short comment where the reason is not obvious,
and no ADR. Do not write an essay above a colour picker.

The tiers are about **where the cost of being wrong lands**, not about how interesting the
code is. When a change spans both, the load-bearing half sets the bar for that half only.

## Release gates

`npm run gates` runs the v0.2 milestone gates — the numbers the release is defined by,
too slow to sit in `npm run verify`. Run it before cutting a release, and when changing
anything on the merge or storage path. A scaled-down version of each gate runs in the
normal suite so a gross regression still surfaces on every push.

## Format and ADR changes

Anything touching the on-disk or on-cloud format still needs an ADR, a FORMAT.md
update, and a golden-file fixture, per CONTRIBUTING.md. Knowtion has no backend, so a
format mistake ships to users permanently — don't shortcut this because an agent is
making the change quickly.
