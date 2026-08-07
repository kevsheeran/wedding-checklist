# Wedding Checklist — Session Notes & Project State

**Last updated: 2026-08-08**
**Last commit: `2828dbf` — "Remove Blocked By / task dependencies feature"**

## File
`/Users/mark/Desktop/Milk/Mark /Wedding/wedding-checklist.html`
Single-file vanilla HTML + CSS + JS. No build step. Google Fonts only.

## Live URL
`https://kevsheeran.github.io/wedding-checklist/wedding-checklist.html`

## GitHub
- Repo: `kevsheeran/wedding-checklist` (public)
- Push command (replace TOKEN with your GitHub PAT):
  ```
  git -C "/Users/mark/Desktop/Milk/Mark /Wedding" push https://kevsheeran:TOKEN@github.com/kevsheeran/wedding-checklist.git main
  ```
- If push rejected (remote ahead): run these first, then push:
  ```
  git -C "/Users/mark/Desktop/Milk/Mark /Wedding" fetch https://kevsheeran:TOKEN@github.com/kevsheeran/wedding-checklist.git main
  git -C "/Users/mark/Desktop/Milk/Mark /Wedding" merge FETCH_HEAD --no-edit
  ```
- Token is stored in the app's localStorage (`gh-token-v1`) and in your local SESSION-NOTES-private.md (not pushed)

## PIN & Auth
- App PIN: `0419`
- GitHub token stored in localStorage key `gh-token-v1`
- Flow: Enter PIN → GitHub token prompt appears (if no token) → loads data.json from GitHub

## localStorage Keys
| Key | Purpose |
|---|---|
| `wedding-checklist-v1` | All app data (categories, items, decisions) |
| `wedding-pin-v1` | PIN hash (currently `0419`) |
| `gh-token-v1` | GitHub PAT for sync |
| `wedding-dark-v1` | Dark mode preference (`1` = on) |
| `wn-seen-v1` | What's New badge dismissed (`1` = seen) |

## Tech Stack
- Vanilla HTML + CSS + JS (no framework, no build step)
- Google Fonts: **Gilda Display** (headings, italic) + **Raleway** (body/UI)
- GitHub Contents API for cloud sync (PUT to push, raw.githubusercontent.com to pull)
- Canvas API for confetti
- LocalStorage for all persistence

## CSS Design Tokens
```css
--ink: #1e1715
--rust: #7a1a2e       (maroon — wedding colour)
--rust-dark: #58121e
--rust-pale: rgba(122,26,46,0.09)
--cream: #faf3e8      (beige — wedding colour)
--panel: #f2e6d2
--hairline: #ddd1b8
--radius-sm: 10px  --radius-md: 14px  --radius-lg: 20px
```

## Branding
- **Monogram:** K & C (header top-left)
- **Header subtitle:** Wedding Planner
- **Wedding date:** Abu Dhabi 2027 (used for countdown)

## Data Model
```js
item = {
  id, categoryId, title, notes,
  status: 'todo' | 'doing' | 'done' | 'na',
  assignee: 'Mark' | 'Fiancée' | 'Both' | 'Family' | 'Vendor' | '',
  dueDate: 'YYYY-MM-DD' | null,
  estCost: number | null,   // AED
  tag: 'required' | 'optional' | 'reception-only' | null,
  starred: boolean,
  // NOTE: blockedBy field has been REMOVED entirely
}

category = { id, name, order, icon }

decision = {
  id, itemId,
  finalOptionId: string | null,
  options: [{
    id, name, status,
    price,          // AED — used for budget when "Select as Final"
    notes, conversation, details, link, updatedAt,
    // Venue & Reception extra fields:
    minGuests, pricePerPerson, quoteFor100, agentName, mobile, telephone, email,
    // Prenup Shoot extra fields:
    photoPackage, coverageHours, deliverables, travelFee, contactPerson, mobile, email,
    // Suppliers & Payments extra fields:
    serviceType, packageName, inclusions, trialDate, contactPerson, mobile, email,
    // Attire & Rings extra fields:
    shopType, alterationFee, firstFittingDate, finalFittingDate, deliveryDate, contactPerson, mobile, email,
  }]
}
```

## Vendor Comparison System

### How it works
- Accessed from any item via "Vendor Options" in the item sheet
- Opens a full-screen decision overlay (`#decision-overlay`) with two panels: list + form
- `getCatType(itemId)` detects category from category name string match → returns `'venue' | 'photo' | 'supplier' | 'attire' | null`
- Category-specific extra fields shown/hidden based on catType

### Vendor statuses
`Proposal Requested` · `Quoted` · `Contacted` · `Considering` · `Visited` · `Declined`

### Context-aware extra sections (shown in form below Status field)
| Category match | Section shown | Key price field |
|---|---|---|
| `venue` (Venue & Reception) | Hotel Details | `v-quote` (100-pax quote) → `opt.price` |
| `prenup` (Prenup Shoot) | Photography Details | no auto-price (set item cost manually) |
| `supplier` (Suppliers & Payments) | Service Details | no auto-price |
| `attire` (Attire & Rings) | Boutique/Tailor Details | no auto-price |
| anything else | none | no auto-price |

