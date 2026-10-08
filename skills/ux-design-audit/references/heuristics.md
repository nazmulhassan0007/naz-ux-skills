# Nielsen's 10 Usability Heuristics — what to look for in a static design

For each heuristic: the evidence you can actually see in Figma, and common fixes.

## 1. Visibility of system status
Look for: active/selected states on nav, tabs, filters; loading, saving, syncing indicators; progress in multi-step flows; "last updated" time on dashboards; badge counts; upload progress.
Red flags: tabs with no clear selected state; steppers without "Step 2 of 4"; dashboards with no data freshness indicator.
Fixes: distinct selected state (fill + weight, not color alone), progress indicator, timestamps, toasts/inline confirmation.

## 2. Match between system and the real world
Look for: internal jargon, database field names ("user_id", "Txn"), developer terms, unfamiliar abbreviations, icons with non-standard meaning, date/currency formats wrong for the market.
Fixes: user vocabulary, full words, locale formats (e.g. ৳ / BDT, DD MMM YYYY), recognizable metaphors.

## 3. User control and freedom
Look for: back/cancel/close on every modal and step, undo after destructive actions, clear exits from flows, editable inputs after submission.
Red flags: modal without close; "Delete" with no confirmation or undo; wizards that can't go back.

## 4. Consistency and standards
Look for: same action named differently ("Save"/"Submit"/"Confirm"), mixed button styles for the same priority, inconsistent icon sets, inconsistent spacing/radius, components that look clickable but aren't (or vice versa), platform conventions broken.
Tip: use the inspection output — count distinct font sizes, text colors, radii. More than ~6 text sizes or ~8 grey values usually signals a missing system.

## 5. Error prevention
Look for: constraints on inputs (date pickers vs free text), inline validation hints, disabled submit until valid (with reason), confirmations on irreversible actions, sensible defaults.

## 6. Recognition rather than recall
Look for: visible labels (not placeholder-only fields), icons without labels, users forced to remember info from a previous screen, missing recent/search history, hidden navigation behind unlabeled icons.

## 7. Flexibility and efficiency of use
Look for: bulk actions on tables, search, filters, keyboard shortcuts hints, saved views, quick actions. Matters most for dashboards and power-user tools.

## 8. Aesthetic and minimalist design
Look for: competing focal points, more than one primary CTA per view, decorative elements stealing attention, redundant text, dense tables without grouping, too many card styles.

## 9. Help users recognize, diagnose, and recover from errors
Look for: error states designed? Messages say what happened + how to fix, in plain language, placed next to the field, not color-only.
If no error states are present in the frames: report "Not shown — verify" (Low).

## 10. Help and documentation
Look for: contextual help (tooltips, info icons) on complex fields/metrics, empty states that teach, onboarding hints, links to docs on advanced features.

---

## Platform fit

### Mobile apps (iOS / Android)
- Touch targets ≥ 44×44 pt (iOS) / 48×48 dp (Android); spacing ≥ 8 between targets.
- Primary actions in the thumb zone (bottom ⅓); avoid key actions in top corners only.
- Bottom nav: 3–5 items, labeled. Respect safe areas (status bar, home indicator).
- Body text ≥ 16 (iOS 17pt default), no text < 11.
- Native patterns: back affordance, sheets, pull-to-refresh where expected.

### Websites / landing pages
- One clear primary CTA above the fold; value proposition readable in 5 seconds.
- Scannability: headings, short paragraphs (≤ 80 chars line length), F/Z patterns.
- Nav ≤ 7 top-level items; clear current-page indicator; footer essentials.
- Trust signals near conversion points; forms short.
- Check responsive frames exist (desktop + mobile) — if only one, note "verify responsive behavior".

### Dashboards / data apps
- The 3–5 most important KPIs first (top-left), with comparison context (vs last period, target).
- Charts: labeled axes, units, legends not color-only, ≤ 5–7 series, sensible chart type.
- Tables: right-align numbers, sticky header, sorting/filter affordances, row actions discoverable, pagination or count.
- Empty, loading, and error states for widgets.
- Information density appropriate to the user (ops monitoring vs executive).
