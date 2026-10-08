# Plan 2 - LARI web template, interactions, open after validation

> For agentic workers: use superpowers:executing-plans or superpowers:subagent-driven-development.
> Steps use `- [ ]`. Progress is recorded in `progress.md`, section `## plan2.md`.
> Prerequisite: `plan1.md` fully ticked.

**Goal:** the web deck looks like the compiled LARI Beamer reference and gains the MVP
interactions, all inline and offline.
**Architecture:** the template receives three new context values from `render_template`:
`lab` (plan 1), `pacing` (per-slide seconds) and `web` (interaction constants). It renders
one layout per kind and embeds the constants as a JSON data block read by its own script.
**Spec:** `spec.md`. **Tech:** Jinja2, plain CSS custom properties, vanilla JavaScript.

## Files (7)

1. `.claude/skills/paper2talk/assets/web_skeleton.html.j2`
2. `.claude/skills/paper2talk/scripts/talk_model.py`
3. `.claude/skills/paper2talk/scripts/talk_rules.py`
4. `.claude/skills/paper2talk/scripts/talk_validate.py`
5. `.claude/skills/paper2talk/scripts/Test/test_web_skeleton.py` (new)
6. `.claude/skills/paper2talk/scripts/Test/test_talk_model.py`
7. `.claude/skills/paper2talk/scripts/Test/test_talk_validate.py`

`test_web_skeleton.py` renders a fixture model through the real template with generated
images and fictitious authors (R34), then asserts on the HTML text. It needs Jinja2 and must
report NOT RUN cleanly through the suite runner when Jinja2 is absent.

## Review focus

- A title, author or note containing `<`, `&`, `</script>` or quotes.
- A URL hash that is not a number, is zero, negative, or past the last slide.
- A deck with a single slide (progress and counter at their limits).
- A very long title with no `meta.short_title` (footer clipping).
- Printing: interaction chrome must not appear on paper; the footer must.

## Task 5 - context for the template

**Files:** `talk_rules.py`, `talk_model.py`, `Test/test_talk_model.py`.
**Produces:**
- `talk_rules.WEB_INTERACTION`: a small dict of the confirmed interaction constants (swipe
  threshold) and `talk_rules.FOOTER_FIELD_MAX_CHARS`, each with provenance (R4).
- `render_template` adds `pacing` (list of `budget_rows()` seconds in slide order) and `web`
  (`WEB_INTERACTION`) to the context.
- `validate_model` reports a footer field (title or authors) over `FOOTER_FIELD_MAX_CHARS`
  when no `meta.short_title` or `meta.short_author` replaces it.

- [ ] Failing tests: `pacing` equals `budget_rows()` seconds for a three-slide fixture;
  `web` carries the swipe threshold; changing the constant in `talk_rules` changes the
  rendered value (proves it is not hard-coded); a long title without a short form is
  reported, and the same title with `short_title` is not (review focus).
- [ ] Run: fail. Implement. Run: pass.

## Task 6 - lab layouts per kind

**Files:** `web_skeleton.html.j2`, `Test/test_web_skeleton.py`.
**Produces:** header lines `supported-blocks` (unchanged list), `supported-kinds` (all six)
and `requires: lab-assets`. CSS tokens for the brand and the gabarit colours, defaulting to
`#006341`, with a provenance comment naming the gabarit. Sans-serif face for headings and
body. Title layout with banner, rounded green box, subtitle, authors, institute, `meta.date`
and the logo. Frame-title band on every non-title slide. Distinct `outline` (sections from
the model), `divider`, `thanks` and `backup` layouts. A footer region outside the content
area holding short author, short title, date and `n / N`. Figures use `data_uri` only.

- [ ] Create `Test/test_web_skeleton.py` with failing tests: the default brand is `#006341`;
  the logo data URI appears once and only inside the title slide; the banner appears inside
  the title slide; every non-title slide has the title band; each kind has its own layout
  marker; the footer holds the short fields and `n / N`; no `Cambria`, `Caladea`, `Georgia`
  or `serif` heading face; no `http`, no `<script src`, no `@import`; the rendered file passes
  `talk_validate.check_web`; markup-sensitive text in titles and notes is escaped (review
  focus); a one-slide deck renders (review focus).
- [ ] Run: fail. Rewrite the template layout. Run: pass.

## Task 7 - interactions and accessibility

**Files:** `web_skeleton.html.j2`, `Test/test_web_skeleton.py`.
**Produces:** inline script, own code, reading the JSON config block. Keyboard navigation
ignores events while an input, textarea or editable element has focus and when Ctrl, Alt or
Meta is held. Counter and progress bar follow the current slide. The hash `#/n` is read on
load (clamped to the deck, invalid values ignored) and updated on change. Fullscreen on F,
blackout on B and period, `?` toggles a help overlay listing the keys, N toggles notes with
pacing (elapsed time against cumulative `pacing`). Click advances; a single-touch swipe past
the threshold moves; multi-touch never navigates. An `aria-live` region announces the slide.
`prefers-reduced-motion` removes transitions. Print hides counter, progress, help, notes and
blackout, and keeps the footer.

- [ ] Failing tests: each behaviour above has a structural marker in the HTML (handler,
  element id or CSS rule); the hash handling clamps (review focus); the threshold in the HTML
  equals the context value; print rules hide the interaction chrome and keep the footer
  (review focus).
- [ ] Run: fail. Implement. Run: pass.

## Task 8 - open only after a clean check

**Files:** `talk_validate.py`, `Test/test_talk_validate.py`.
**Produces:** `--open` with `--web`: after a clean check, `webbrowser.open` receives the deck
as a `file://` URI; with findings it returns 1 and opens nothing; without `--open` nothing
opens.

- [ ] Failing tests with `webbrowser.open` patched: clean deck plus `--open` opens once;
  findings plus `--open` exit 1 and do not open; clean deck without `--open` does not open.
- [ ] Run: fail. Implement. Run: pass. Then run every paper2talk suite.
