# Plan 3 - documentation (R23, R31, R33)

> For agentic workers: use superpowers:executing-plans. Progress in `progress.md`, `## plan3.md`.
> Prerequisite: `plan2.md` fully ticked. Docs describe what was built, never more (R14, R15).

**Goal:** every document that names the web target, the slide kinds or the changed CLIs
says what the code now does.
**Spec:** `spec.md`. Examples use fictitious identities and relative paths only (R34).

## Files (7)

1. `.claude/skills/paper2talk/SKILL.md`
2. `.claude/skills/paper2talk/references/renderer-contracts.md`
3. `.claude/agents/talk-builder.md`
4. `.claude/commands/talk.md`
5. `.claude/rules/testing.md`
6. `docs/manual/04-skills.md`
7. `README.md` (only if its paper2talk capability line changes)

## Task 9 - skill and contract docs

**Files:** `SKILL.md`, `references/renderer-contracts.md`.
- [ ] `SKILL.md`: the web target section states the lab identity, `--lab-dir` (the gabarit
  folder, relative path as already documented in section 3), the `outline` kind and its
  divider timing, figure embedding, the stricter offline check and `--open`. Keep the
  frontmatter `description` unchanged so the Codex mirror does not change.
- [ ] `renderer-contracts.md`: add `outline` to the slide kinds, the `KIND_TIER` rule, the
  `supported-kinds` and `requires` header lines, and the web context (`lab`, `pacing`, `web`).

## Task 10 - agent and command

**Files:** `talk-builder.md`, `talk.md`.
- [ ] `talk-builder.md`: the web build passes `--lab-dir`; refuses and asks for the path when
  the folder is missing; runs `talk_validate.py --web <deck> --open` unless the operator opted
  out. Keep the frontmatter `description` unchanged.
- [ ] `talk.md`: document `--no-open` and the lab-folder requirement for `--target web`.

## Task 11 - inventories and manual

**Files:** `testing.md`, `docs/manual/04-skills.md`, `README.md`.
- [ ] `testing.md` (R23): update the `paper2talk` script-surface line for `talk_model.py`
  (`--lab-dir`, embedding, `outline`) and `talk_validate.py` (`--open`, relative references);
  add the `test_web_skeleton.py` line and the new test counts to the offline-test block.
- [ ] `docs/manual/04-skills.md` (R33): extend the paper2talk entry with the LARI web deck.
- [ ] `README.md`: update only if its paper2talk line describes the web target.
- [ ] Verify every link added or changed resolves.
