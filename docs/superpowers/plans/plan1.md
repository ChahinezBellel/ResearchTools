# Plan 1 - offline validation, asset embedding, lab assets, outline kind

> For agentic workers: use superpowers:executing-plans or superpowers:subagent-driven-development.
> Steps use `- [ ]`. Progress is recorded in `progress.md`, section `## plan1.md`.

**Goal:** make the web deck truly self-contained and give the model an `outline` kind, before
any visual work.
**Architecture:** extend `talk_validate.check_web`, add asset embedding and lab-asset loading
to `talk_model.render_template`, add a kind-to-tier map in `talk_rules`, and teach the
PowerPoint, Beamer and notes paths about `outline`.
**Spec:** `spec.md`. **Tech:** Python standard library, Jinja2, python-pptx (already pinned).

## Files (10)

1. `.claude/skills/paper2talk/scripts/talk_rules.py`
2. `.claude/skills/paper2talk/scripts/talk_model.py`
3. `.claude/skills/paper2talk/scripts/talk_validate.py`
4. `.claude/skills/paper2talk/scripts/talk_notes.py`
5. `.claude/skills/paper2talk/scripts/talk_pptx.py`
6. `.claude/skills/paper2talk/assets/beamer_skeleton.tex.j2`
7. `.claude/skills/paper2talk/scripts/Test/test_talk_validate.py`
8. `.claude/skills/paper2talk/scripts/Test/test_talk_model.py`
9. `.claude/skills/paper2talk/scripts/Test/test_talk_notes.py`
10. `.claude/skills/paper2talk/scripts/Test/test_talk_pptx.py`

Run each suite with the project Python, e.g.
`python .claude/skills/paper2talk/scripts/Test/test_talk_model.py`.

## Review focus (inputs no step exists for by default; each gets a test below)

- An image file whose extension lies about its content (a `.png` that is HTML).
- An asset path using `..`, an absolute path, or a Windows drive-relative form.
- A model with no `section` values at all, rendered with an `outline` slide.
- The template's own inline script text containing the letters `src=` (false positive).
- A gabarit folder that exists but lacks one of the two images.

## Task 0 - confirm values (no file changes)

- [x] Ask the operator to confirm the four values in `spec.md` "Values confirmed by the
  operator". Record the answers in `progress.md`. No constant is written before this.
  Done 2026-10-08; values in `spec.md` and `progress.md`.

## Task 1 - `check_web` flags relative references

**Files:** `talk_validate.py`, `Test/test_talk_validate.py`.
**Produces:** `check_web(path) -> list[str]` now reports any `src`, `href` or CSS `url()`
whose value is not a `data:` URI, a fragment `#...` or a `mailto:` link. Existing `http(s)`
reporting is unchanged.

- [ ] In `Test/test_talk_validate.py`, class `TestWebTarget`, add failing tests:
  a relative `<img src="fig.png">` is reported; a relative `<link href="x.css">` is reported;
  `url(img/a.png)` with and without quotes is reported; `data:`, `#slide-3` and `mailto:` are
  not reported (negative control); a line of inline script mentioning `src=` inside a string
  that is not an attribute is not reported (review focus).
- [ ] Run the suite: the new tests fail, the existing four web tests still pass.
- [ ] In `talk_validate.py`, extend the reference detection so the tests pass. Keep the
  message naming the line and the offending reference.
- [ ] Run the suite: all pass.

## Task 2 - figures embedded as data URIs for HTML output

**Files:** `talk_rules.py`, `talk_model.py`, `Test/test_talk_model.py`.
**Produces:**
- `talk_rules.WEB_ASSET_MAX_BYTES` and `talk_rules.IMAGE_SIGNATURES` (MIME by leading bytes:
  PNG, JPEG, GIF, SVG), each with a provenance comment.
- `talk_model.AssetError(ValueError)`.
- `talk_model.embed_asset(path: str, root: str, max_bytes: int) -> str`: resolves `path`
  against `root`, refuses any result outside `root` (R24, resolve before compare), refuses a
  missing file, a file over `max_bytes`, or an unrecognised signature; returns
  `data:<mime>;base64,<...>`.
