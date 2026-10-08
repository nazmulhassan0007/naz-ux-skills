# Figma scripts (run with `use_figma`)

Load the figma-use guidance first if the Figma MCP requires it. Never call `loadAllPagesAsync`, `setPluginData` or `createImageAsync` (unsupported). Change pages only with `await figma.setCurrentPageAsync(page)`.

---

## A. Inspection script (read-only)

Replace `TARGET_IDS`. Returns JSON with every text layer's computed contrast, small targets, and style counts. Background is resolved by compositing every solid fill painted below the text (depth-first order = paint order) whose bounds contain the text's center. If the top-most underlying paint is an image or gradient, `bg` is `"complex"` — check that one visually.

```js
const TARGET_IDS = ["1:2"]; // frames to inspect

const hex = c => '#' + [c.r, c.g, c.b].map(v => Math.round(v * 255).toString(16).padStart(2, '0')).join('').toUpperCase();
const lum = c => { const f = v => v <= 0.03928 ? v / 12.92 : Math.pow((v + 0.055) / 1.055, 2.4); return 0.2126 * f(c.r) + 0.7152 * f(c.g) + 0.0722 * f(c.b); };
const ratio = (a, b) => { const L1 = lum(a), L2 = lum(b); return (Math.max(L1, L2) + 0.05) / (Math.min(L1, L2) + 0.05); };
const over = (top, a, bot) => ({ r: top.r * a + bot.r * (1 - a), g: top.g * a + bot.g * (1 - a), b: top.b * a + bot.b * (1 - a) });
const visibleFills = n => ('fills' in n && Array.isArray(n.fills)) ? n.fills.filter(f => f.visible !== false && (f.opacity ?? 1) > 0) : [];
const center = n => { const b = n.absoluteBoundingBox; return b ? { x: b.x + b.width / 2, y: b.y + b.height / 2 } : null; };
const contains = (n, p) => { const b = n.absoluteBoundingBox; return b && p && p.x >= b.x && p.x <= b.x + b.width && p.y >= b.y && p.y <= b.y + b.height; };
const isVisible = n => { let x = n; while (x && x.type !== 'PAGE') { if (x.visible === false) return false; x = x.parent; } return true; };

const out = [];
for (const id of TARGET_IDS) {
  const frame = await figma.getNodeByIdAsync(id);
  if (!frame) { out.push({ id, error: 'not found' }); continue; }
  const ordered = [frame];
  if ('findAll' in frame) ordered.push(...frame.findAll(() => true));
  const texts = [], small = [], sizes = {}, colors = {};
  ordered.forEach((n, idx) => {
    if (!isVisible(n)) return;
    if (n.type === 'TEXT') {
      const fill = visibleFills(n).find(f => f.type === 'SOLID');
      const p = center(n);
      let bg = { r: 1, g: 1, b: 1 }, complex = false;
      for (let i = 0; i < idx; i++) {
        const m = ordered[i];
        if (m.type === 'TEXT' || !isVisible(m) || !contains(m, p)) continue;
        for (const f of visibleFills(m)) {
          if (f.type === 'SOLID') { bg = over(f.color, (f.opacity ?? 1) * (m.opacity ?? 1), bg); complex = false; }
          else complex = true;
        }
      }
      const size = typeof n.fontSize === 'number' ? n.fontSize : 'mixed';
      const weight = typeof n.fontWeight === 'number' ? n.fontWeight : (n.fontName && n.fontName.style) || 'mixed';
      let fg = null, r = null;
      if (fill) { fg = over(fill.color, (fill.opacity ?? 1) * (n.opacity ?? 1), bg); r = Math.round(ratio(fg, bg) * 100) / 100; }
      const large = typeof size === 'number' && (size >= 24 || (size >= 18.66 && typeof weight === 'number' && weight >= 700));
      sizes[size] = (sizes[size] || 0) + 1;
      if (fg) colors[hex(fg)] = (colors[hex(fg)] || 0) + 1;
      texts.push({
        id: n.id, name: n.name, text: n.characters.slice(0, 90), size, weight,
        fg: fg ? hex(fg) : 'non-solid', bg: complex ? 'complex' : hex(bg), ratio: complex ? null : r,
        large, passAA: complex || r === null ? null : r >= (large ? 3 : 4.5),
        lineHeight: n.lineHeight && n.lineHeight.unit !== 'AUTO' ? n.lineHeight : 'auto'
      });
    } else if (/(button|btn|icon|link|tab|chip|toggle|checkbox|radio|close|menu)/i.test(n.name) || n.type === 'INSTANCE') {
      const b = n.absoluteBoundingBox;
      if (b && (b.width < 44 || b.height < 44) && b.width > 0) small.push({ id: n.id, name: n.name, w: Math.round(b.width), h: Math.round(b.height) });
    }
  });
  out.push({
    frame: frame.name, id, w: Math.round(frame.width), h: Math.round(frame.height),
    textCount: texts.length,
    failingContrast: texts.filter(t => t.passAA === false),
    needsVisualCheck: texts.filter(t => t.passAA === null),
    fontSizes: sizes, textColors: colors,
    smallTargets: small.slice(0, 60),
    allText: texts.map(t => ({ id: t.id, text: t.text, size: t.size, ratio: t.ratio }))
  });
}
return JSON.stringify(out);
```

