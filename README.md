# research-figure-rework

A WorkBuddy skill for reworking and auditing scientific figures so that a reader can understand the mechanism from the figure alone.

Distilled from a real journal-paper rework: one overall architecture figure plus three mechanism figures, driven by an advisor's recorded meeting comments and many rounds of user correction. It turns "the figure is too simple / I can't understand the arrows / where did this weight come from" into a repeatable, checkable workflow.

## Install

Copy this folder into `~/.workbuddy/skills/research-figure-rework/` (user scope) or `<project>/.workbuddy/skills/research-figure-rework/` (project scope). WorkBuddy picks it up automatically.

## Contents

- `SKILL.md` — when to use, six hard rules, anti-red-lines, the 10-step workflow, scope boundary.
- `references/acceptance_rules.md` — the point-and-read (指读) acceptance test, the six-rule table with the advisor's verbatim complaints, output format, and patch recipes.
- `references/vector_build.md` — the vector-figure patch pattern: baseline spec → item-level patch → five-artifact regeneration → audit → font-floor and paper-level verification.

## The core bar

For every sentence in the method text, a reader's finger must land on a concrete figure element, and a naive reader must be able to retell the module's data flow from the figure and legend alone. If not, the figure fails.

## License

MIT