- `render_template(model, template_path, out_path, *, model_dir: str | None = None, ...)`:
  for `.html` output, every `figure` block gets `data_uri` from `embed_asset(asset, model_dir)`.
  For any other output the model is untouched.

- [ ] Add failing tests in a new class `TestAssetEmbedding`: a generated PNG becomes a
  `data:image/png;base64` URI; a missing file raises `AssetError` naming it; `..`, an absolute
  path outside the root and a drive-relative path are refused (review focus); a `.png` whose
  bytes are HTML is refused (review focus); a file over the cap is refused; a `.tex` render
  keeps the plain asset path (negative control).
- [ ] Run: new tests fail.
- [ ] Implement in `talk_rules.py` and `talk_model.py`; `main()` passes the model file's
  folder as `model_dir` and maps `AssetError` to exit 2.
- [ ] Run: all pass.

## Task 3 - lab assets via `--lab-dir`

**Files:** `talk_rules.py`, `talk_model.py`, `Test/test_talk_model.py`.
**Produces:**
- `talk_rules.LAB_ASSET_FILES`: role to filename (`banner`, `logo`).
- `talk_model.template_requirements(template_path) -> dict`: parses the template header
  lines `supported-blocks`, `supported-kinds` and `requires`, using the same comment
  convention as the existing `supported-blocks` line. `supported_blocks()` keeps its contract.
- `talk_model.load_lab_assets(lab_dir: str) -> dict[str, str]`: role to data URI, through
  `embed_asset`, refusing a missing folder or file by name.
- `render_template(..., lab_dir: str | None = None)`: when the template requires
  `lab-assets`, a missing `lab_dir` raises `AssetError`; the context gains `lab`.
- CLI `talk_model.py --lab-dir DIR`.

- [ ] Add failing tests `TestLabAssets`: a requiring template without `--lab-dir` exits 2
  through `main()`; a folder lacking the logo exits 2 naming the logo (review focus); with a
  generated banner and logo, the context holds two data URIs; a template without the
  `requires` line renders without the flag (negative control).
- [ ] Run: fail. Implement. Run: pass.

## Task 4 - `outline` kind with divider timing

**Files:** `talk_rules.py`, `talk_model.py`, `talk_notes.py`, `talk_pptx.py`,
`beamer_skeleton.tex.j2`, `Test/test_talk_model.py`, `Test/test_talk_notes.py`,
`Test/test_talk_pptx.py`.
**Produces:**
- `talk_rules.KIND_TIER = {"outline": "divider"}`, `talk_rules.KINDS` (every tier plus every
  key of `KIND_TIER`), `talk_rules.tier_of(kind: str) -> str` (raises `ValueError` on an
  unknown kind). `TIER_MINUTES` is unchanged.
- `talk_model.slide_kind(slide) -> str` validated against `KINDS`; `slide_tier(slide)` returns
  `tier_of(slide_kind(slide))`. `assert_renderable` also checks slide kinds against the
  template's `supported-kinds` and raises `RendererGap` on a missing one.
- `talk_notes` builds its tier sets through `tier_of`, with no literal tier list.
- `talk_pptx`: an `outline` slide prefers a layout matching `programme`, `sommaire`, `outline`
  or `agenda`, then falls back to the content layouts; its body lists the model's sections.
- `beamer_skeleton.tex.j2`: declares `supported-kinds`; an `outline` frame lists the model's
  distinct `section` values in order.

- [ ] Add failing tests: `slide_tier` of an `outline` slide is `divider` and its budget equals
  a divider's; `tier_of` on an unknown kind raises; a template whose `supported-kinds` lacks
  `outline` raises `RendererGap`; model and `talk_notes` budgets agree on a deck containing an
  outline; a fixture deck with a `Programme` layout uses it and one without falls back; the
  Beamer render lists sections; an outline on a model with no sections renders an empty list
  without error and `validate_model` reports it (review focus).
- [ ] Run: fail. Implement. Run: pass. Then run every paper2talk suite.
