# UX Copy & Content Review

Pull all strings from the inspection script output and read them as a set — inconsistencies only show up when you see them together. For every copy issue, write the improved copy in the Solution column.

## Checklist
**Clarity**
- Plain language, ~grade 7–8 reading level for consumer products. Replace jargon, internal names, acronyms (or expand on first use).
- Front-load the meaning: "Delete project?" not "Are you sure you want to proceed with this action?"
- Numbers and units explicit: "Due in 3 days", "৳12,400 / month", not "12.4k".

**Buttons & CTAs**
- Verb + object: "Create invoice", "Download report" — not "OK", "Submit", "Click here".
- Button label matches the result and the title of the next screen/modal.
- Destructive actions name the consequence: "Delete 3 files".
- One primary CTA per view; secondary CTAs clearly lower priority.

**Labels & forms**
- Visible labels above fields (placeholder is a hint, not a label).
- Helper text for format/constraints before the user errs ("At least 8 characters").
- Mark required or optional consistently (prefer marking the minority).

**Errors, empty & success states**
- Error = what happened + why (if useful) + how to fix. No blame, no codes alone. "Card declined. Check the number or try another card."
- Empty state = what this area is + why it's empty + a next action.
- Success confirms the specific outcome: "Invoice #1042 sent to rahim@…".

**Consistency**
- One term per concept across all screens (Student/Learner/User; Remove/Delete).
- Consistent capitalization (pick Sentence case for UI; Title Case only if it's the system).
- Consistent date, time, currency formats.

**Tone & voice**
- Matches brand and context; serious in errors/payments, warmer in onboarding.
- No unnecessary exclamation marks, no humor in error states.

**Scannability**
- Headings describe content; paragraphs ≤ 3 lines on mobile; bullets for lists of 3+.
- Truncation: long names/values will they ellipsize sensibly? Flag fixed-width text likely to break with real data or translation (Bangla/German strings are often 30–40% longer).

**Placeholder content**
- Flag lorem ipsum, "Lorem", "Text", "Button", "Title" layers left in final designs — and unrealistic sample data that hides layout problems (all names 5 letters, all numbers 2 digits).

## Copy issue row example
Issue: Modal CTA says "Yes" / "No" under "Do you want to discard changes?"
Solution: Title "Discard unsaved changes?" · Buttons "Discard" (destructive) / "Keep editing".
