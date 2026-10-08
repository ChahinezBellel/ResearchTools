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

- [x] Task 5 - template context: pacing, web constants, footer check (2026-10-08). 13 tests
  in `test_talk_model.py`: `TestTemplateContext` (6) and `TestFooterFields` (7), fictitious
  data. RED: 49 -> 62 tests, failures=3 errors=7 (missing `WEB_INTERACTION`,
  `FOOTER_FIELD_MAX_CHARS`, `wpm`; `pacing` Undefined in the context; no footer report); one
  probe bug fixed first (it wrote `.tex`, which switches Jinja delimiters). GREEN:
  `talk_rules.WEB_INTERACTION = {"swipe_threshold_px": 40}` and `FOOTER_FIELD_MAX_CHARS = 48`
  (operator, 2026-10-08); `talk_model.slide_pacing` (budget_rows seconds, aligned with the
  model's own slide order); `render_template(..., wpm=)` adds `pacing` and `web`; CLI passes
  `--wpm`; `_footer_problems` in `validate_model` checks the shown field (short form when
  given) against the cap. Results: model 62 OK, validate 32 OK, notes 14 OK, 8 other suites
  OK, 3 runnable pptx outline tests OK. The web template does not consume `pacing`/`web`
  yet (Tasks 6-7).
- [x] Task 6 - lab layouts per kind (2026-10-08). New `Test/test_web_skeleton.py`, 17 tests,
  real template, generated images, fictitious metadata; imports jinja2 at the top so the
  runner reports it NOT RUN without it. RED: 16 tests, failures=12 errors=1 (no `#006341`,
  `supported-kinds` None, no `layout-<kind>`/`frametitle`/`footline`, no banner or logo,
  `Cambria` present, no alert block, no outline list, no `1 / 1`); escaping, no-external-ref
  and offline-check controls passed. `web_skeleton.html.j2` rewritten: header declares
  `supported-kinds` (six) and `requires: lab-assets`; gabarit colour tokens, brand default
  `#006341`, retired `5A7210` fallbacks gone; sans-serif only; per-kind layouts (title:
  banner, rounded green box, subtitle, authors, institute, date, logo; outline: numbered
  sections; divider: centred green title, no band; thanks: large green headline; backup and
  content: title band); footer row of its own on every NON-title slide (spec.md) with `short_author` or authors
  joined ", " (the Task 5 rule), `short_title` or title, date, `n / N`; figures `data_uri`
  only. One later RED/GREEN pair after a visual check: the closing slide's block drawn as a
  large green headline. Five accepted Task 2/3 web tests in `test_talk_model.py` given a
  generated lab folder (the template now requires it); the two that would otherwise have
  passed on the lab-folder refusal now assert their own message. Results: web 17 OK, model
  62 OK, validate 32 OK, notes 14 OK, 8 other suites OK, 3 runnable pptx outline tests OK.
  Informal visual check (headless Edge print, scratchpad only): 7 pages for 7 slides,
  layouts close to compiled reference pages 1, 2, 3, 5, 12, 21.
  Review correction (2026-10-08): the footer had also been rendered on the title slide.
  Footer test now requires no footer on the title slide and exactly one on every other
  slide (one-slide and footer-row tests adjusted to non-title slides). RED: 17 tests,
  failures=1 (title slide held a footer). GREEN: footer rendered only when kind != "title";
  web 17 OK, model 62 OK, validate 32 OK, notes 14 OK; probe: title 0 footers, the 6 others
  1 each, 7 pages printed, `--web` clean.
- [x] Task 7 - interactions and accessibility (2026-10-08). 13 tests in new class
  `TestInteractions` (`test_web_skeleton.py`), structural markers on the rendered HTML. RED:
  17 -> 30 tests, failures=10 errors=2 (no `DECK` config, guard, counter, progress,
  `slideFromHash`, fullscreen, blackout, help dialog, click/touch handlers, `announce`,
  `pacing`, print rules); reduced-motion already passed. GREEN: `web_skeleton.html.j2` adds
  `DECK` (a `tojson` literal of `web.swipe_threshold_px` and `pacing`, inside the deck's one
  script), typing/modifier guard, counter and progress bar on the viewport (outside the
  stage), `#/n` read on load (clamped 1..N, anything else ignored) and written with
  `replaceState`, `hashchange`, F fullscreen, B/. blackout, `?` help dialog (role dialog,
  aria-modal, Escape closes, focus restored), click advance with an interactive-element and
  text-selection guard, single-touch swipe at the threshold measured at stage scale with a
  multi-touch guard, `aria-live` announcement and `aria-hidden` on inactive slides, pacing in
  the notes panel (elapsed against cumulative `pacing`, clock from the first move), print
  hides the chrome. A browser check found the print path at 8 pages for 7 slides (the
  `#announce` region printed); a new RED/GREEN pair added it to the print-hidden chrome, back
  to 7 pages. Results: web 30 OK, model 62 OK, validate 32 OK, notes 14 OK, 8 other suites
  OK, 3 runnable pptx outline tests OK; probe `--web` clean, title 0 footers, others 1.
  Browser smoke (installed Edge, headless `--dump-dom`): no hash -> 1, `#/4` -> 4 (`4 / 7`,
  bar 50%), `#/99` -> 7, `#/0` -> 1, `#/abc` and `#/-2` ignored -> 1; announce text updates.
  Keyboard, click and touch not exercised in the browser (no automation tool). Found,
  pre-existing at e2f24ef and not fixed: in a window smaller than the stage the scaled
  stage is offset and cropped (grid centring of an oversized item), unchanged since then.
  Post-review responsive fix (2026-10-08): root cause, `#viewport` centred `#stage` with
  CSS grid; a grid item larger than its track starts at the container's corner, so after
  `scale()` about its centre the stage landed at 960*(1-s), 540*(1-s) (measured in Edge:
  left 403 at 1280x720, right edge 1517 past a 1254 px viewport; cropped at all four sizes
  tested). New class `TestStageFit` (4 tests). RED: 34 tests, failures=3 (stage not
  absolutely centred, `fit()` without the -50% translate, print not resetting position).
  GREEN: `#stage` absolute at left/top 50%, `fit()` applies `translate(-50%, -50%)
  scale(min(vw/1920, vh/1080))`, print sets `position: static`; canvas still 1920x1080.
  Edge measurements at `#/4` (stage inside the viewport and centred within 1 px, ratio
  1.778, slide 4, `4 / 7`, bar 50%): 1920x1080 window (viewport 1894x987) 70,0 1755x987;
  1280x720 (1254x627) 70,0 1115x627; 1366x768 (1340x675) 70,0 1200x675; 1024x768
  (998x675) 0,57 998x561. Print 7 pages; probe `--web` clean, title 0 footers + logo,
  others 1 footer. Web 34 OK, model 62 OK, validate 32 OK, notes 14 OK.
- [x] Task 8 - open only after a clean check (2026-10-08). 7 tests in `test_talk_validate.py`:
  new class `TestOpenAfterCheck` (6, `webbrowser.open` patched in every case) and one
  `--web`-without-`--open` control in `TestMainDispatch`. RED pass 1: 32 -> 39 tests,
  errors=6 (`talk_validate` had no `webbrowser`); RED pass 2 after adding only the import:
  errors=6, `SystemExit: 2` (argparse did not know `--open`); both no-open controls passed.
  GREEN: `--open` flag; after a clean `--web` check `open_in_browser()` hands
  `pathlib.Path(...).as_uri()` to the standard-library `webbrowser.open` once; findings exit
  1 and never open; an unreadable deck exits 2 and never opens; `--open` without `--web` is
  a usage refusal (exit 2); a browser that does not start (`False` or `webbrowser.Error`)
  is reported on stderr with the file to open by hand and leaves exit 0. Suppression is the
  absence of `--open` (the `/talk --no-open` opt-out is documentation, Plan 3). No browser
  is named. Results: validate 39 OK, web 34 OK, model 62 OK, notes 14 OK, 8 other suites OK.
  Integration (real CLI, probe, opener replaced by a recorder, no browser launched): clean
  + `--open` exit 0 opened once (`file:///...deck.html`); deck with one relative `src` +
  `--open` exit 1 opened 0; clean without `--open` opened 0; probe has no http, script src
  or @import.

## plan3.md

- [ ] Task 9 - SKILL.md and renderer-contracts.md
- [ ] Task 10 - talk-builder.md and talk.md
- [ ] Task 11 - testing.md, docs/manual/04-skills.md, README.md

## plan4.md

- [ ] Task 12 - mirrors regenerated
- [ ] Task 13 - full suite, offline render, manual visual check
- [ ] Task 14 - R35 review loop clean, rounds recorded