If the JSON is too large, drop `allText` for big frames and re-run on sub-sections for copy review.

---

## B. Report builder

Run in steps. Keep each call under ~40k characters.

### B1. Page, summary header and annotated screens

Fill `SUMMARY` and `ISSUES` (all issues — markers need them). `nodeId` is the layer the issue points at; `screenId` is its top-level frame.

```js
const SUMMARY = {
  title: "UX Audit Report — Project name",
  date: "2026-10-08",
  auditor: "naz",
  productType: "Dashboard",
  screens: ["Overview", "Students list"],
  score: 68,
  lensRatings: { "UX copy": "Needs work", "Complexity": "Good", "Heuristics": "Needs work", "WCAG": "Poor", "Color": "Needs work" },
  priorities: ["Fix secondary text contrast (12 instances)", "Add active filter state", "Rename ambiguous CTAs"],
  strengths: ["Clear KPI hierarchy", "Consistent card system", "Good empty state on Reports"]
};
const ISSUES = [
  { id: 1, screenId: "1:2", nodeId: "1:45", severity: "Critical" }
  // ... only id/screenId/nodeId/severity are needed here
];

const SEV = { Critical: '#B42318', High: '#B54708', Medium: '#175CD3', Low: '#475467' }; // all ≥ 4.5:1 with white text
const rgb = h => ({ r: parseInt(h.slice(1, 3), 16) / 255, g: parseInt(h.slice(3, 5), 16) / 255, b: parseInt(h.slice(5, 7), 16) / 255 });
for (const s of ['Regular', 'Semi Bold', 'Bold']) await figma.loadFontAsync({ family: 'Inter', style: s });

let page = figma.root.children.find(p => p.name === 'UX Audit Report');
if (!page) { page = figma.createPage(); page.name = 'UX Audit Report'; }
await figma.setCurrentPageAsync(page);
page.children.filter(n => n.name.startsWith('Audit /')).forEach(n => n.remove()); // rebuild cleanly

const txt = (s, size, style = 'Regular', color = '#101828', w) => {
  const t = figma.createText(); t.fontName = { family: 'Inter', style }; t.fontSize = size; t.characters = String(s);
  t.fills = [{ type: 'SOLID', color: rgb(color) }];
  if (w) { t.textAutoResize = 'HEIGHT'; t.resize(w, t.height); }
  return t;
};
const vstack = (name, gap = 16, pad = 0) => {
  const f = figma.createFrame(); f.name = name; f.layoutMode = 'VERTICAL'; f.itemSpacing = gap;
  f.paddingTop = f.paddingBottom = f.paddingLeft = f.paddingRight = pad;
  f.primaryAxisSizingMode = 'AUTO'; f.counterAxisSizingMode = 'AUTO'; f.fills = []; return f;
};

// Summary
const sum = vstack('Audit / Summary', 20, 48);
sum.fills = [{ type: 'SOLID', color: rgb('#FFFFFF') }]; sum.cornerRadius = 16;
sum.appendChild(txt(SUMMARY.title, 40, 'Bold'));
sum.appendChild(txt(`${SUMMARY.productType} · ${SUMMARY.screens.length} screens · ${SUMMARY.date} · Audited by ${SUMMARY.auditor || 'naz'}`, 16, 'Regular', '#475467'));
const counts = {}; ISSUES.forEach(i => counts[i.severity] = (counts[i.severity] || 0) + 1);
const row = figma.createFrame(); row.name = 'Counts'; row.layoutMode = 'HORIZONTAL'; row.itemSpacing = 16; row.fills = [];
row.primaryAxisSizingMode = 'AUTO'; row.counterAxisSizingMode = 'AUTO';
const card = (label, value, color) => { const c = vstack(label, 4, 20); c.fills = [{ type: 'SOLID', color: rgb('#F9FAFB') }]; c.cornerRadius = 12; c.strokes = [{ type: 'SOLID', color: rgb(color) }]; c.strokeWeight = 2; c.appendChild(txt(value, 32, 'Bold', color)); c.appendChild(txt(label, 14, 'Semi Bold', '#475467')); return c; };
row.appendChild(card('Score / 100', SUMMARY.score, '#101828'));
for (const s of Object.keys(SEV)) row.appendChild(card(s, counts[s] || 0, SEV[s]));
sum.appendChild(row);
sum.appendChild(txt('Lens ratings: ' + Object.entries(SUMMARY.lensRatings).map(([k, v]) => `${k}: ${v}`).join('  ·  '), 15, 'Regular', '#344054', 1100));
sum.appendChild(txt('Top priorities', 18, 'Semi Bold'));
sum.appendChild(txt(SUMMARY.priorities.map((p, i) => `${i + 1}. ${p}`).join('\n'), 15, 'Regular', '#344054', 1100));
sum.appendChild(txt('What works well', 18, 'Semi Bold'));
sum.appendChild(txt(SUMMARY.strengths.map(p => `• ${p}`).join('\n'), 15, 'Regular', '#344054', 1100));
sum.appendChild(txt('Score is a conversation starter: 100 − Critical×10 − High×5 − Medium×2 − Low×0.5.', 12, 'Regular', '#667085', 1100));
page.appendChild(sum); sum.x = 0; sum.y = 0;

// Annotated clones
let x = 0; const y = sum.height + 120;
const screenIds = [...new Set(ISSUES.map(i => i.screenId))];
for (const sid of screenIds) {
  const orig = await figma.getNodeByIdAsync(sid);
  if (!orig) continue;
  const wrap = figma.createFrame(); wrap.name = 'Audit / Annotated / ' + orig.name; wrap.fills = []; wrap.clipsContent = false;
  const clone = orig.clone(); wrap.resize(orig.width, orig.height); wrap.appendChild(clone); clone.x = 0; clone.y = 0;
  const ob = orig.absoluteBoundingBox;
  for (const is of ISSUES.filter(i => i.screenId === sid)) {
    const target = await figma.getNodeByIdAsync(is.nodeId);
    const tb = target && target.absoluteBoundingBox; if (!tb) continue;
    const box = figma.createRectangle(); box.name = `Outline #${is.id}`;
    box.resize(Math.max(tb.width, 8), Math.max(tb.height, 8)); box.x = tb.x - ob.x; box.y = tb.y - ob.y;
    box.fills = []; box.strokes = [{ type: 'SOLID', color: rgb(SEV[is.severity]) }]; box.strokeWeight = 2; box.dashPattern = [6, 4];
    const pin = figma.createEllipse(); pin.name = `Pin #${is.id}`; pin.resize(28, 28);
    pin.x = box.x - 14; pin.y = box.y - 14; pin.fills = [{ type: 'SOLID', color: rgb(SEV[is.severity]) }];
    const n = txt(is.id, 12, 'Bold', '#FFFFFF'); n.textAlignHorizontal = 'CENTER'; n.textAlignVertical = 'CENTER';
    n.resize(28, 28); n.x = pin.x; n.y = pin.y;
    wrap.appendChild(box); wrap.appendChild(pin); wrap.appendChild(n);
  }
  page.appendChild(wrap); wrap.x = x; wrap.y = y;
  const label = txt(orig.name, 20, 'Semi Bold'); page.appendChild(label); label.x = x; label.y = y - 40; label.name = 'Audit / Label / ' + orig.name;
  x += orig.width + 120;
}
const bottom = Math.max(y, ...page.children.filter(n => n.name.startsWith('Audit / Annotated')).map(n => n.y + n.height));
return JSON.stringify({ page: page.id, summaryId: sum.id, suggestedTableY: bottom + 160 });
```

Use the returned `suggestedTableY` as `TABLE_Y` in B2.

### B2. Issues table — first call creates the table, later calls append rows

Columns and widths (total 1960px): `#` 56 · Screen / Element 220 · Category 160 · Issue 360 · Why it matters 300 · Severity 120 · Guideline 220 · Recommended solution 524.

