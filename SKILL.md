---
name: research-figure-rework
description: Rework and audit scientific figures in academic papers (especially deep-learning / SAR / CV papers) so that a reader can understand the mechanism from the figure alone. Use when a user asks to redraw, fix, simplify, or review a method/framework/architecture/mechanism figure, when an advisor says a figure is "too simple", "cannot understand it", "arrows have no meaning", "where did this weight come from", "input and output look the same", "draw heat maps", or when running a figure-vs-text ("指读") consistency check. Covers the full loop: read teacher/user feedback, apply six hard rules, patch vector specs, regenerate five artifacts, audit, visual check, recompile, and dual-gate verification.
agent_created: true
---

# Research Figure Rework

## Purpose

Make a paper figure explain its own mechanism. The acceptance bar is a **指读测试 (point-and-read test)**: read each sentence of the paper's method text aloud, and for every sentence the reader's finger must land on a concrete figure element (box / arrow / icon / label / legend). If any step cannot be pointed to, or a naive reader cannot retell "what goes in, how many branches, what is fused, what comes out" from the figure alone, the figure FAILS.

This skill was distilled from a real multi-day rework of journal figures: one overall architecture figure plus three mechanism figures (multi-angle rotation refinement, cascaded channel/spatial attention, cross-scale fusion with a stochastic gate), driven by an advisor's recorded meeting comments and repeated user corrections. The rules below are those that actually moved a figure from FAIL to PASS.

## When to use

- "Redraw / fix / simplify this mechanism figure" ; "make the figure clearer".
- Advisor says: too simple / I can't understand it / this arrow has no operation / where did ω come from / draw it as a valve / use heat maps / input and output must differ in colour / don't repeat / no big blocks of text.
- Need a figure-vs-text consistency review ("图文对照验收").
- Need to regenerate a vector figure's PDF/PNG/SVG/DrawIO/spec together and verify nothing overflowed.

## Core principles (six hard rules, each PASS/FAIL)

Read `references/acceptance_rules.md` for the full table with the advisor's verbatim complaints and the point-and-read procedure. Summary:

1. **R1 — Every logic step has a landing spot.** Each "input → branch → fuse → weight → output" step in the text has a corresponding box/arrow in the figure. Never collapse three operations into one bare line.
2. **R2 — Every trainable quantity shows its origin.** ω, gates, channel weights w1/w2/w3 etc. must have an explicit generation path (e.g. `GAP → MLP → Softmax`) drawn from the inputs, not appear from nowhere.
3. **R3 — Every arrow answers "what flows?".** Distinguish feature / weight / coordinate / residual by colour and line style, with a legend. Blue solid = feature flow, orange dashed = skip/weight, warm colour = gated update is a proven convention.
4. **R4 — Symbols match text match caption, character for character.** One symbol, one spelling, including subscripts (w1 vs ω1 vs w₁L is a FAIL).
5. **R5 — First use of an abbreviation gives the full name**, and the in-figure label matches later text.
6. **R6 — A naive reader can retell the main line** from figure + legend alone.

## Anti-red-lines (learned the hard way)

