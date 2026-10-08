# WCAG 2.2 AA — checks verifiable from a static design

Target AA by default. Mention AAA only as a bonus suggestion. Cite criteria by number and name.

## Contrast (use computed values from the inspection script)
| Criterion | Requirement |
|---|---|
| 1.4.3 Contrast (Minimum) | Normal text ≥ 4.5:1. Large text (≥ 24px regular, or ≥ 18.66px / 14pt bold) ≥ 3:1 |
| 1.4.11 Non-text Contrast | UI component boundaries (input borders, toggles, checkbox outlines), focus indicators, and meaningful chart/icon graphics ≥ 3:1 against adjacent colors |
| 1.4.6 (AAA, optional) | Normal text ≥ 7:1 |

Exempt: disabled controls, pure decoration, logos. Still note disabled text if it's the *only* way info is conveyed.
Text on images/gradients: script can't resolve it — inspect the screenshot, flag as "verify: text over image" and suggest an overlay/scrim.

Severity guide: body text < 3:1 → Critical; 3–4.49 → High; large text/icon/border just under 3:1 → Medium.

## Color (beyond contrast)
- **1.4.1 Use of Color** — status (error/success/warning), chart series, links in body text, required fields, selected states must not rely on color alone. Add icon, text, pattern, underline or weight.
- **Color-vision deficiency** — red/green pairs (P&L, status pills, charts) fail for ~8% of men. Suggest blue/orange, or add shape/labels.
- **Palette consistency** — count distinct greys/brand variants from the inspection output; near-duplicates (e.g. #6B7280 and #6C7381) signal missing tokens. Recommend a token set.
- **Semantic consistency** — the same color must mean the same thing everywhere (don't use brand red for both CTA and errors).
- **Dark mode** (if present) — avoid pure #000 bg with pure #FFF text glare; check contrast in both themes.

## Other visual-verifiable criteria
| Criterion | What to check in Figma |
|---|---|
| 1.1.1 Non-text Content | Icon-only buttons need an accessible name — recommend a label or annotate aria-label |
| 1.3.1 Info and Relationships | Form labels visibly tied to fields; headings look like headings; tables have headers |
| 1.3.4 Orientation | Mobile designs not locked to one orientation without reason |
| 1.4.4 / 1.4.12 Resize & Text Spacing | Fixed-height containers that would clip text when enlarged; very tight line-height (< 1.2) |
| 1.4.10 Reflow | Web: a 320px-wide version should work without horizontal scroll |
| 2.4.4 Link Purpose | "Click here", "Read more" without context |
| 2.4.6 Headings and Labels | Descriptive headings and field labels |
| 2.4.7 / 2.4.11 Focus Visible / Not Obscured | Focus state designed for interactive components? Sticky headers/footers covering focused items? |
| 2.5.3 Label in Name | Visible label text matches what an accessible name would be |
| 2.5.7 Dragging Movements | Drag-only interactions (kanban, sliders) need a click alternative |
| 2.5.8 Target Size (Minimum) | Targets ≥ 24×24 CSS px (or enough spacing). Recommend 44/48 for mobile |
| 3.2.4 Consistent Identification | Same function → same icon/label across screens |
| 3.3.1 / 3.3.3 Error Identification & Suggestion | Error text explains and suggests fix |
| 3.3.2 Labels or Instructions | Placeholder-only fields fail; formats shown (e.g. "DD/MM/YYYY") |
| 3.3.7 Redundant Entry | Multi-step forms don't ask for the same info twice |
| 3.3.8 Accessible Authentication | Login doesn't require cognitive tests without alternative; allow paste/password managers |

## Typography readability (best practice, not strict WCAG)
- Body ≥ 16px web / 16–17 mobile; nothing below 12px except legal fine print.
- Line height 1.4–1.6 for body; line length 45–80 characters.
- Avoid all-caps for long text; avoid light weights (300 or below) at small sizes.
