# CLAUDE.md — Card Pack Opener

## Project Overview

**Card Pack Opener** is a single-file web application that simulates opening trading card packs for three games: Pokémon TCG, Magic: The Gathering, and Yu-Gi-Oh!. It uses real card data from public APIs, supports a swipe-to-tear pack mechanic, and stores a persistent collection in `localStorage`.

**Key fact:** The entire application is contained in a single file — `index.html` — with no build system, no dependencies, and no package manager.

---

## Repository Structure

```
card-pack-opener/
└── index.html     # The entire application (HTML + CSS + JS, ~1115 lines)
```

There is no `package.json`, `node_modules`, build output, or test suite. The file can be opened directly in a browser or served with any static file server.

### Layout of `index.html`

| Lines      | Content                                                     |
|------------|-------------------------------------------------------------|
| 1–6        | HTML boilerplate, meta tags, viewport                       |
| 7–226      | CSS: variables, game themes, screen layouts, animations     |
| 228–382    | HTML markup: 4 app screens                                  |
| 384–593    | `GAMES` object: all game/set definitions                    |
| 596–651    | State variables and `localStorage` collection helpers        |
| 653–799    | Screen navigation, swipe/tear mechanics                     |
| 800–965    | Card reveal sequence and pack completion logic              |
| 967–1051   | Confetti animation and collection rendering                 |
| 1052–1112  | Collection search/sort/filter and utility functions         |
| 1112       | App initialization                                          |

---

## Application Architecture

### Single-Page Application (SPA) Pattern

The app has four screens that are toggled by setting `display: none/flex`:

| Screen ID        | Purpose                                    |
|------------------|--------------------------------------------|
| `#s-home`        | Game selection (Pokémon, MTG, Yu-Gi-Oh!)  |
| `#s-sets`        | Set selection for the chosen game          |
| `#s-pack`        | Interactive pack opening experience        |
| `#s-collection`  | View all cards collected across games      |

Navigation is handled by `showScreen(id)`, which hides all screens then shows the target.

### State Variables

```javascript
let activeGame   = null;   // Selected game key: 'pokemon' | 'mtg' | 'ygo'
let activeSet    = null;   // Selected set definition object from GAMES
let packCards    = [];     // Array of card objects for current pack
let revealed     = 0;      // Number of cards revealed so far
let swipeActive  = false;  // Whether the swipe/tear is in progress
let packOpened   = false;  // Whether pack has been fully opened
let collGameFilter = 'all'; // Collection view filter
```

### Data Persistence

Collection is stored in `localStorage`:

- **Key:** `cpo_collection` — JSON array of `{ card, count, addedAt }` entries
- **Key:** `cpo_packs` — Integer count of total packs opened

---

## Game Definitions (`GAMES` Object)

Each game entry defines:

```javascript
GAMES[gameId] = {
  label: string,         // Display name
  color: string,         // CSS hex accent color
  g1: string,            // Gradient start color
  g2: string,            // Gradient end color
  sets: [{
    id: string,          // Set identifier used in API queries
    label: string,       // Display name
    year: number,        // Release year
    total: number,       // Total cards in set
    packSize: number,    // Cards per pack
    fetch: async () => [], // Function that fetches and returns cards array
    fillPack: (cards) => [], // Function that builds a pack from fetched cards
  }]
}
```

### Supported Games and APIs

| Game        | Key       | API                                          | Price Source            |
|-------------|-----------|----------------------------------------------|-------------------------|
| Pokémon TCG | `pokemon` | `https://api.pokemontcg.io/v2/cards`         | TCGPlayer market price  |
| MTG         | `mtg`     | `https://api.scryfall.com/cards/search`      | Scryfall USD price      |
| Yu-Gi-Oh!   | `ygo`     | `https://db.ygoprodeck.com/api/v7/cardinfo.php` | TCGPlayer market price |

**API note for MTG (Scryfall):** Requests are paginated (up to 8 pages) with an 80ms delay between pages to respect rate limits. Do not remove or reduce this delay.

### Pack Composition

| Game    | Pack Size | Rarity Slots                                                              |
|---------|-----------|---------------------------------------------------------------------------|
| Pokémon | 10        | 1 rare (33% holo), 3 uncommon, 5 common, 1 energy                        |
| MTG     | 15        | 1 rare/mythic (12.5% mythic), 3 uncommon, 10 common, 1 land              |
| Yu-Gi-Oh! | 9       | 1 top (5% secret, 20% ultra, 30% super, 45% rare), 2 super/rare, 6 common |

---

## Key Functions Reference

### Navigation
- `showScreen(id)` — Show one of the four screens
- `selectGame(gameId)` — Set active game, populate set list, navigate to sets screen
- `selectSet(gameId, setId)` — Fetch cards for a set, prepare pack, navigate to pack screen

### Pack Opening Mechanics
- `initSwipe()` — Attach touch/mouse event listeners for the tear mechanic
- `moveSwipe(e)` — Update tear progress bar and animate scissors/pack
- `triggerPackOpen()` — Animate pack flip, transition to card reveal UI
- `revealNext()` — Reveal one card with flip animation
- `revealAll()` — Reveal all remaining cards
- `finishPack()` — Calculate pack value, save cards to collection, show results