- **Never dump a prose block into the figure.** The advisor's exact reaction: "don't write a big paragraph — show it with boxes or patterns in the figure." Use enlarged graphic symbols + word-level labels, not explanatory sentences.
- **Never draw a long string of tiny operator icons** ("一长串...谁也看不懂"). One pictorial operator + one short word per step. If a chain has 5+ micro-icons, replace with word-level short labels.
- **Never duplicate a mechanism.** If an inset explains GAP in panel (b), do not repeat the same GAP inset in panel (c); keep a compact operator token there instead. Same for a repeated CCM chain — draw once, then give only a colour key. When deleting a duplicate, fill the freed space with genuinely new information (e.g. a colour legend, a bottleneck formula).
- **GAP in a detail figure must be drawn, with graphic and text on the same row**: `H×W grid → mean over H×W → C values`, plus "one value per channel". In an overview/framework figure a single GAP token is enough.
- **Heat-map colour is semantics, not decoration.** Input cool/diffuse → output warm/focused. Heat maps must be sourced from a real dataset image when the advisor asks for that; use the project's own generated asset, never crop a third-party paper. Label text must never cover the colour block — put labels below. Mark such maps "schematic / illustrative, not measured".
- **Gate = valve.** A 0/1 gate must be drawn as a valve with two branches and a merge, labelled "keep Ri (g=0)" / "+0.1 Di (g=1)", not just a `g = 0/1` text box.
- **Show weight/gate heat maps as schematic small tiles with a low–high colour key**, not 3D magic cubes.
- **When the reader keeps saying "I don't get it" and edits get more complex, fall back to the exact reference figure the user first approved** and copy its visual vocabulary (large 3D blocks / real SAR tiles / clear arrows) instead of inventing new glyphs.
- **Arrow visibility is a layering problem.** If an arrowhead is hidden, check draw order: the arrow must be appended *after* the white circle/box it meets. Endpoints should stop outside the target shape's edge. Prefer a single straight line aligned with the target's centre over an elbow.
- **Minimum-font floor.** If a spec font is 25pt but LaTeX inserts it at `width=0.8\textwidth`, the embedded size is ~6.4pt — below an 8pt floor. Either draw at full `\textwidth` or raise the spec font; compute the real embedded size, not the nominal one.

## Workflow (follow in order)

1. **Read the source of truth.** Load the teacher/user feedback first (meeting transcript, comments, screenshots) and the figure spec JSON. Never invent requirements. Map each comment to a specific panel.
2. **Establish the baseline backup.** Copy current figure artifacts to `backups/before_<change>_<timestamp>/`. All subsequent builds must be idempotent from that baseline.
3. **Patch, do not redesign from scratch.** Load the original spec, apply targeted item-level edits (see `references/vector_build.md`), preserve item ids, coordinates not touched, and all numeric/formula semantics.
4. **Regenerate the five artifacts together** — PDF / PNG / SVG / DrawIO / spec.json — keeping the spec's `items`/`bounds`/`group_kind` structure intact.
5. **Run the figure audit** (`f.audit()` or equivalent). It must be empty: no arrow crossing an operation, no out-of-bounds text, no overlapping label on a heat tile.
6. **Visual check at journal scale.** Render the inserted page at ~150 dpi and inspect at actual printed size; a figure that passes audit can still be unreadable when shrunk.
7. **Point-and-read test** on the patched figure (see acceptance_rules.md). Any FAIL → patch and re-verify.
8. **Independent re-review.** Spawn separate reviewers (text/claims, experiment/reproducibility, figure/visualization). Triage their FAILs one by one; re-review until PASS.
9. **Recompile the paper** and run both verification gates. Check page count, 0 error / 0 LaTeX Warning / 0 Overfull / 0 undefined, citation/bibitem balance, identical SHA across all delivered PDF copies, and minimum embedded figure font ≥ 8pt.
10. **Log the change** with a version-record entry and update any tracked-doc manifests.

## Scope boundary (do not cross)

- Change only the figure's visual explanation: add missing landing spots and arrow semantics. **Never change module structure, formula values, experiment data, numbers, or measured activations.**
- Keep schematic overlays explicitly labelled illustrative, not measured.
- Keep figure text ≤ body text size; do not open a larger font for new labels.
- Sync every textual twin: English caption, Chinese caption, translated manuscript, generated HTML, and version record.

## Bundled resources

- `references/acceptance_rules.md` — the full six-rule point-and-read acceptance table with verbatim advisor complaints and the FAIL/PASS output format.
- `references/vector_build.md` — the vector-figure patch pattern: load/patch/export flow, the `set_text` bounds refresh, operator-icon and GAP-inset helpers, layering rule for arrows, and the five-artifact + audit + font-floor verification.
