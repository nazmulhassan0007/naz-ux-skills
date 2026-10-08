# UX Complexity & Cognitive Load

Complexity issues are about effort: how much a user must read, decide, remember and do. Quantify where you can — counts make the issue credible.

## What to measure (from screenshot + metadata)
- **Decision points per view** — count interactive elements and distinct actions. Many equal-weight choices → Hick's law. Suggest grouping, progressive disclosure, smart defaults.
- **Primary actions** — more than one visually-primary button competing → recommend one primary, rest secondary/tertiary.
- **Steps to complete the core task** — count screens/clicks in the flow. Look for steps that could merge, be pre-filled, or be deferred.
- **Form length** — fields per screen; ask "is every field needed now?" Suggest deferring, grouping into sections, or autofill.
- **Information density** — number of cards/widgets, table columns, chart series. Dashboards: > ~7 widgets at the same level of emphasis usually needs a hierarchy or tabs.
- **Visual hierarchy** — squint test on the screenshot: what stands out first? Is it the most important thing? Count distinct type sizes/weights/colors (from inspection output).
- **Navigation depth & breadth** — nav items > 7, nested menus > 2 levels, unclear where you are.
- **Memory load** — info needed from a previous screen, unlabeled icons, codes users must remember.
- **Gestalt grouping** — related items close and aligned; unrelated items separated. Inconsistent spacing breaks perceived groups.
- **Fitts's law** — important/frequent targets large and near the user's focus; destructive actions not adjacent to frequent ones.
- **Mode & state complexity** — toggles/modes that change behavior silently; edit vs view modes unclear.

## Common recommendations
- Progressive disclosure: hide advanced options behind "More options"/accordion.
- Chunk long forms into 3–5 field groups or steps with a progress indicator.
- Smart defaults & pre-fill.
- Collapse low-frequency actions into an overflow menu.
- Establish a type scale (e.g. 5–6 sizes) and spacing scale (4/8-pt) to restore hierarchy.
- For dashboards: summary KPIs → trends → detail tables, top to bottom.

## Rating a screen
- Good: one obvious primary task, ≤ 5 main choices, clear hierarchy.
- Needs work: competing focal points or long forms, but task is findable.
- Poor: user can't tell what to do first, or core task needs avoidable extra steps.
