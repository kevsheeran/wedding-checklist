# Wedding Checklist — Session Notes & Project State

## File
`/Users/mark/Desktop/Milk/Mark /Wedding/wedding-checklist.html`
Single-file vanilla HTML + CSS + JS. No build step. Google Fonts only.

## Live URL
`https://kevsheeran.github.io/wedding-checklist/wedding-checklist.html`

## GitHub
- Repo: `kevsheeran/wedding-checklist` (public)
- Push token: stored in your password manager / Safari keychain (kevsheeran account — ghp_… PAT)
- Push command: `git push https://kevsheeran:<YOUR_TOKEN>@github.com/kevsheeran/wedding-checklist.git main`

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

## Data Model
```js
item = {
  id, categoryId, title, notes,
  status: 'todo' | 'doing' | 'done' | 'na',
  assignee: 'Mark' | 'Fiancée' | 'Both' | 'Family' | 'Vendor' | '',
  dueDate: 'YYYY-MM-DD' | null,
  estCost: number | null,   // AED
  tag: 'required' | 'optional' | 'reception-only' | null,
  starred: boolean,          // priority star
}

category = { id, name, order, icon }

decision = {
  id, itemId,
  finalOptionId: string | null,
  options: [{
    id, name, status, price, notes, conversation,
    details, link, updatedAt
  }]
}
```

## All Features Built (as of last session)

### Core
- [x] PIN lock screen (dark maroon gradient, 4-digit, pre-seeded to 0419)
- [x] GitHub sync — loads `data.json` on unlock, saves on every change
- [x] 13 seed categories, ~120 seed items (Abu Dhabi civil wedding)
- [x] LocalStorage persistence with JSON export/import backup

### Navigation & Views
- [x] Sticky header (monogram, countdown chip, progress ring %)
- [x] View tabs: All / Priority / To Do / Done / Board
- [x] Category nav — horizontal scrolling chips (mobile), sidebar (desktop)
- [x] Filter bar: search, status, assignee, tag, Due Soon toggle
- [x] Board view (Kanban: To Do / Doing / Done columns)
- [x] Priority view (required items, grouped by Overdue / Due Soon / Upcoming)

### Items
- [x] 3-state checkbox cycle: todo → doing → done (tap)
- [x] Item edit bottom sheet (full CRUD: title, notes, status, assignee, due date, cost, tag, category)
- [x] ⭐ Priority star — tap star on item row to pin it to top of category
- [x] 👆 Swipe right on item row → mark done (mobile gesture)
- [x] 📋 Duplicate item button (in edit sheet, copies to same category)
- [x] 📅 Add to Calendar button (in edit sheet when due date set, downloads .ics)
- [x] Mark N/A toggle (for symbolic/ceremony items)
- [x] Undo delete (toast with undo action)

### Vendor Comparison
- [x] Each item can have vendor options (accessed via "Vendor Options" section in item sheet)
- [x] Vendor option fields: name, status, price (AED), notes, communication history, important details, attachment link, last updated
- [x] "Select as Final" → marks item done + fills estimated cost
- [x] "Unselect" button to remove final selection
- [x] Attachment link field (Google Drive, iCloud, any URL) — shows "Open attachment" button on card
- [x] Vendor status badges: Considering / Contacted / Quoted / Visited / Declined / Selected

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

### Other
- [x] 🎉 Confetti animation — fires when a category reaches 100% complete
- [x] 💬 WhatsApp share — chat bubble in header, sends all pending tasks grouped by category
- [x] 🌙 Dark mode toggle (Settings → Display) — deep maroon/charcoal, remembered in localStorage
- [x] Day Captain Pack — printable A4 PDF: handover checklist, payables table, supplier contacts
- [x] Settings panel: wedding date, partner name, budget ceiling, PIN management, GitHub token, data export/import, reset

## What Could Be Done Next (ideas discussed)

### High value
- [ ] **Notification / push reminder** — remind x days before due date (would need a service worker)
- [ ] **Timeline view** — show items on a horizontal date timeline
- [ ] **Shared access** — read-only link for partner to view progress without PIN

### Nice to have
- [ ] **Drag to reorder** — drag-and-drop items within a category (currently no reorder)
- [ ] **Drag to reorder categories** — (currently up/down buttons only)
- [ ] **Batch status update** — select multiple items → mark all done
- [ ] **Item duplication across categories** — current duplicate stays in same category
- [ ] **Budget pie chart** — visual breakdown instead of bar
- [ ] **Recurring items** — e.g. "Follow up with hotel" every 2 weeks
- [ ] **Search in vendor options** — can't currently search inside vendor details

### App store / PWA
- [ ] **Add to Home Screen (PWA)** — service worker + manifest so it installs like an app
- [ ] **Offline mode** — works without internet after first load (service worker cache)

## Key Functions Reference
| Function | Purpose |
|---|---|
| `render()` | Full re-render: header, nav, filter, main, budget |
| `renderMain()` | Category sections + item rows (also checks confetti) |
| `renderItem(item)` | Returns a single item row DOM element |
| `openItemModal(id?, catId?)` | Opens item add/edit bottom sheet |
| `saveItemModal()` | Validates + saves item, closes sheet |
| `openDecisionSheet(itemId)` | Opens vendor comparison overlay |
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
| `hardResetApp()` | Clears all localStorage keys + reloads (was removed from UI) |

## Notes
- Gilda Display only supports `font-weight: 400` — never set it higher
- `backdrop-filter: blur()` is only on modals and header — NOT on item rows (causes GPU lag)
- `decisions[]` is keyed by `itemId` (not categoryId — was changed from category-level)
- Decision sheet z-index: 250 (above item sheet at 200)
- The `add-item-btn` inline is hidden on mobile (`display:none`) — FAB handles add on mobile
- `_prevCatProgress` object tracks previous category % to know when confetti should fire
