# Progress - Issue #53, paper2talk web deck LARI MVP

Marks: `- [ ]` not done, `- [x]` done with a short result, `- [!]` blocked with one line of
reason. One section per plan file. Updated by the executing model only.

## plan1.md

- [x] Task 0 - operator confirmed the four values on 2026-10-08 (provenance: chosen by the
  operator, R4): swipe threshold 40 px; maximum embedded asset size 10 MiB per file; footer
  field cap 48 characters; lab asset filenames `en-tete1260.jpg` (banner) and
  `LogoLARI_FINAL.png` (logo). Execution method approved: native, sequential.
- [x] Task 1 - check_web flags relative references (2026-10-08). 7 tests added to
  `TestWebTarget` in `test_talk_validate.py`. RED: 25 -> 32 tests, 6 failures (relative img,
  relative stylesheet, relative CSS url() x3 quoting forms, protocol-relative `//`); the 3
  negative controls (data:/#/mailto: pass, `src=` in script text ignored, external URL
  reported once) passed before and after. GREEN: `talk_validate.py` adds `_RE_ATTR_REF`,
  `_RE_CSS_URL`, `_INLINE_PREFIXES`, `_EXTERNAL_PREFIXES` and a second pass in `check_web`;
  http(s) keeps its original message. `test_talk_validate.py`: Ran 32, OK. Other paper2talk
  suites unchanged (test_talk_pptx / test_to_a4 error only on missing `pptx` / `pypdf`,
  pre-existing, NOT RUN under the suite runner). Known effect: until Task 2, a rendered web
  deck whose figure has no `data_uri` is now reported, by design.
- [x] Task 2 - figures embedded as data URIs for HTML output (2026-10-08). 12 tests added
  in `TestAssetEmbedding` (`test_talk_model.py`), fictitious fixtures only. RED: 13 -> 25
  tests, failures=1 errors=15 (missing `AssetError`, `embed_asset`, `WEB_ASSET_MAX_BYTES`,
  `model_dir`; CLI returned 0 instead of 2). GREEN: `talk_rules.py` adds
  `WEB_ASSET_MAX_BYTES` (10 MiB, operator), `IMAGE_SIGNATURES`, `SVG_MIME`; `talk_model.py`
  adds `AssetError`, `_image_mime`, `embed_asset` (resolve then contain, R24),
  `_embed_figures` (deep copy) and `render_template(..., model_dir=)`; `main()` passes the
  model's folder and maps `AssetError` to exit 2. `test_talk_model.py` Ran 25 OK;
  `test_talk_validate.py` (Task 1) Ran 32 OK. Probe replay: missing figure -> exit 2 naming
  it; present figure -> data:image/png in the deck and `--web` clean.
- [x] Task 3 - lab assets via --lab-dir (2026-10-08). 9 tests in new class `TestLabAssets`
  plus 2 Task 2 edge-case tests in `TestAssetEmbedding` (`test_talk_model.py`), fictitious
  fixtures generated in the tests, no lab asset copied into the repository. RED pass 1:
  25 -> 36 tests, errors=11 (9 at setUp on missing `LAB_ASSET_FILES`; 2 `KeyError: 'asset'`).
  RED pass 2, after adding only the data constant: errors=9 (`template_requirements` and
  `load_lab_assets` missing; argparse `SystemExit: 2` on unknown `--lab-dir`; Jinja
  `UndefinedError: 'lab'` because `requires` was not enforced; the 2 `KeyError`s). GREEN:
  `talk_rules.LAB_ASSET_FILES` (banner `en-tete1260.jpg`, logo `LogoLARI_FINAL.png`,
  operator); `talk_model.template_requirements` (header comment lines, both delimiters),
  `supported_blocks` now delegates to it with its contract unchanged; `load_lab_assets`
  through `embed_asset`; `render_template(..., lab_dir=)` adds `lab` only for a template
  declaring `requires: lab-assets` and refuses with `AssetError` without it; CLI
  `--lab-dir`. Edge case: a web figure with no `asset` now raises `AssetError` naming the
  slide (CLI exit 2) instead of `KeyError`. One test corrected during GREEN: it expected exit
  0, but the fixture's text-only slide is a content rule (exit 1, by design); it now asserts
  "not refused" (exit != 2) plus both data URIs. Results: `test_talk_model.py` 36 OK,
  `test_talk_validate.py` 32 OK; real web and Beamer templates declare no requirement and
  render as before.
- [x] Task 4 - outline kind with divider timing (2026-10-08). Tests: `test_talk_model.py`
  new class `TestOutlineKind` (12) plus a budget-agreement test with an outline;
  `test_talk_notes.py` 1; `test_talk_pptx.py` 3 layout tests on a fake presentation (run
  without python-pptx) plus 1 render test that needs python-pptx. RED: model 36 -> 49 tests,
  failures=3 errors=7 (missing `KIND_TIER`, `KINDS`, `tier_of`, `slide_kind`,
  `outline_sections`; `ValueError ... declares tier 'outline'`; RendererGap not raised;
  Beamer sections absent; no missing-section report); notes 1 failure; pptx 5 layout
  failures. One RED test was tightened first: it passed on the old unknown-kind message.
  GREEN: `talk_rules.KIND_TIER = {"outline": "divider"}`, `KINDS`, `tier_of()`; 1/3 min
  stays only in `TIER_MINUTES` (single literal, `talk_rules.py`). `talk_model.slide_kind`,
  `slide_tier = tier_of(slide_kind)`, `outline_sections`, an outline with no section is
  reported, `assert_renderable(..., kinds)` honours `supported-kinds`, context gains
  `sections`. `talk_notes.tiers_from_model` uses `rules.TIERS` and `tier_of`. `talk_pptx`:
  outline patterns (programme, sommaire, outline, agenda, then content), layout chosen by
  kind, body lists the sections. `beamer_skeleton.tex.j2`: `supported-kinds` line and an
  outline frame listing sections, no list when there is none. Two wording fixes during GREEN
  (a test assertion and a template comment). Results: model 49 OK, notes 14 OK, validate 32
  OK, pptx 3 runnable outline tests OK. BLOCKED: `test_an_outline_slide_lists_the_sections_in_order`
  needs python-pptx (not installed, not installed by design); the pptx render path is
  unverified here.

## plan2.md

- [ ] Task 5 - template context: pacing, web constants, footer check
- [ ] Task 6 - lab layouts per kind
- [ ] Task 7 - interactions and accessibility
- [ ] Task 8 - open only after a clean check

## plan3.md

- [ ] Task 9 - SKILL.md and renderer-contracts.md
- [ ] Task 10 - talk-builder.md and talk.md
- [ ] Task 11 - testing.md, docs/manual/04-skills.md, README.md

## plan4.md

- [ ] Task 12 - mirrors regenerated
- [ ] Task 13 - full suite, offline render, manual visual check
- [ ] Task 14 - R35 review loop clean, rounds recorded
