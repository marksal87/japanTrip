# Japan Trip Companion App — Project Handoff

## What This Is

A self-contained offline-capable travel companion web app for a trip to Tokyo, Kyoto, and Osaka. It lives as a single `index.html` file hosted on GitHub Pages, saved to the iPhone Home Screen as a PWA (no App Store required). The app combines deep curated research content with the user’s personal Google Maps saved list, GPS-based proximity sorting, and a Claude-powered “ask” feature.

-----

## Tech Stack

- **Single file:** `index.html` — all HTML, CSS, and JS inline, no build step
- **Hosted:** GitHub Pages (public repo), accessed via a stable `github.io` URL
- **Maps:** Leaflet.js + OpenStreetMap tiles (loaded from CDN, requires internet for tiles only)
- **Storage:** `localStorage` for all personal user data (saved places, notes, visited flags)
- **Fonts:** System fonts only (`-apple-system`, `Georgia`) — fully offline
- **No framework, no bundler, no dependencies beyond Leaflet**

-----

## Requirements

### Core

- Works offline once loaded (except map tiles and Claude-powered Ask)
- Installed on iPhone Home Screen via Safari “Add to Home Screen”
- All personal data stays on-device in `localStorage`
- App shell + research content lives in the public GitHub repo — this is fine
- **Private data (hotel addresses, specific stay dates) must NOT be in the GitHub repo** — stored in `localStorage` only, imported by the user via the Refresh panel

### Navigation

- **Explore** — toggles full-screen Leaflet map with coloured pins; tapping again returns to Browse view
- **Browse** (default/home view) — scrollable list of places with filters
- **My List** — shows only saved places in a panel
- **Deep Dive** — city research guides (Tokyo, Kyoto, Osaka) in a panel
- **Refresh** — Google Maps list import + personal data export/restore

### Browse View

- City tabs: Tokyo / Kyoto / Osaka / All
- Category filter pills: All / Food / Drink / Culture / Shopping / Art / Nature / Hidden
- “Nearest on your list” horizontal strip — filters in sync with category pills, hidden when empty, sorted by GPS distance
- Searchable card list sorted by: My list first / A→Z / By category
- Category and nearby filters always stay in sync

### Place Cards

- Name (English + Japanese)
- Category badge + “✓ on list” badge if saved
- Truncated description
- Distance / walk time / price
- Tap → slide-up detail panel

### Detail Panel

- Full description
- Insider tips section
- ☆ Save / ★ Saved toggle (persisted to localStorage)
- Mark visited toggle (persisted)
- Note field (persisted)
- Open in Apple Maps button
- Close via ✕ button (top right) or swipe down

### Map View

- Leaflet + OpenStreetMap, full screen
- Category filter pill strip floats over map
- Colour-coded pins by category; saved places get a green ring
- Tap pin → popup with name, category, “View details →” button → opens detail panel
- City tabs fly map to correct city centre

### Ask Bar

- Suggested prompt chips (e.g. “Find lunch near me”, “Cocktail bar tonight”)
- Free-text input
- Responses shown inline above the category strip
- **Offline:** canned curated responses based on keyword matching
- **Online:** Claude API powered (not yet implemented)

### Refresh Panel (Google Maps Import)

- Paste place names (one per line) from Google Maps saved list
- Matched against research database, merged with saved list
- **Export backup** — downloads `japan-trip-YYYY-MM-DD.json` (saved places, notes, visited flags)
- **Restore backup** — pick JSON file, re-applies personal data; shows toast confirmation

### GPS

- `navigator.geolocation` fires on startup (iOS permission prompt first time)
- Haversine distance calculation to all places with coordinates
- Updates card distances + nearby strip in real time
- Graceful fallback when GPS unavailable

### Data Safety

- `SCHEMA_VERSION = 1` stored in localStorage
- `migrate()` function runs on startup to translate old data shapes forward
- Keys: `japan_schema_v`, `japan_saved`, `japan_notes`, `japan_visited`
- When Claude updates the app: export backup first, update file, restore backup
- Structural changes to place data require bumping SCHEMA_VERSION and updating migrate()

### iPhone / PWA Details

- `<meta name="apple-mobile-web-app-capable" content="yes">`
- `<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">`
- `viewport-fit=cover` in viewport meta — required for safe-area-inset to work on GitHub Pages
- Bottom nav padding: `max(env(safe-area-inset-bottom, 0px) + 8px, 16px)` — sits above home bar
- All JS wrapped in `DOMContentLoaded` listener — required for GitHub Pages (stricter than local file)

