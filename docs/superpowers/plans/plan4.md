# Plan 4 - mirrors, verification, R35 security gate

> For agentic workers: use superpowers:executing-plans. Progress in `progress.md`, `## plan4.md`.
> Prerequisite: `plan3.md` fully ticked. The R35 review writes `audit.md`, never the executor.

**Goal:** regenerated mirrors committed with their sources, a green suite, a manual visual
check against the reference, and a clean R35 review before the pull request opens.
**Spec:** `spec.md`.

## Files (5 generated or produced, none edited by hand)

1. `.github/agents/talk-builder.agent.md` (generated)
2. `.opencode/agents/talk-builder.md` (generated)
3. `.github/prompts/talk.prompt.md` (generated)
4. `.github/instructions/testing.instructions.md` (generated)
5. `docs/superpowers/plans/audit.md` (written by the reviewing pass)

## Task 12 - regenerate mirrors

- [ ] Run `.\install.ps1 -Profile engineering`. Expect changes only in the four generated
  files above. If `.agents/skills/paper2talk/SKILL.md`, `AGENTS.md` or `CONVENTIONS.md`
  change, stop: the frontmatter description was altered by mistake.

## Task 13 - verification

- [ ] Run `.\scripts\test\run-offline-tests.ps1`. Expect the baseline (100 passed, 5 not run)
  plus the new tests, 0 failed.
- [ ] Render a fixture deck with figures through `talk_model.py --render` with `--lab-dir`,
  then `talk_validate.py --web` with the network off: clean.
- [ ] Open it in a browser with the network off. Compare by slide type with the reference
  pages listed in `spec.md`. Record the result per slide type in `progress.md`. Check the
  four Beamer defects are absent, the logo appears on the title slide only, and every key in
  the help overlay works. Print to PDF once: one page per slide, footer present, no chrome.

## Task 14 - R35 security and code review loop

- [ ] Run `/security-review` and `/code-review` at high effort on the full branch diff against
  `main`. Ask explicitly who can bypass the stricter `--web` check.
- [ ] For each finding: a test that fails on the current code, then the fix; or a decline with
  a measurement in `audit.md`.
- [ ] Re-run until a round returns no finding. Record every round and its fixes; the PR body
  repeats them (R35, R30).
