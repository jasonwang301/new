# Vector Figure Patch Pattern

Distilled from a ReportLab-based figure pipeline (`build_reference_style_v2.py` + a `Figure` object whose `items` are JSON). Adapt names to the local project, but keep the structure and the verification discipline.

## Load → patch → export (idempotent from a baseline)

```python
BAK = ROOT / "backups/before_<change>_<timestamp>/figures_final"
R.OUT = ROOT / "figures/final"; R.QA = QA

def load(name):
    return json.loads((BAK / (name + ".spec.json")).read_text(encoding="utf-8"))

def make(name, spec):
    f = R.Figure(name, spec["height"], spec["width"])
    f.items = spec["items"]
    f.counter = max(int(it["id"].rsplit("_", 1)[-1])
                    for it in f.items if it["id"].rsplit("_", 1)[-1].isdigit())
    return f

def finish(f, report):
    f.export()                       # regenerates PDF/PNG/SVG/DrawIO/spec
    print(json.dumps({**report, "name": f.name, "items": len(f.items),
                      "height": f.height, "audit": f.audit()}, ...))
```

Rules:

- Source of truth is the **baseline spec**, not last time's output. Rebuild the same patch each run so results are deterministic.
- Preserve item ids; to find one, `item(f, n) = next(it for it in f.items if num(it) == n)`.
- The builder's `Figure` API offers at least `rect`, `icon`, `text`, `arrow`, `heatmap`, `heat_legend`, `audit`, `export`. Look for the real method names in `scripts/figure_generation/`.

## Text edits must refresh bounds

Changing `it["text"]` without updating `it["bounds"]` breaks collision audits and hidden-arrow detection. Recompute measured width with the actual font:

```python
def set_text(it, value):
    it["text"] = value
    font = "TNR-Bold" if it.get("bold") else "TNR"
    w = max(pdfmetrics.stringWidth(s, font, it["size"]) for s in value.split("\n"))
    left = it["x"] if it.get("align") == "left" else it["x"] - w / 2
    it["bounds"] = [left, it["y"], left + w,
                    it["y"] + len(value.split("\n")) * it["size"] * 1.16]
```

For center-aligned text, remember the real bounds left edge is `x - half_text_width` — use it when avoiding images/arrows, not the `x` anchor.

## Operator icons with word-level labels

```python
def icon_with_label(f, x, y, kind, label, icon=30, size=25):
    f.icon(x, y, icon, icon, kind)
    f.text(x + icon + 10, y + 1, label, size, align="left")

def icon_row(f, centers, y_icon, y_label, kinds, labels, icon=28, size=25, arrow_y=None):
    for x, kind, label in zip(centers, kinds, labels):
        f.icon(x - icon/2, y_icon, icon, icon, kind)
        f.text(x, y_label, label, size)
    if arrow_y is not None:
        for x1, x2 in zip(centers[:-1], centers[1:]):
            f.arrow([[x1 + icon/2 + 3, arrow_y], [x2 - icon/2 - 3, arrow_y]], "#5B6B78", 1.8)
```

One pictorial operator + one short word per step. Replace a 5+ icon micro-chain with these word-level labels.

## GAP inset (graphic and text on the same row)

Two helpers: a small `gap_operator` token used inside processing chains, and a labelled `gap_detail` inset that explains the operation. The inset draws three graphic elements, each paired with a label:

```
[G×W colour grid] --arrow--> [averaging funnel icon] --arrow--> [channel-value vector]
   "H×W grid"                 "mean over H×W"                  "C values"
                plus a line "one value per channel"
```

Implementation notes:

- `compact=True` variant uses icon=30, cell=9, n=3, labels `H×W grid / mean / C values` for a corner token.
- Full variant uses icon=44, cell=14, and a title `GAP: grid → funnel → C vector`.
- Use a fixed palette of colour bands for the grid and vector; everything is schematic, no measured values.
- Draw **one** full inset in the detail figure. Other panels keep only the compact token plus (at most) a colour key.

## Colour key (instead of repeating a mechanism)

A `ccm_colour_key` draws a framed box with a title and a low→high heat legend; it explains how to read schematic weight tiles without redrawing the channel-recalibration chain. This is the standard replacement when a duplicated process strip must be removed.

## Arrow layering and straightness

- Hidden arrowhead = the arrow was appended **before** the white circle/rect it meets, so the later filled shape paints over it. Move the arrow's item to after that shape.
- Stop the endpoint just outside the target's edge (e.g. 2px above a circle, at the left edge of a Conv icon) so the fill never covers the tip.
- For an aligned connection prefer one straight horizontal segment at the target's centre height:
  `[[1183, 772], [1210, 772]]` rather than a multi-point elbow. Verify both endpoints share the same y when the line should be horizontal.
- Avoid arrows crossing operation glyphs; the audit flags "arrow through operation". If a GAP box carries `obstacle=True`, internal arrows get flagged — set it to False when the arrow must pass through.

## Heat-map tiles and labels

- Use a continuous TURBO band (`heat_color(t)`) or an importance band split into cool/mid/warm t-ranges to create input-vs-output contrast. All generated in vector form; keep PDFs bitmap-free if the project requires all-vector.
- When the advisor wants heat maps "from the dataset image", generate tiles from the project's **own** real SAR crop (e.g. `tmp/sarheat/make_tiles.py` from the figure's existing asset), producing `sar_heat_input.png` (cool/diffuse) and `sar_heat_output.png` (warm/focused). Never crop a third-party paper figure.
- Put labels **below** tiles, never on top: a white-background label drawn over a warm tile hides the whole point.
- Caption must say schematic/illustrative, not measured.

## Verification checklist

1. `f.audit()` empty — no out-of-bounds, no arrow-through-operation, no overlap.
2. Report JSON records `items`, `height`, `audit` and is saved under `qa/`.
3. Figure repo check: PDF/PNG/SVG/DrawIO/spec all regenerated; SHA consistent across delivered copies.
4. Inserted-page render at 150 dpi inspected at journal scale; no clipping/overlap.
5. Embedded font floor: if inserted at `width = w·\textwidth`, an N-pt spec font embeds as roughly `N·w` pt. Keep ≥ 8pt. A 0.8 scaling of a 25pt font is ~6.4pt → FAIL; use full `\textwidth` or raise the font.
6. Paper-level gates: 0 error / 0 LaTeX Warning / 0 Overfull / 0 undefined; citation keys == bibitems; version record and translated twins updated.