### Collection
- `getCollection()` — Read and parse `localStorage` collection
- `saveCollection(col)` — Write collection to `localStorage`
- `addToCollection(card)` — Add a single card (increment count if duplicate)
- `renderCollection()` — Render all collection cards with current filter/sort
- `filterCollByGame(g)` / `sortColl(val)` / `searchColl(q)` — Collection view controls

### Utilities
- `apiFetch(url)` — Fetch with in-memory caching (`cardCache` object)
- `fireConfetti(count)` — Spawn confetti particles for celebrations
- `fmtPrice(n)` — Format a number as a USD price string

---

## CSS Conventions

### CSS Custom Properties (Variables)

Defined on `:root` and overridden per game via `data-game` attribute:

```css
--bg        /* Page background color */
--surface   /* Card/panel surface color */
--text      /* Primary text color */
--accent    /* Game accent color (set via JS on <body>) */
--radius    /* Border radius (8px) */
--g1, --g2  /* Gradient start/end colors for game theme */
```

### Naming Patterns

- **Screen IDs:** `s-{name}` (e.g., `s-home`, `s-pack`)
- **Component classes:** hyphenated lowercase (e.g., `game-card`, `c-tile`, `pack-result`)
- **JS-set inline styles:** Used for dynamic game theming colors

### Animations (Keyframes)

| Name          | Purpose                                    |
|---------------|--------------------------------------------|
| `card-flip`   | Card reveal flip (rotateY 90°→0°)          |
| `holo-shimmer`| Shimmer effect on rare cards               |
| `confetti-fall`| Confetti particle drop                    |
| `pack-shake`  | Pack trembles during swipe tear            |
| `slideUp`     | Screen transition entrance                 |

---

## Development Workflow

### Running Locally

No build step required. Open the file directly:

```bash
open index.html
# or serve it:
python3 -m http.server 8080
# then visit http://localhost:8080
```

### Making Changes

Since all code is in `index.html`, edit that file directly. There is no compilation, transpilation, or bundling step.

**Recommended approach for larger changes:**
1. Identify the relevant line range (see the layout table above)
2. Edit only within that range
3. Test by opening `index.html` in a browser
4. Verify all four screens still function: home, sets, pack opening, collection

### No Linting or Formatting Tools

There is no ESLint, Prettier, or other formatter configured. Follow the existing code style:
- 2-space indentation
- `let`/`const` (no `var`)
- Arrow functions for callbacks
- Template literals for string interpolation
- Semicolons used throughout

---

## Testing

**There is no automated test suite.** All testing is manual. When making changes, verify:

1. **Home screen** — All three game cards display and are clickable
2. **Set screen** — Sets load for each game; collection progress shown
3. **Pack opening** — Swipe tear works (mouse drag and touch); cards reveal one at a time and all-at-once
4. **Collection** — Cards added to collection; search, sort, and filter work; count increments on duplicates
5. **API calls** — Network requests succeed and card images load
6. **localStorage** — Collection persists across page reloads; packs-opened counter increments

**Browser compatibility target:** Modern evergreen browsers (Chrome, Firefox, Safari, Edge). No IE11 support needed.

---

## Common Pitfalls

1. **Scryfall rate limiting** — The 80ms delay between API pages (`await new Promise(r=>setTimeout(r,80))`) is intentional. Do not remove it or MTG set fetches will get rate-limited.

2. **Single file constraint** — Do not refactor into multiple files unless specifically asked. The project's simplicity is a feature.

3. **No module system** — There are no `import`/`export` statements. All functions are in global scope. Be careful about naming conflicts when adding new functions.

4. **localStorage limits** — Large collections may approach browser storage limits. The current implementation does no pruning; be mindful when adding card data size.

5. **API key for Pokémon** — The Pokémon TCG API works without a key at low rate limits. If heavy testing causes throttling, an API key can be added to the `apiFetch` headers.

6. **Image loading** — Card images use `loading="lazy"` and have `onerror` fallbacks (emoji). Test image fallback behavior when working on card rendering code.

---

## Adding a New Game

To add a new trading card game:

1. Add an entry to the `GAMES` object with `label`, `color`, `g1`, `g2`, and `sets` array
2. Each set needs `id`, `label`, `year`, `total`, `packSize`, `fetch()`, and `fillPack()` functions
3. Add a game card button in the `#s-home` screen HTML
4. Define the game's rarity tiers and pack composition logic in `fillPack()`
5. Test all three phases: set selection, pack opening, and collection display

## Adding a New Set to an Existing Game

1. Find the game's entry in `GAMES`
2. Add a new object to the `sets` array with the appropriate fields
3. Implement `fetch()` using the game's existing API pattern
4. Implement `fillPack()` using the game's rarity distribution rules

---

## Git Conventions

Commit messages follow an imperative style:
- `Add feature X`
- `Fix bug in Y`
- `Refactor Z for clarity`

Branch naming: `claude/<description>-<id>` (used for AI-generated changes).