### Venue & Reception fields (IDs)
`v-min-guests` · `v-ppp` (price/pax) · `v-quote` (100-pax auto-calc) · `v-agent` · `v-mobile` · `v-tel` · `v-email`

**Auto-compute:** `v-ppp × v-min-guests (default 100) → v-quote` — fires on input of either field.

### Photography fields (IDs)
`ph-package` · `ph-hours` · `ph-deliverables` · `ph-travel-fee` · `ph-contact` · `ph-mobile` · `ph-email`

### Supplier fields (IDs)
`sp-service-type` · `sp-package` · `sp-inclusions` · `sp-trial-date` · `sp-contact` · `sp-mobile` · `sp-email`

### Attire fields (IDs)
`at-shop-type` · `at-alteration-fee` · `at-first-fitting` · `at-final-fitting` · `at-delivery` · `at-contact` · `at-mobile` · `at-email`

### Budget integration
- `selectFinalOption(optId)` copies `opt.price` → `item.estCost` for budget tracking
- For Venue: `opt.price` is auto-set from `v-quote` (100-pax quote) when saving
- For Photo / Supplier / Attire: `opt.price` = null — set `item.estCost` in the item sheet manually
- **No generic "Total Quote (AED)" field in the form** — price comes from the category-specific section

### `saveDecOption()` — price derivation logic
```js
if (ct === 'venue')    return { price: gn('v-quote'), /* + hotel fields */ };
if (ct === 'photo')    return { price: null, /* + photo fields */ };
if (ct === 'supplier') return { price: null, /* + supplier fields */ };
if (ct === 'attire')   return { price: null, /* + attire fields */ };
return { price: null };
```

### Vendor form header layout (current)
- Back arrow ← on the left
- Form title (flex:1) in the middle
- **Duplicate** pill button (top-right, hidden when new)
- **Delete** pill button (top-right danger style, hidden when new)
- Full-width **Draft enquiry email** action button above footer
- Footer: Unselect | Select as Final | Save Option

### Last Updated field
- Hidden `<input id="opt-updated" type="hidden">` — auto-set to today's date on save
- Read-only display `<div id="opt-updated-display">` shown below form title when editing existing option
- Never manually editable — always stamped by `saveDecOption()`

## Vendor Form UX (as of last session)
- Delete + Duplicate moved to header bar as pill buttons (not inline with form fields)
- Draft Email = full-width button above the footer (not crammed with save)
- Footer = clean: only Unselect / Select as Final / Save Option
- Last Updated = auto-stamped, read-only display

## What's New Feature
- Info circle button in header (before Settings icon)
- Red dot badge on first view — badge class `.new-badge` removed on open, flag stored in `localStorage('wn-seen-v1','1')`
- Modal `#whats-new-overlay` lists all features grouped by section
- On boot: if `wn-seen-v1` already set, badge class not added

## All Features Built

### Core
- [x] PIN lock screen (dark maroon gradient, 4-digit, pre-seeded to 0419)
- [x] GitHub sync — loads `data.json` on unlock, saves on every change
- [x] 13 seed categories, ~120 seed items (Abu Dhabi civil wedding)
- [x] LocalStorage persistence with JSON export/import backup

### Navigation & Views
- [x] Sticky header (monogram K & C, countdown chip, progress ring %)
- [x] View tabs: All / Priority / To Do / Done / Board
- [x] Category nav — horizontal scrolling chips (mobile), sidebar (desktop)
- [x] Filter bar: search, status, assignee, tag, Due Soon toggle
- [x] Board view (Kanban: To Do / Doing / Done columns)
- [x] Priority view (required items, grouped by Overdue / Due Soon / Upcoming)

### Items
- [x] 3-state checkbox cycle: todo → doing → done (tap)
- [x] Item edit bottom sheet (full CRUD: title, notes, status, assignee, due date, cost, tag, category)
- [x] Priority star — tap star on item row to pin it to top of category
- [x] Swipe right on item row → mark done (mobile gesture)
- [x] Duplicate item button (in edit sheet, copies to same category)
- [x] Add to Calendar button (in edit sheet when due date set, downloads .ics)
- [x] Mark N/A toggle (for symbolic/ceremony items)
- [x] Undo delete (toast with undo action)

### Vendor Comparison
- [x] Each item can have vendor options (accessed via "Vendor Options" in item sheet)
- [x] Status: Proposal Requested / Quoted / Contacted / Considering / Visited / Declined
- [x] Context-aware extra fields per category (Venue / Photo / Supplier / Attire)
- [x] "Select as Final" → marks item done + fills estimated cost (from category price field)
- [x] "Unselect" button to remove final selection
- [x] Attachment link field (Google Drive, iCloud, any URL) — shows "Open attachment" on card
- [x] Notes, communication history, important details fields
- [x] Auto-compute: price/pax × guests → 100-pax quote (Venue only)
- [x] Delete + Duplicate in form header bar (pill buttons)
- [x] Draft Enquiry Email — full-width action button above footer
- [x] Last Updated — auto-stamped on save, read-only display

