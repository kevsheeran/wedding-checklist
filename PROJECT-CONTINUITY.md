# Wedding Checklist — Project Continuity

## File
`/Users/mark/Desktop/Milk/Mark /Wedding/wedding-checklist.html`
Single-file vanilla HTML + CSS + JS. No build step. Google Fonts only external dependency.

## Architecture

### Storage
- Key: `wedding-checklist-v1` (localStorage)
- Seed data loads once on first run (key absent). Reset available in Settings.
- Export / Import via JSON download / file upload.

### Data model
```js
item = {
  id: string,          // crypto.randomUUID()
  categoryId: string,
  title: string,
  notes: string,
  status: "todo" | "doing" | "done" | "na",
  assignee: "Mark" | "Fiancée" | "Both" | "Family" | "Vendor" | "",
  dueDate: string | null,   // ISO date "YYYY-MM-DD"
  estCost: number | null,   // AED
  tag: "required" | "optional" | "reception-only" | null
}
category = { id, name, order, icon }  // icon = key in ICON_DEFS
```

### State variables (runtime, not persisted)
- `activeCategory` — `'all'` or a category id
- `filters` — `{ search, status, assignee, tag, dueSoon }`
- `_budgetOpen` — whether the budget breakdown panel is expanded
- `_editingItemId` — id of item currently open in the item sheet
- `_catModalId` — id of category currently open in cat modal
- `_catSelectedIcon` — icon key selected in the icon picker

## Design tokens
```css
--ink: #221d17   --rust: #b85733   --rust-dark: #8f3f22
--cream: #f7f2e9  --panel: #f1e9da  --hairline: #e2d9c8
```
Fonts: Cormorant Garamond (headings, italic) + Jost (body, UI).

## Key functions
| Function | Purpose |
|---|---|
| `buildSeedData()` | Returns fresh state with all 13 seed categories + items |
| `render()` | Full re-render: header, cat nav, sidebar, filter bar, main, budget |
| `renderMain()` | Renders category sections with filtered items |
| `applyFilters(items)` | Returns items matching current `filters` state |
| `openItemModal(id?, catId?)` | Opens item add/edit sheet |
| `openCatModal(id?)` | Opens category add/edit modal |
| `toggleBudgetBreakdown()` | Expands/collapses per-category cost breakdown |
| `openDayCapPack()` | Builds DCP HTML, opens new tab, triggers print |
| `buildDCPHtml()` | Assembles A4-ready Day Captain Pack HTML string |
| `moveCat(direction)` | Reorders category by swapping `order` values |
| `catIcon(key, size)` | Returns inline SVG string for a given icon key |

## Icon set
33 icons defined in `ICON_DEFS` object, ordered by `ICON_ORDER` array.
Seed categories use: gavel, file, shield, plane, camera, building, ring, bag, shirt, list, gift, invoice, star.

## Seed version
v1 — 13 categories, ~120 items. SYMBOLIC ceremony items default to `status: "na"`.
Wedding date seeded as `2027-03-15` (editable in Settings).
Budget ceiling seeded as AED 24,900 (editable in Settings).

## Phases completed
- **Phase 1** — Shell, header with progress ring + countdown, category nav (mobile chips + desktop sidebar), seed data, localStorage, budget strip
- **Phase 2** — Item CRUD (add/edit/delete + undo toast), bottom-sheet mobile / right-panel desktop, 3-state checkbox cycle, N/A toggle, Enter to save, Escape to close, form validation highlight
- **Phase 3** — Filter bar (search, status, person, tag, due-soon), category reorder (↑/↓ in edit modal), per-category progress bars
- **Phase 4** — Per-category budget breakdown (expandable), Day Captain Pack (new tab print: handover checklist, reception program, payables table, supplier contacts, day kits)

## Open TODOs / future enhancements
- Drag-to-reorder categories (currently up/down buttons only)
- Push notifications for due-date reminders
- Wedding-day timeline view (time-based ordering for the day itself)
- Drag-to-reorder items within a category
- Item duplication
- Batch status update (select multiple → mark done)
- Collaborative sync (currently local-only)