-----

## Privacy Architecture

|Data                                       |Location                                   |In GitHub repo?   |
|-------------------------------------------|-------------------------------------------|------------------|
|Research content (place descriptions, tips)|`index.html`                               |✅ Yes — fine      |
|Place coordinates                          |`index.html`                               |✅ Yes — fine      |
|User’s saved/visited/notes                 |`localStorage`                             |❌ No — device only|
|Google Maps imported list                  |`localStorage`                             |❌ No — device only|
|Hotel addresses                            |`localStorage` (imported via Refresh panel)|❌ No — device only|
|Specific travel dates                      |`localStorage` (imported via Refresh panel)|❌ No — device only|

**Implementation note for private data:** Hotel stays and dates should be added as a separate private data layer — a JSON structure the user pastes into the Refresh panel (similar to the Google Maps import), stored in localStorage under a separate key (e.g. `japan_private`). The app reads from this at runtime for features like “near my hotel tonight” but it never touches the HTML file or the repo.

-----

## What’s Done

- [x] Full app shell with all navigation working
- [x] City tabs, category filter pills, search, sort
- [x] Nearby strip (GPS-filtered, synced with category filter)
- [x] Place cards with detail panels
- [x] Save / visited / notes — all persisted to localStorage
- [x] Schema versioning + migration system
- [x] Export / restore personal data as JSON
- [x] Leaflet map with colour-coded pins, category filter, popup → panel
- [x] Explore tab toggles map on/off (icon changes to ☰ Browse when map open)
- [x] City tabs fly map to correct centre + re-filter pins
- [x] GPS distance calculation (Haversine), live distance updates
- [x] Ask bar with canned offline responses
- [x] Google Maps import (paste flow — UI done, matching logic placeholder)
- [x] GitHub Pages deployment fixes (DOMContentLoaded, viewport-fit, safe-area nav)
- [x] Panel close via ✕ button + swipe down + overlay tap

-----

## What’s Left To Do

### High priority (before trip)

- [ ] **Add all real places** — full coordinates, descriptions, tips for Tokyo / Kyoto / Osaka places on the user’s Google Maps list, plus Claude-curated recommendations per category
- [ ] **Deep Dive content** — neighbourhood guides, transport tips, customs, food culture, seasonal info for each city. Consider whether this is flat text or linked to places (e.g. tapping a neighbourhood surfaces all places in that area)
- [ ] **Google Maps import matching** — fuzzy-match pasted place names against the research database and mark as saved; handle unrecognised names gracefully (add as custom place)
- [ ] **Private data layer** — localStorage key for hotels/dates, import via Refresh panel, used for contextual “near my hotel” suggestions
- [ ] **Claude-powered Ask** — wire up Anthropic API for online use; current offline keyword matching stays as fallback
- [ ] **“Near me” GPS state** — graceful UI when location unavailable (show “tap to locate” rather than silently failing)

### Nice to have

- [ ] Custom place entry — add a place not in the database during the trip
- [ ] “Open in Google Maps” as alternative to Apple Maps
- [ ] Day planner / itinerary view
- [ ] Filter by “not yet visited”
- [ ] Visited count / trip progress indicator

-----

## Place Data Structure

```js
{
  id: 1,                        // unique integer — never change once set
  city: 'kyoto',                // 'tokyo' | 'kyoto' | 'osaka'
  cat: 'culture',               // 'food'|'drink'|'culture'|'shopping'|'art'|'nature'|'hidden'
  saved: false,                 // overridden at runtime from localStorage
  name: 'Fushimi Inari Taisha',
  jp: '伏見稲荷大社',
  desc: 'Full description...',
  tips: ['Tip one', 'Tip two', 'Tip three'],
  dist: '—',                    // placeholder; overwritten by GPS at runtime
  walk: '—',                    // placeholder; overwritten by GPS at runtime
  price: 'Free',                // '¥' | '¥¥' | '¥¥¥' | '¥¥¥¥' | 'Free' | range
  icon: '⛩️',
  lat: 34.9671,
  lng: 135.7727
}
```

**When adding places:** always assign the next sequential `id`, never reuse or renumber existing IDs (localStorage references them by id).

-----

## GitHub Repo Workflow

- Repo is public — only research content and app code live there
- `index.html` in root = the entire app
- To update: edit `index.html` in GitHub web editor or via Claude Code, commit to main, GitHub Pages redeploys in ~30 seconds
- Before any structural update: user exports backup from Refresh panel, updates file, restores backup
- Personal data never leaves the device
 