### Budget
- [x] Budget strip (sticky bottom) — total estimated vs ceiling
- [x] Over-budget toast alert (fires once when total crosses ceiling)
- [x] Per-category budget breakdown (expandable)
- [x] Budget ceiling editable in Settings

### Categories
- [x] Add / edit / delete categories
- [x] Icon picker (33 icons)
- [x] Reorder (↑/↓ buttons in category modal)
- [x] Per-category progress bar

### Mobile UX
- [x] Bottom-sheet modals (item, category, vendor)
- [x] Mobile FAB (+ button fixed bottom-right, adds item to current category)
- [x] Sticky category chips while scrolling
- [x] iOS Safari font-size fix (16px on inputs, prevents auto-zoom)
- [x] Touch swipe-to-done gesture
- [x] Category header flex-wrap on narrow screens (no overflow)

### Other
- [x] Confetti animation — fires when a category reaches 100% complete
- [x] WhatsApp share — chat bubble in header, sends all pending tasks grouped by category
- [x] Dark mode toggle (Settings → Display) — deep maroon/charcoal, remembered in localStorage
- [x] Day Captain Pack — printable A4 PDF: handover checklist, payables table, supplier contacts
- [x] Settings panel: wedding date, partner name, budget ceiling, PIN management, GitHub token, data export/import, reset
- [x] What's New button in header — red dot badge, dismisses on open, lists all features

### Removed
- ~~Blocked By / task dependencies~~ — removed entirely (CSS, HTML, JS all cleaned up)

## What Could Be Done Next

### High value
- [ ] **Notification / push reminder** — remind x days before due date (would need a service worker)
- [ ] **Timeline view** — show items on a horizontal date timeline
- [ ] **Shared access** — read-only link for partner to view progress without PIN

### Nice to have
- [ ] **Drag to reorder** — drag-and-drop items within a category (currently no reorder)
- [ ] **Batch status update** — select multiple items → mark all done
- [ ] **Budget pie chart** — visual breakdown instead of bar
- [ ] **Search in vendor options** — can't currently search inside vendor details

## Key Functions Reference
| Function | Purpose |
|---|---|
| `render()` | Full re-render: header, nav, filter, main, budget |
| `renderMain()` | Category sections + item rows (also checks confetti) |
| `renderItem(item)` | Returns a single item row DOM element |
| `openItemModal(id?, catId?)` | Opens item add/edit bottom sheet |
| `saveItemModal()` | Validates + saves item, closes sheet |
| `openDecisionSheet(itemId)` | Opens vendor comparison overlay |
| `showDecFormPanel(optionId?)` | Opens vendor option form, populates fields by catType |
| `saveDecOption()` | Saves option; derives price from category field; auto-stamps updatedAt |
| `getCatType(itemId)` | Returns `'venue'|'photo'|'supplier'|'attire'|null` |
| `selectFinalOption(optId)` | Marks vendor as final, marks item done, updates cost |
| `fireConfetti()` | Canvas confetti burst |
| `shareWhatsApp()` | Builds + opens WhatsApp share URL |
| `addToCalendar()` | Generates + downloads .ics file for item due date |
| `duplicateItem()` | Copies current editing item (adds " (Copy)") |
| `toggleDarkMode()` | Toggles body.dark class + persists to localStorage |
| `loadFromGitHub()` | Fetches data.json from GitHub raw URL |
| `saveToGitHub()` | PUTs state JSON to GitHub Contents API |
| `initPinLock()` | Pre-seeds PIN 0419, shows lock screen |
| `checkPinEntry()` | Validates PIN, triggers GitHub load on success |
| `showSyncPrompt()` | Shows GitHub token modal (if no token stored) |
| `buildSeedData()` | Returns fresh state with all 13 seed categories + items |

## Notes
- Gilda Display only supports `font-weight: 400` — never set it higher
- `backdrop-filter: blur()` is only on modals and header — NOT on item rows (causes GPU lag)
- `decisions[]` is keyed by `itemId` (not categoryId)
- Decision sheet z-index: 250 (above item sheet at 200)
- The `add-item-btn` inline is hidden on mobile (`display:none`) — FAB handles add on mobile
- `_prevCatProgress` object tracks previous category % to know when confetti should fire
- Remote GitHub repo gets "Update checklist" commits from in-app data.json syncs — always `git fetch` + merge before pushing if rejected
- `loadState()` migration guards: always check for missing `timeline` and `dismissedMilestones` arrays
- `applyFilters`: always use `(item.notes || '').toLowerCase()` — user-created items may have no notes
- Priority view empty state: use `appendChild` not `innerHTML =` to avoid wiping the milestone banner