```js
const TABLE_Y = 2400;      // below annotated screens
const ROWS = [              // ≤ 20 per call, already sorted by severity then screen
  { id: 1, screen: "Overview", element: "KPI card subtitle", category: "Accessibility", issue: "...", why: "...", severity: "Critical", guideline: "WCAG 1.4.3 Contrast (Minimum)", solution: "..." }
];

const SEV = { Critical: '#B42318', High: '#B54708', Medium: '#175CD3', Low: '#475467' }; // all ≥ 4.5:1 with white text
const COLS = [['#', 56, r => String(r.id)], ['Screen / Element', 220, r => `${r.screen}\n${r.element}`], ['Category', 160, r => r.category], ['Issue', 360, r => r.issue], ['Why it matters', 300, r => r.why], ['Severity', 120, null], ['Guideline', 220, r => r.guideline], ['Recommended solution', 524, r => r.solution]];
const rgb = h => ({ r: parseInt(h.slice(1, 3), 16) / 255, g: parseInt(h.slice(3, 5), 16) / 255, b: parseInt(h.slice(5, 7), 16) / 255 });
for (const s of ['Regular', 'Semi Bold', 'Bold']) await figma.loadFontAsync({ family: 'Inter', style: s });
const page = figma.root.children.find(p => p.name === 'UX Audit Report');
await figma.setCurrentPageAsync(page);

const txt = (s, size, style, color, w) => { const t = figma.createText(); t.fontName = { family: 'Inter', style }; t.fontSize = size; t.lineHeight = { value: 150, unit: 'PERCENT' }; t.characters = String(s ?? ''); t.fills = [{ type: 'SOLID', color: rgb(color) }]; t.textAutoResize = 'HEIGHT'; t.resize(w, t.height); return t; };
const cell = (w, fill) => { const c = figma.createFrame(); c.layoutMode = 'VERTICAL'; c.primaryAxisSizingMode = 'AUTO'; c.counterAxisSizingMode = 'FIXED'; c.resize(w, 10); c.paddingTop = c.paddingBottom = 14; c.paddingLeft = c.paddingRight = 12; c.fills = fill ? [{ type: 'SOLID', color: rgb(fill) }] : []; c.layoutAlign = 'STRETCH'; return c; };
const mkRow = (bg) => { const r = figma.createFrame(); r.layoutMode = 'HORIZONTAL'; r.primaryAxisSizingMode = 'AUTO'; r.counterAxisSizingMode = 'AUTO'; r.fills = [{ type: 'SOLID', color: rgb(bg) }]; r.strokes = [{ type: 'SOLID', color: rgb('#EAECF0') }]; r.strokeBottomWeight = 1; r.strokeTopWeight = r.strokeLeftWeight = r.strokeRightWeight = 0; return r; };

let table = page.findOne(n => n.name === 'Audit / Issues table');
if (!table) {
  table = figma.createFrame(); table.name = 'Audit / Issues table'; table.layoutMode = 'VERTICAL';
  table.primaryAxisSizingMode = 'AUTO'; table.counterAxisSizingMode = 'AUTO'; table.cornerRadius = 12; table.clipsContent = true;
  table.strokes = [{ type: 'SOLID', color: rgb('#D0D5DD') }]; table.fills = [{ type: 'SOLID', color: rgb('#FFFFFF') }];
  page.appendChild(table); table.x = 0; table.y = TABLE_Y;
  const head = mkRow('#101828'); head.name = 'Header';
  for (const [label, w] of COLS) { const c = cell(w); c.appendChild(txt(label, 13, 'Semi Bold', '#FFFFFF', w - 24)); head.appendChild(c); }
  table.appendChild(head);
}
const existing = table.children.length - 1;
ROWS.forEach((r, i) => {
  const row = mkRow((existing + i) % 2 ? '#F9FAFB' : '#FFFFFF'); row.name = `Issue #${r.id}`;
  for (const [label, w, get] of COLS) {
    const c = cell(w);
    if (label === 'Severity') {
      const tag = figma.createFrame(); tag.layoutMode = 'HORIZONTAL'; tag.primaryAxisSizingMode = 'AUTO'; tag.counterAxisSizingMode = 'AUTO';
      tag.paddingLeft = tag.paddingRight = 10; tag.paddingTop = tag.paddingBottom = 4; tag.cornerRadius = 999;
      tag.fills = [{ type: 'SOLID', color: rgb(SEV[r.severity]) }];
      const t = figma.createText(); t.fontName = { family: 'Inter', style: 'Semi Bold' }; t.fontSize = 12; t.characters = r.severity; t.fills = [{ type: 'SOLID', color: rgb('#FFFFFF') }];
      tag.appendChild(t); c.appendChild(tag);
    } else {
      c.appendChild(txt(get(r), 13, label === '#' ? 'Bold' : 'Regular', label === 'Recommended solution' ? '#05603A' : '#344054', w - 24));
    }
    row.appendChild(c);
  }
  table.appendChild(row);
});
return JSON.stringify({ table: table.id, rows: table.children.length - 1 });
```

The severity palette is chosen so white tag text passes 4.5:1 — the report itself should pass the audit.

### B3. Finish
- Add a small legend under the table if useful (severity meanings).
- `get_screenshot` the page's summary and first rows to verify layout; fix overflow if any.
- Give the user the page name and, if available, a link: `https://www.figma.com/design/<fileKey>/?node-id=<summaryId with : → ->`.
