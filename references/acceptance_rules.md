# Point-and-Read Acceptance Rules (指读验收标准)

Source: an advisor's recorded meeting (transcript 4:13–10:56 plus 12:19/17:32/25:21) and the user's follow-up instruction. The verbatim complaints are included because they define the FAIL conditions more precisely than any paraphrase.

## The core action

Take one figure together with the paper paragraph and caption that describe it, then run a **point-and-read test**:

1. Read each sentence of the method text in order.
2. For every sentence, the finger must land on one concrete figure element (box / arrow / icon / label / legend). Cannot point → FAIL.
3. For every arrow, answer three questions: **who flows to whom? what is flowing? why does it flow that way?** Cannot answer → FAIL.
4. A person who has never seen the paper retells, from the figure and legend alone, "what this module does and how data moves". Cannot retell or retells wrongly → FAIL.

One-line criterion: **the figure must be able to "say" the text's workflow by itself, without the author standing beside it adding words.**

## Six hard rules

| # | Rule | Typical FAIL | Advisor's verbatim complaint |
|---|------|--------------|------------------------------|
| R1 | Every logic step has a landing spot on the figure | Text says "weighted fusion" but the figure only has one line dropping three branches together, with no "weighting"/"concat" action box | "他3个做啥操作？一次性扔下去了" |
| R2 | Every trainable quantity shows how it was obtained | w1/w2/w3 or ω appear out of nowhere, with no GAP/MLP/softmax generation path from the inputs | "这三个参数的获取过程是哪儿啊？完全没有" |
| R3 | Every arrow answers "what flows?" | Blue and black lines mixed with no legend; a bare line with no label "turns this into that" | "这条直线它是啥操作" / "箭头线属于什么操作都不知道" |
| R4 | Symbols match text match caption, character for character | Text w1w2w3, caption ω1ω2ω3, figure w₁L — three spellings | "这不是 L 吗？这不是 Q 吗" (character-by-character check) |
| R5 | Abbreviation's first use gives full name; in-figure label matches later text | CMLCA/CCM/OAF written with no expansion | "缩写人家不知道是啥呀" |
| R6 | A naive reader can retell the main line | Just a pile of boxes and lines; "features go back and forth with no logic" | "特征来回走，走来走去，就没有一点儿的逻辑关系" |

## Additional advisor anchors (meeting transcript)

- 9:08 "the figure is a bit too simple" → use heat maps for weights and gates, not plain colour blocks.
- 9:14 "input is this colour and output is this colour too" → input/output must contrast (cool/diffuse input, warm/focused output); stop drawing everything as 3D blocks.
- 9:56 "the formula has nothing to do with the figure" → draw the logical relationship in the figure; formulas and figure must correspond.
- 10:36 "draw the logic relationship up here and also down here; these two formulas appear nowhere in your figure" → every formula in the text needs a visual counterpart.
- 10:52 "go fix these figures, they are really bad right now".
- 12:00 "every abbreviation in the figure must have its formula/meaning somewhere, otherwise you are writing nonsense".
- 8:22 "0/1 should be a valve but it isn't drawn well, nobody can tell" → gate must be a valve with visible branches.

## Output format: the point-and-read table

For each figure produce:

| Text/caption sentence | Figure element (box name / arrow endpoints / coordinates) | Three questions (who→who / what / why) | Rules hit (R1–R6) | Verdict |
|-----------------------|-----------------------------------------------------------|----------------------------------------|-------------------|---------|

- Any FAIL → figure "not passed"; patch by the hit rule number and re-verify.
- All PASS and naive retelling succeeds → "passed".

## Boundary while patching (avoid overreach)

- Minimal diff: add only the missing landing spots and arrow semantics. Do not change module structure, formula values, or experiment data.
- No self-exposure: legends/labels describe only this figure's own processing; do not import experiments or comparisons the paper did not do.
- Font: figure text ≤ body text (this project locked an 8pt embedded floor); new labels follow the existing glyph/legend size, never larger.
- Five-artifact sync: PDF / PNG / SVG / DrawIO / spec.json regenerated together; spec keeps its `items`/`bounds`/`group_kind` structure.

## Common patch recipes that satisfy the rules

- Weight origin (R2): draw warm dashed lines from the *pre-weighting* L/Q/S branch outputs into a `GAP → MLP → Softmax` chain; label the resulting w1/w2/w3 warm to match. Do not feed the chain from the already-weighted bus (that creates a semantic loop).
- Arrow payload (R3): put the tensor name on the arrow (e.g. `F_k`, `R_k`, `weights`), and add a two-line legend "blue = feature flow, orange dashed = skip".
- Valve (gate): inside the gate box draw `keep Ri (g=0)` and `+0.1 Di (g=1)` on two branches that merge; keep the hard label `g = 0/1`. Ensure the two lines fit inside the box with ≥ line-height spacing and do not collide with the exit arrow.
- Heat tiles: use a low–high colour key; place the key where it does not overlap another panel's legend or a frame border.
- Duplicates: when removing a repeated inset/chain, replace it with information not shown elsewhere (colour key, formula, or a distinct operation), so the freed space is not wasted and the same mechanism is not drawn twice.
