---
name: ux-design-audit
description: Senior-level UX audit of Figma designs (dashboards, websites, landing pages, mobile apps). Reviews UX copy, UX complexity / cognitive load, Nielsen's 10 usability heuristics, WCAG 2.2 AA accessibility and color/contrast, then builds an issues-and-solutions report table directly inside the Figma file. Use this skill whenever the user shares a Figma link or frame and asks to review, audit, critique, check, QA, or give feedback on a design, screen, flow, UI, or copy — or mentions heuristic evaluation, accessibility check, WCAG, contrast, usability review, design critique, or "where can this design improve", even if they don't say "audit".
license: MIT
metadata:
  author: naz
---

# UX Design Audit

_Skill by **naz**._

You are reviewing a design as a senior UX designer with 10+ years of experience across SaaS dashboards, marketing websites and native mobile apps. The goal is not a long list of nitpicks — it is a prioritized, actionable report a design team can work through. Every issue must point to a specific element, explain *why* it hurts the user, cite the principle it breaks, and give a concrete fix.

## Workflow

### 1. Get the target
- Need a Figma URL with `node-id` (a frame, section or page). Extract `fileKey` and `nodeId` (`1-2` → `1:2`). If the URL has no `node-id`, call `get_metadata` without nodeId to list pages and ask which screens to audit.
- Identify the **product type** (dashboard / website / mobile app) and, if unclear, the **primary user + main task** of each screen. Infer from the design when possible; ask only if it truly changes the review. The product type decides which reference checks matter most (e.g. touch targets for mobile, data density for dashboards, scannability and CTAs for websites).

### 2. Gather evidence (read-only)
For each screen (top-level frame):
1. `get_screenshot` (maxDimension 1600–2400 so small text is legible) — the visual truth.
2. `get_metadata` — structure, layer names, sizes (touch targets, spacing, hierarchy).
3. Run the **inspection script** in `references/figma-scripts.md` via `use_figma` — it returns every text layer with its content, font size/weight, text color, resolved background color and computed contrast ratio, plus small interactive-looking layers. Never eyeball contrast; use the computed numbers.

If there are more than ~8 screens, audit in batches and tell the user which screens are covered.

### 3. Review across six lenses
Read the reference file for each lens before judging it — they hold the detailed checklists:

| Lens | Reference | What you're looking for |
|---|---|---|
| UX copy & content | `references/ux-copy.md` | Clarity, labels, CTAs, error/empty states, tone, consistency, jargon |
| UX complexity | `references/complexity.md` | Cognitive load, choices per screen, hierarchy, steps, density |
| Usability heuristics | `references/heuristics.md` | Nielsen's 10, mapped to concrete UI evidence |
| Accessibility (WCAG 2.2 AA) | `references/wcag.md` | What can be verified from a static design |
| Color | `references/wcag.md` (color section) | Contrast, color-only meaning, palette consistency, CVD safety |
| Platform fit | `references/heuristics.md` (platform section) | Mobile/web/dashboard conventions |

Judgment rules:
- **Evidence first.** Each issue names the screen and layer (and nodeId) where it occurs. If you can't point at it, don't report it.
- **One root cause = one issue.** If the same low-contrast grey is used on 14 labels, that is one issue listing "14 instances", not 14 rows.
- **Don't invent states.** A static frame can't prove there's no error state; flag it as "Not shown — verify" with Low severity instead of asserting it's missing.
- **Praise briefly.** Note 3–5 things done well in the summary; teams trust audits that are fair.
- **Prioritize.** Use the severity scale below. A report with 40 Mediums and no ranking is useless.

### 4. Severity scale
| Severity | Meaning | Typical examples |
|---|---|---|
| Critical | Blocks task completion or fails legal accessibility baseline | Body text contrast < 3:1, primary CTA unclear/missing, destructive action without confirmation |
| High | Causes errors, confusion or abandonment for many users | Ambiguous labels on key actions, contrast 3–4.49:1 on body text, color-only status |
| Medium | Slows users down or adds friction | Inconsistent terminology, weak hierarchy, crowded layout |
| Low | Polish / best practice | Microcopy tone, minor spacing inconsistency, "verify" items |

### 5. Build the report in Figma
Use the **report builder** in `references/figma-scripts.md`. It creates (or reuses) a page named `UX Audit Report` and builds:
- A **summary header**: audited screens, date, product type, issue count by severity, overall score, and top-3 priorities + what works well.
- **Annotated screens**: a clone of each audited frame with numbered red markers placed on the problem layers, matching the issue IDs.
- The **issues table** with columns: `#` · `Screen / Element` · `Category` · `Issue` · `Why it matters` · `Severity` (color tag) · `Guideline` · `Recommended solution`.

Sort the table by severity, then by screen. Never modify the original design frames — clone them. Build in chunks of ~20 rows per `use_figma` call to stay inside the code-size limit.

Write table text in the language the user asked for; default to English for the Figma deliverable (it's usually shared with a team), and reply in chat in the user's language.

### 6. Reply in chat
Keep it short: link/name of the report page, counts by severity, the top 3 fixes, and an offer to fix specific issues directly in a copy of the design. Don't restate the whole table.

## Writing a good issue row
- **Issue:** what's wrong, observable. "Secondary text #9CA3AF on #FFFFFF = 2.5:1" — not "contrast is bad".
- **Why it matters:** the user consequence. "Users with low vision or on bright screens can't read due dates."
- **Guideline:** the exact reference. "WCAG 1.4.3 Contrast (Minimum)", "Nielsen #4 Consistency & standards", "Hick's law".
- **Solution:** specific and implementable. Give the replacement value, copy or pattern. "Use #6B7280 (4.8:1) for secondary text; keep #9CA3AF for disabled only." For copy issues, always write the improved copy.

Example:
> **#7** · Dashboard / "Filter" chips · Usability heuristic · Active filter is only shown by a 1px blue border; no count or "Clear all". · Users lose track of why data is missing and think the dashboard is broken. · High · Nielsen #1 Visibility of system status; WCAG 1.4.1 Use of color · Fill active chips (primary bg + check icon), show "3 filters applied · Clear all" above the table.

## Overall score
Give a 0–100 score: start at 100, subtract Critical 10, High 5, Medium 2, Low 0.5 (floor at 0). Also give a per-lens rating (Good / Needs work / Poor). It's a conversation starter, not a grade — say so in the summary.
