# Cartridge — Build Task List

> Every task is atomic. Every task has a pass condition.
> Do not begin a task until the previous one passes.
> Do not move to the next phase until every task in the current phase passes.
>
> **For AI agents:** Any task with a `🤔 Decisions Required` block must stop and ask the user those specific questions before writing a single line of code. Do not assume. Do not use the suggested defaults without confirmation.

**Stack:** Tauri v2 · React 18 · TypeScript · SQLite · Supabase
**Phases:** 11 · **Tasks:** 89

---

## Table of Contents

- [Phase 0 — Decisions & Architecture](#phase-0--decisions--architecture)
- [Phase 1 — Project Scaffold](#phase-1--project-scaffold)
- [Phase 2 — Core UI Shell](#phase-2--core-ui-shell)
- [Phase 3 — Library Management](#phase-3--library-management)
- [Phase 4 — Emulator Integration](#phase-4--emulator-integration)
- [Phase 5 — Metadata & Box Art](#phase-5--metadata--box-art)
- [Phase 6 — RetroAchievements Integration](#phase-6--retroachievements-integration)
- [Phase 7 — Social Backend](#phase-7--social-backend)
- [Phase 8 — Social UI](#phase-8--social-ui)
- [Phase 9 — In-Game Overlay *(Stretch Goal)*](#phase-9--in-game-overlay-stretch-goal)
- [Phase 10 — Polish & Ship](#phase-10--polish--ship)

---

## Tag Legend

| Tag | Meaning |
|---|---|
| `[RUST]` | Tauri / Rust backend work |
| `[UI]` | Frontend component |
| `[DB]` | Database work |
| `[API]` | External API integration |
| `[INFRA]` | Infrastructure / config |
| `[HARD]` | High complexity — budget extra time |

---

## Phase 0 — Decisions & Architecture

> Nothing gets built until every decision is documented. Agents need explicit answers — they will invent their own if you don't provide them.

---

### T0.1 — Write and commit the Architecture Decision Record (ADR) `[INFRA]`

Create a single `ARCHITECTURE.md` file at the root of the repo documenting every major technology choice, its rationale, and what is explicitly out of scope for v1.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Desktop framework:** This plan uses Tauri v2. Do you confirm, or do you prefer Electron or another framework?
> 2. **UI framework:** This plan uses React 18 + TypeScript. Do you confirm, or do you prefer Vue, Svelte, or something else?
> 3. **Backend provider:** This plan uses Supabase. Do you confirm, or do you prefer Firebase, PocketBase, a self-hosted option, or no backend at all for v1?
> 4. **Target platforms for v1:** This plan targets Windows only. Do you want to also target macOS and/or Linux from the start?
> 5. **Emulator backend:** This plan uses RetroArch portable. Do you confirm, or do you want to support standalone emulators (e.g., separate Snes9x, PCSX2) instead of or alongside RetroArch?
> 6. **Metadata source:** This plan uses ScreenScraper for box art and game info. Do you confirm, or do you prefer IGDB, TheGamesDB, or another source?

**Pass Criteria**
- [ ] File exists at repo root and is committed to git
- [ ] Every technology choice has a one-sentence rationale
- [ ] Out-of-scope items are listed explicitly
- [ ] Any teammate or agent can read it and know exactly what to use without asking

---

### T0.2 — Register all required external accounts and store credentials `[INFRA]`

Register accounts and obtain API credentials for all external services confirmed in T0.1. Create `.env.local` (gitignored) and `.env.example` (committed with placeholder values).

> 🤔 **Decisions Required — Ask the user:**
> 1. Do you already have accounts for any of these services (ScreenScraper, RetroAchievements, Supabase)? If so, which ones?
> 2. Will a single Supabase project be used for both dev and production, or do you want separate projects for each environment?

**Pass Criteria**
- [ ] `.env.example` exists in repo with all required variable names
- [ ] `.env.local` is gitignored and contains real credentials
- [ ] A manual API call to each service returns a valid response — not an auth error

---

### T0.3 — Create the database schema diagram before writing any migration `[DB]`

Design and document the full database schema (local SQLite + Supabase Postgres) before writing a single line of code. Store as `SCHEMA.md`.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Local tables:** This plan includes `games`, `platforms`, `play_sessions`, `save_states`, `achievements_cache`. Are there any tables to add or remove?
> 2. **Supabase tables:** This plan includes `profiles`, `friendships`, `presence`, `messages`, `activity_feed`. Are there any to add or remove?
> 3. Should the `games` table store a user-assigned rating (e.g., 1–5 stars) in v1?
> 4. Should play sessions track which save state was loaded at the start of the session?

**Pass Criteria**
- [ ] `SCHEMA.md` exists and documents every table and column
- [ ] Primary keys, foreign keys, and indexes are all specified
- [ ] A second person can read it and write the migration SQL without asking questions

---

## Phase 1 — Project Scaffold

> A blank but correctly configured project. No features. Just the right bones so nothing has to be retrofitted later.

---

### T1.1 — Install all system prerequisites on the dev machine `[INFRA]`

Install Rust stable via `rustup`, Node.js 20 LTS, Visual Studio Build Tools 2022 with "Desktop Development with C++" workload, and WebView2. Verify all versions match Tauri v2 prerequisites.

**Pass Criteria**
- [ ] `rustc --version` returns stable 1.77+
- [ ] `node --version` returns 20.x
- [ ] `npm --version` returns 10.x
- [ ] Running `cargo tauri info` shows no missing prerequisites

---

### T1.2 — Initialize the Tauri v2 + React + TypeScript project `[INFRA]`

Run `npm create tauri-app@latest cartridge -- --template react-ts`. Initialize a git repository and `.gitignore`. Make an initial commit.

> 🤔 **Decisions Required — Ask the user:**
> 1. What should the git remote be? (GitHub repo URL, or set it up later?)
> 2. Should the initial branch be named `main` or something else?

**Pass Criteria**
- [ ] `npm run tauri dev` opens a native window with the default React app — no errors in terminal
- [ ] `npm run tauri build` produces a working `.msi` installer without errors
- [ ] Git log shows exactly one commit

---

### T1.3 — Configure TypeScript strict mode and path aliases `[INFRA]`

Enable `"strict": true` in `tsconfig.json`. Add path aliases and configure them in `vite.config.ts`.

> 🤔 **Decisions Required — Ask the user:**
> 1. This plan uses: `@/` → `src/`, `@components/` → `src/components/`, `@store/` → `src/store/`, `@lib/` → `src/lib/`. Do you want to use these, change any, or add more?

**Pass Criteria**
- [ ] `npm run build` passes with zero TypeScript errors on the default scaffold
- [ ] A test import using `@/` resolves correctly — no "cannot find module" error

---

### T1.4 — Install and configure Tailwind CSS v3 `[UI]`

Install Tailwind CSS, PostCSS, and Autoprefixer. Configure content paths and extend the theme with Cartridge's design tokens.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Fonts:** This plan uses Outfit (display) and JetBrains Mono (monospace). Do you confirm these, or do you want different fonts?
> 2. **Accent color:** This plan uses a blue accent (`#60a5fa`). Do you want a different default accent color?
> 3. Should the theme include a light mode variant from the start, or dark mode only for v1?

**Pass Criteria**
- [ ] A component using a Tailwind class like `bg-blue-500` visibly applies that color in the running app
- [ ] A component using a custom design token from the extended theme applies correctly
- [ ] No Tailwind warnings in the console

---

### T1.5 — Set up ESLint, Prettier, and pre-commit hooks `[INFRA]`

Install and configure ESLint, Prettier, and Husky with lint-staged pre-commit hooks.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Prettier style:** This plan uses single quotes, no semicolons, 2-space indent. Do you confirm, or do you have different preferences?
> 2. Should the pre-commit hook block commits on lint **warnings**, or only on **errors**?

**Pass Criteria**
- [ ] `npm run lint` runs with zero errors on the fresh scaffold
- [ ] Intentionally introducing a lint error and trying to commit is blocked by the pre-commit hook

---

### T1.6 — Configure the Tauri window: frameless, fullscreen-capable, correct defaults `[RUST]`

Configure the main window in `tauri.conf.json`. Implement a custom drag region and custom window controls (close, minimize, maximize).

> 🤔 **Decisions Required — Ask the user:**
> 1. **Default window size:** This plan uses 1280×800. Do you confirm, or do you prefer a different default size?
> 2. **Minimum window size:** This plan uses 1024×640. Do you confirm, or want a different minimum?
> 3. Should the app launch **maximized** by default, or at the configured default size?
> 4. Should the app **remember its window size and position** between sessions?

**Pass Criteria**
- [ ] Window opens with no system title bar
- [ ] Window can be dragged by clicking the custom drag region
- [ ] Custom close button closes the app
- [ ] Window cannot be resized below the confirmed minimum size

---

### T1.7 — Install and initialize the Tauri SQLite plugin `[DB]` `[RUST]`

Add `tauri-plugin-sql` to `Cargo.toml` with the `sqlite` feature flag. Establish the database connection using a path inside Tauri's `appDataDir`.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Database filename:** This plan names it `cartridge.db`. Do you confirm, or do you want a different name?

**Pass Criteria**
- [ ] App launches without a Rust panic or JS error related to the DB plugin
- [ ] The database file is created in the expected `appDataDir` on first launch
- [ ] A simple `SELECT 1` query from the frontend returns a result without error

---

### T1.8 — Write and run the initial database migration `[DB]`

Create a migration system using numbered SQL files. Write migration 001 creating all local tables from the schema in T0.3. Run migrations automatically on app startup.

**Pass Criteria**
- [ ] On fresh launch, all tables exist — verifiable by querying `sqlite_master`
- [ ] Launching the app a second time does not re-run the migration or throw an error
- [ ] All columns match the schema document exactly (verified by `PRAGMA table_info(games)`)

---

### T1.9 — Set up global state management with Zustand `[UI]`

Install `zustand`. Create the initial store structure with slices for library, UI, and session state.

**Pass Criteria**
- [ ] Store is accessible from any component without prop drilling
- [ ] State updates in one component are immediately reflected in another reading the same slice
- [ ] No TypeScript errors in the store definitions

---

### T1.10 — Configure environment variable handling for dev and prod builds `[INFRA]`

Configure Vite to load variables from `.env.local`. Verify no API keys are included in the compiled JS bundle.

**Pass Criteria**
- [ ] `import.meta.env.VITE_SUPABASE_URL` returns the correct value in dev mode
- [ ] Grepping the production build output for a known API key returns zero results
- [ ] Attempting to use a variable that doesn't exist returns `undefined`, not a build error

---

## Phase 2 — Core UI Shell

> The interface exists but has no real data. Fully navigable with keyboard and controller. Every visual element is pixel-perfect before any logic is wired.

---

### T2.1 — Build the animated background system with dynamic per-game color DNA `[UI]`

Create a `Background` component with animated blobs, a noise overlay, and a vignette. Accept a `colorDNA` prop and animate color transitions when the focused game changes.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Animation speed:** Should the background blobs animate slowly and subtly (~12–15 second cycles), or faster/slower?
> 2. **Blur intensity:** Should the blobs be very blurred and diffuse (~120px blur), or tighter and more defined?
> 3. Should the background also subtly show a **blurred version of the game's box art** as the backdrop (like Daijishō), or keep it abstract color-only?

**Pass Criteria**
- [ ] Background fills the full window with no white gaps
- [ ] The blobs visibly, slowly animate — not static
- [ ] Passing different `colorDNA` props produces visually different backgrounds
- [ ] Transitioning between two color sets animates smoothly
- [ ] No frame drops on the animation (verified in browser DevTools)

---

### T2.2 — Build the top navigation bar component `[UI]`

Create the `TopBar` component with logo, navigation tabs, user status pill, and live clock. Bar uses `backdrop-filter: blur` and must sit in the Tauri drag region except for interactive elements.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Nav tabs:** This plan uses: Library, Discover, Achievements, Settings. Do you want to keep these, rename any, or add/remove tabs?
> 2. Should the "Discover" tab be included in v1 (greyed out as "coming soon"), or hidden entirely until it's implemented?

**Pass Criteria**
- [ ] Clicking each nav tab visually activates it and deactivates the others
- [ ] Clock displays current time and updates every minute
- [ ] Bar can be used to drag the window
- [ ] Clicking nav buttons does NOT drag the window
- [ ] Bar looks correct at both 1024px and 1920px window widths

---

### T2.3 — Build the platform filter strip `[UI]`

Create the `PlatformStrip` component as a horizontal row of scrollable pill chips. Selected chip is persisted in the Zustand store.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Supported platforms:** This plan includes: All, NES, SNES, N64, GBA, DS, 3DS, PS1, PS2, PSP, Genesis, Saturn, GCN. Are there any platforms to add (e.g., Atari 2600, Dreamcast, Arcade/MAME, Wii, Game Gear) or remove?
> 2. Should platforms with **zero games** in the library be automatically hidden from the strip, or always shown?

**Pass Criteria**
- [ ] Only one chip can be selected at a time
- [ ] Selected chip state survives a React re-render (persisted in store)
- [ ] At 1024px width, all chips are accessible via horizontal scroll with no overflow clipping

---

### T2.4 — Build the coverflow carousel with mock data `[UI]` `[HARD]`

Create the `GameCarousel` component. Renders cards in a staggered coverflow layout. All position changes are animated via CSS `transition`. `currentIndex` lives in the Zustand store.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Center card size:** This plan uses 200×280px for the center card. Does this feel right, or do you want it larger or smaller?
> 2. **Cards visible:** This plan shows 7 cards at once (1 center + 2 near + 2 far + 2 edge). Do you want more or fewer visible?
> 3. **Card animation speed:** Should card transitions animate fast (~300ms) or slower and more cinematic (~550ms)?
> 4. Should non-center cards have a slight **perspective/tilt** effect (true coverflow style), or stay flat and just scale down?

**Pass Criteria**
- [ ] 7 cards visible simultaneously in the correct staggered layout
- [ ] Cards wrap around at both ends of the array — no blank positions
- [ ] Changing `currentIndex` animates all cards smoothly — no jump cuts
- [ ] Center card's focus ring is clearly visible and distinct
- [ ] At 1280px wide, edge cards are partially visible

---

### T2.5 — Build the game info panel (bottom metadata strip) `[UI]`

Create the `GameInfoPanel` component docked to the bottom-left. When the selected game changes, text fades up and the progress bar animates its width.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Metadata chips:** This plan shows: playtime, achievement progress (X/Y), and release year. Do you want different or additional chips (e.g., genre, publisher, last played date)?
> 2. Should the game title display in **all-caps**, title case, or exactly as stored?

**Pass Criteria**
- [ ] Info updates within animation duration when `currentIndex` changes
- [ ] Long game titles wrap gracefully — no overflow or unexpected truncation
- [ ] Progress bar width animates from old value to new — not a jump
- [ ] Panel is fully readable on top of any background color combination

---

### T2.6 — Build the controller hint strip `[UI]`

Create the `ControllerHints` component docked to the bottom-right. Hints change based on the current active view.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Button mapping:** This plan uses: D-pad = Browse, A = Launch, LB = Menu/Profile, RB = Details, Y = Toggle Favorite, Start = Search. Do you want any remapped?
> 2. Should controller hints show **Xbox-style** labels (A, B, LB, RB) or **PlayStation-style** (✕, ○, L1, R1), or auto-detect based on the connected controller?

**Pass Criteria**
- [ ] All hints are readable at normal viewing distance
- [ ] Button icons visually resemble their real controller counterparts
- [ ] Hint set changes when the active view changes

---

### T2.7 — Implement keyboard navigation for the carousel `[UI]`

Add a `useKeyboardNav` hook with a 200ms navigation lockout to prevent runaway scrolling on keyhold.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Keyboard shortcuts:** This plan uses: Arrow Left/Right = navigate, Enter = launch, `/` = open search, Escape = close panel/overlay, Tab = open profile panel. Do you want any changed?
> 2. Should `W`/`A`/`S`/`D` also work as navigation keys in addition to arrow keys?

**Pass Criteria**
- [ ] Pressing → cycles through all games and wraps at the end
- [ ] Holding → does not skip games — 200ms lockout is enforced
- [ ] Pressing Enter logs the selected game's title to the console
- [ ] Pressing Escape with no panel open does nothing (no error)

---

### T2.8 — Implement Gamepad API input handling `[UI]` `[HARD]`

Create a `useGamepad` hook using the browser's Gamepad API with a `requestAnimationFrame` polling loop. Dispatches the same actions as keyboard — does NOT duplicate logic.

> ⚠️ Test on real hardware. WebView2 gamepad support has quirks — verify before proceeding.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Gamepad button mapping:** This plan uses: D-pad or left stick = navigate, A/Cross = confirm, B/Circle = back, LB/L1 = open menu, RB/R1 = open details, Y/Triangle = favorite, Start = search. Do you want any remapped?
> 2. Should the **left analog stick** also navigate the carousel, or D-pad only?
> 3. Should there be an analog stick **deadzone threshold**? (Suggested: 0.5)

**Pass Criteria**
- [ ] Plugging in a USB controller and pressing D-pad right navigates the carousel
- [ ] Tested with at least: Xbox controller and a generic Xinput controller
- [ ] Disconnecting a controller mid-use throws no errors
- [ ] Both keyboard and controller work simultaneously without conflict

---

### T2.9 — Build the slide-in profile/social panel `[UI]`

Create the `SidePanel` component that slides in from a confirmed direction with a blurred backdrop. Panel has tabs.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Panel tabs:** This plan uses: Profile, Friends, Trophies. Do you want different tab names or a different set?
> 2. Should the panel slide in from the **left** or the **right**?
> 3. Should the panel be a **fixed width** or scale with the window size?

**Pass Criteria**
- [ ] Panel slides in and out smoothly with no jank at 60fps
- [ ] Clicking outside the panel (on the backdrop) closes it
- [ ] All tab placeholders render without errors
- [ ] While panel is open, carousel navigation input is disabled

---

### T2.10 — Build the social presence sidebar (friend avatar column) `[UI]`

Create the `SocialSidebar` component docked to the right edge. Hovering/focusing an avatar expands a glass tooltip.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Max avatars shown:** This plan shows up to 5 friend avatars. Do you want more or fewer?
> 2. **Priority:** When more friends are online than the cap allows, should the sidebar prioritize friends who are **currently in-game**, or the **most recently active**?
> 3. Should the sidebar be on the **right edge** (current plan), or elsewhere?

**Pass Criteria**
- [ ] Avatars visible at rest without obscuring game cards
- [ ] Hovering an avatar shows the tooltip with a smooth fade-in
- [ ] Tooltip does not overflow the right edge of the window
- [ ] Status dot colors match the confirmed status definitions

---

### T2.11 — Build the empty state screen (no games added) `[UI]`

Create an `EmptyState` component shown when the game library is empty. Must feel intentional and beautiful — not like an error.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Copy:** This plan uses the headline "Your library is empty" and CTA "Add Games". Do you want different wording?

**Pass Criteria**
- [ ] Empty state renders when game count is 0
- [ ] Carousel does NOT render in this state — no empty card slots
- [ ] Clicking the CTA fires a console log (wired to real logic in Phase 3)
- [ ] Looks polished — not like a placeholder or error state

---

## Phase 3 — Library Management

> Real games, real data. The user can point Cartridge at their ROM folders and see their actual library. No scraping yet — just raw file data.

---

### T3.1 — Build the Tauri command to scan a folder and return ROM files `[RUST]`

Write a Tauri command `scan_rom_folder(path: String) -> Vec<RomFile>` that recursively walks a directory, filters by known ROM extensions, and returns file metadata. Handle permission errors gracefully.

> 🤔 **Decisions Required — Ask the user:**
> 1. **ROM extensions:** This plan includes: `.sfc`, `.smc`, `.gba`, `.gbc`, `.gb`, `.nes`, `.md`, `.gen`, `.iso`, `.bin`, `.cue`, `.n64`, `.z64`. Are there any to add (e.g., `.a26` Atari, `.lnx` Lynx, `.gg` Game Gear, `.pce` PC Engine) or remove?
> 2. Should the scanner **follow symbolic links**, or only real directories?
> 3. Should **hidden folders** (prefixed with `.`) be skipped?

**Pass Criteria**
- [ ] Calling the command returns a non-empty array for a folder with known ROM files
- [ ] Files with non-ROM extensions are not included
- [ ] Pointing at a folder with no ROMs returns an empty array, not an error
- [ ] Pointing at a non-existent path returns a meaningful error message, not a crash

---

### T3.2 — Build the platform detection logic from file extension and filename `[RUST]`

Write a pure function `detect_platform(extension: &str, filename: &str) -> Option<Platform>` with unit tests for every platform mapping.

> 🤔 **Decisions Required — Ask the user:**
> 1. For ambiguous extensions like `.bin`, should detection also look at the **parent folder name** as a hint (e.g., a `.bin` inside a folder named "PS1" → PS1)?
> 2. Should undetectable files be **silently skipped**, or added to the library as "Unknown" platform?

**Pass Criteria**
- [ ] All Rust unit tests pass: `cargo test`
- [ ] A file named `sonic.md` detects as Genesis, not "Markdown"
- [ ] A `.bin` file inside a folder named "PS1" detects as PS1
- [ ] A truly ambiguous file is handled per the confirmed behavior above

---

### T3.3 — Build the "Add ROM Folder" UI flow `[UI]` `[RUST]`

Wire the CTA button to Tauri's native folder picker. Show a progress indicator while scanning. On completion, display a summary with Confirm and Cancel.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the user be able to add **multiple folders at once** (multi-select in the picker), or one folder at a time?
> 2. If a subfolder structure is detected (e.g., `/ROMs/SNES/`, `/ROMs/GBA/`), should Cartridge treat the root as one source, or list each subfolder separately?
> 3. **Summary copy:** This plan shows: "Found 47 games across 3 platforms — Add to Library?". Do you want different wording?

**Pass Criteria**
- [ ] Native OS folder picker opens on button click
- [ ] Canceling the picker returns to the empty state without error
- [ ] Selecting a folder with ROMs shows the correct count in the summary
- [ ] Progress indicator is visible during scanning of large folders (100+ files)

---

### T3.4 — Persist the game library to SQLite `[DB]`

On confirming the scan, write all detected games to the `games` table using `INSERT OR IGNORE` keyed on file path. Update the Zustand library store.

**Pass Criteria**
- [ ] After confirming, the carousel shows real game titles from the scanned folder
- [ ] Restarting the app reloads the library from SQLite — games persist across sessions
- [ ] Scanning the same folder twice does not double the game count
- [ ] A game's `path` column matches its actual file location on disk

---

### T3.5 — Wire the platform filter to the library query `[UI]` `[DB]`

When the user selects a platform chip, re-query the `games` table with a `WHERE platform = ?` filter. Carousel resets to index 0 on filter change.

**Pass Criteria**
- [ ] Selecting "SNES" shows only SNES games in the carousel
- [ ] Switching to "All" restores the full library
- [ ] Carousel index resets to 0 on filter change — no "index out of bounds" error
- [ ] Filtering to a platform with no games shows the contextual empty state

---

### T3.6 — Build the game search overlay `[UI]`

Opening the search overlay auto-focuses the search input. Results update on every keystroke. Selecting a result navigates the carousel and closes the overlay.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Search trigger:** This plan uses `/` on keyboard and Start on gamepad. Do you confirm, or want different triggers?
> 2. Should search match only on **title**, or also on platform, genre, and publisher once metadata is available?
> 3. Should the search overlay show **recent searches** when the input is empty?

**Pass Criteria**
- [ ] Overlay opens and the text input is focused immediately — no extra click required
- [ ] Typing 3+ characters returns matching results within 100ms
- [ ] Selecting a result navigates the carousel to that exact game
- [ ] Pressing Escape or B closes the overlay without selecting anything
- [ ] Searching for a nonexistent title shows "No results" — not a blank screen

---

### T3.7 — Build play session tracking `[DB]`

Write `startPlaySession` and `endPlaySession`. On startup, close any orphaned open sessions from previous crashes.

> 🤔 **Decisions Required — Ask the user:**
> 1. When an open session is found on startup (crash/force-kill), should Cartridge: (a) use the ROM file's last-modified timestamp as the end time, (b) use the save state file's timestamp, or (c) mark it as "interrupted" with no duration?

**Pass Criteria**
- [ ] After a session, `play_sessions` has a row with correct `game_id`, `started_at`, and `duration_seconds`
- [ ] Total playtime in the game info panel is the correct sum of all sessions
- [ ] Force-killing the app and reopening does not leave orphan open sessions

---

### T3.8 — Implement recently played sort and favorites `[DB]` `[UI]`

Add `last_played_at` and `is_favorite` columns. Default carousel sort: recently played first.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Favorites behavior:** Should favorited games be pinned to the **top of the main library** view, or only appear in a separate "Favorites" filter?
> 2. **Sort for never-played games:** Should they sort **alphabetically (A–Z)** or by **date added** to the library?

**Pass Criteria**
- [ ] After playing a game, it appears first in the carousel on next app launch
- [ ] Toggling favorite on a game adds a star to its card immediately
- [ ] Toggling favorite twice returns the game to its previous sort position

---

### T3.9 — Build the game details overlay `[UI]`

Pressing the Details button on the focused game opens a full-screen overlay with: large box art (placeholder for now), playtime stats, play session history, and achievement progress (placeholder). Has a Launch button and a back button.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the play session history show **individual sessions** (date + duration) or just a **cumulative total**?
> 2. Should this overlay be triggered by **RB** (as planned), or a different button/key?

**Pass Criteria**
- [ ] Overlay opens with smooth animation — not a jump cut
- [ ] Play session history shows real data from `play_sessions`
- [ ] Pressing B / Escape / back returns to the carousel at the same index
- [ ] Launch button in this view starts a play session correctly

---

## Phase 4 — Emulator Integration

> Games actually launch. This is the hardest phase technically. Do not rush it. Every platform must be tested individually on real ROMs.

---

### T4.1 — Bundle RetroArch portable into the app resources `[RUST]` `[HARD]`

Download the RetroArch portable Windows build and configure it to be copied to a writable `appDataDir` on first launch.

> ⚠️ Verify RetroArch portable license allows bundling. Credit RetroArch in the app's About screen.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should Cartridge **bundle RetroArch directly in the installer** (larger installer, ~50–80MB), or **download RetroArch on first launch** (smaller installer, requires internet on setup)?
> 2. Should the initial copy be done **silently**, or with a visible "Installing components…" screen?

**Pass Criteria**
- [ ] RetroArch folder exists in `appDataDir/retroarch/` after first launch
- [ ] On second launch, the copy step is skipped
- [ ] The bundled `retroarch.exe` opens successfully when launched directly
- [ ] Production build includes the RetroArch resources — test on a clean machine

---

### T4.2 — Build the RetroArch core downloader/manager `[RUST]` `[UI]`

Create a `CoreManager` that downloads RetroArch cores from the official buildbot on first use of each platform.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Preferred core per platform — confirm each one:**
>    - NES: `nestopia_libretro.dll` or `fceumm_libretro.dll`?
>    - SNES: `snes9x_libretro.dll` or `bsnes_libretro.dll` (more accurate, heavier)?
>    - N64: `mupen64plus_next_libretro.dll` or `parallel_n64_libretro.dll`?
>    - GBA: `mgba_libretro.dll` or `vba_next_libretro.dll`?
>    - GB/GBC: `gambatte_libretro.dll` or `sameboy_libretro.dll`?
>    - Genesis: `genesis_plus_gx_libretro.dll` or `picodrive_libretro.dll`?
>    - PS1: `beetle_psx_libretro.dll` or `swanstation_libretro.dll`?
>    - DS: `melonds_libretro.dll` or `desmume_libretro.dll`?
> 2. Should the user be able to **manually override the recommended core** per platform in Settings?

**Pass Criteria**
- [ ] Triggering a download for a missing core shows a visible progress indicator
- [ ] Core file appears in the correct directory after download
- [ ] Re-triggering for an already-downloaded core skips the download
- [ ] Network failure shows an error and leaves no corrupt partial file

---

### T4.3 — Build the game launch Tauri command `[RUST]` `[HARD]`

Write Tauri command `launch_game(game_id: i64) -> Result<u32, String>`. Must NOT wait for RetroArch to exit — returns immediately after spawning.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should saves be organized by **game ID** (`appDataDir/saves/[game_id]/`) or by **platform** (`appDataDir/saves/[platform]/`)?
> 2. Should launching a game that is **already running** block the second launch, or allow multiple instances?

**Pass Criteria**
- [ ] Calling the command on a known-good ROM opens RetroArch with the game running
- [ ] The function returns within 500ms
- [ ] Save files are written to the correct directory
- [ ] Tested and passing for: NES, SNES, GBA, Genesis, PS1 (minimum)

---

### T4.4 — Handle RetroArch process lifecycle `[RUST]`

After launching RetroArch, respond per the confirmed window behavior. Spawn a background thread that waits on the process handle and fires a Tauri event when RetroArch exits.

> 🤔 **Decisions Required — Ask the user:**
> 1. When a game launches, should Cartridge: (a) **minimize to taskbar**, (b) **hide completely** (tray only), or (c) **stay open** in the background?
> 2. If the user closes RetroArch from the in-game menu, should Cartridge **ask "Did you mean to quit?"** or just return silently?

**Pass Criteria**
- [ ] Cartridge window behaves exactly as confirmed above after RetroArch launches
- [ ] Closing RetroArch normally brings Cartridge back to the foreground
- [ ] Force-killing RetroArch still brings Cartridge back
- [ ] Play session duration is correctly recorded after normal close
- [ ] Play session is correctly closed after force-kill

---

### T4.5 — Build per-game RetroArch configuration override `[RUST]`

Build a system to write RetroArch `.opt` per-game override files containing at minimum the save directory override.

**Pass Criteria**
- [ ] A `.opt` file is created in the correct RetroArch config directory for each launched game
- [ ] The save directory in the override points to the correct game-specific path
- [ ] Launching the same game twice does not create a duplicate or corrupt override file

---

### T4.6 — Build the save state browser UI `[UI]` `[RUST]`

Build `list_save_states(game_id)` command and a UI component inside the game details overlay listing saves with timestamps.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should save states be shown as a **list** (current plan) or as a **grid with screenshot thumbnails** (if RetroArch generates them)?
> 2. Should Cartridge allow **deleting save states** from within the UI, or read-only?

**Pass Criteria**
- [ ] After creating a save state in RetroArch, it appears in the Cartridge save browser after returning
- [ ] Timestamps are displayed in a human-readable format (e.g., "2 hours ago")
- [ ] Selecting a save state and launching loads from that state — verified in game

---

### T4.7 — Build the first-launch "Set Up Your Controllers" flow `[UI]`

On first launch, show a setup screen that detects connected controllers and writes a RetroArch config.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the controller setup screen be **mandatory** (no skip) or **skippable**?
> 2. Should Cartridge auto-configure **any Xinput controller** it detects, or only known Xbox controllers?

**Pass Criteria**
- [ ] First launch shows the controller setup screen
- [ ] Second launch skips directly to the library
- [ ] An Xbox controller is detected and assigned — verified by launching a game
- [ ] Keyboard-only users can skip if confirmed as skippable

---

### T4.8 — Integration test: launch one game from each supported platform `[HARD]`

Manual QA task. Launch a real ROM from each confirmed platform and verify the full lifecycle.

**Pass Criteria**
- [ ] All confirmed platforms launch successfully on the first try
- [ ] Save states work on all platforms
- [ ] All platforms correctly record playtime
- [ ] No crashes, no orphaned processes after closing
- [ ] Bug list exists (even if empty) documenting what was tested

---

### T4.9 — Wire RetroAchievements hardcore mode toggle to the launch command `[RUST]`

Add RetroAchievements config entries to the RetroArch config and write the `cheevos_hardcore_mode_enable` setting.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should hardcore mode be **on or off by default** for new games?
> 2. Should hardcore mode be a **per-game setting** (as planned) or a **global toggle** in Settings?

**Pass Criteria**
- [ ] RetroArch config contains the `cheevos_*` entries
- [ ] The hardcore mode toggle in game details writes to the correct SQLite column
- [ ] Changing hardcore mode and relaunching generates an updated `.opt` override

---

## Phase 5 — Metadata & Box Art

> This is where Cartridge stops looking like a file manager and starts looking like what was designed.

---

### T5.1 — Implement ROM MD5/CRC hashing for game identification `[RUST]`

Write `hash_rom(path: String) -> RomHashes` computing MD5 and CRC32. Run in a background thread, batched on startup.

**Pass Criteria**
- [ ] Hashing a known ROM produces the correct MD5 (verified against a known hash database)
- [ ] UI remains responsive while hashing runs in the background
- [ ] After startup, all games have non-null hash values within 2 minutes (100-game library)

---

### T5.2 — Integrate the ScreenScraper API for metadata lookup `[API]`

Write a `scrapeGame(md5: string, platform: string)` function calling the ScreenScraper `jeuInfos` endpoint, with a rate-limited queue.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Art type preference:** Which art type should be prioritized? Options: box front (`box-2D`), box back (`box-2D-back`), screenshot (`ss`), fan art, title screen (`sstitle`).
> 2. Should Cartridge scrape **all unscraped games automatically** on startup, or only scrape a game **on-demand** when the user navigates to it?
> 3. If ScreenScraper returns no result, should Cartridge try a **fuzzy filename match** as a fallback, or immediately mark it as "not found"?

**Pass Criteria**
- [ ] A call with a valid MD5 for a known game returns correct title and year
- [ ] Queueing 10 scrape requests takes at least 11 seconds — rate limit is respected
- [ ] An unknown hash returns a "not found" result, not a crash
- [ ] An API auth error returns a meaningful error message

---

### T5.3 — Download and cache box art images locally `[RUST]`

Download box art URLs and save them locally. Never re-download art that already exists.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Image format and quality:** This plan saves art as JPEG at 80% quality. Do you confirm, or prefer WebP or a different quality level?
> 2. **Target image width:** This plan targets ~400px wide. Do you want larger (e.g., 600px) or smaller (e.g., 300px)?
> 3. Should Cartridge download a **second higher-resolution copy** for the details overlay, or upscale the same image?

**Pass Criteria**
- [ ] After scraping, art files exist in the correct directory
- [ ] Re-running the scraper does not re-download already-existing images
- [ ] File sizes are reasonable per the confirmed quality settings
- [ ] A network failure does not leave a 0-byte or corrupt file

---

### T5.4 — Display real box art in the carousel cards and background `[UI]`

Update carousel cards to display local art using the `tauri://localhost/[path]` asset protocol. Extract dominant colors from box art and feed them into the background system as `colorDNA`.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Color extraction method:** Should Cartridge extract the **most dominant** color, or the **most vibrant/saturated** color (tends to produce more interesting backgrounds)?
> 2. How many colors should be extracted for the `colorDNA` — 2, 3, or more?

**Pass Criteria**
- [ ] Scraped games display real box art in the carousel
- [ ] Non-scraped games show the gradient fallback — no broken image icons
- [ ] Background colors visibly reflect the box art palette of the focused game
- [ ] No art image is stretched or pixelated — correct `object-fit: cover`

---

### T5.5 — Build the scraper progress UI `[UI]`

Show a subtle, non-blocking progress indicator while scraping runs. Add a setting to pause/resume.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Progress indicator placement:** This plan puts it in the bottom-right corner. Do you confirm, or prefer a different location?

**Pass Criteria**
- [ ] Scraping indicator appears when active and disappears when done
- [ ] Games can be launched and navigated while scraping runs
- [ ] Pausing scraping stops API calls — verified by watching network traffic
- [ ] Count accurately reflects remaining un-scraped games

---

### T5.6 — Build the manual game info editor `[UI]` `[DB]`

In the game details overlay, add an "Edit" button. Saving writes to `games` with a `manually_edited: true` flag. Manually edited games are skipped by the auto-scraper.

> 🤔 **Decisions Required — Ask the user:**
> 1. If the user manually edits a game, should Cartridge **never auto-scrape it again** (current plan), or **ask before overwriting**?
> 2. Should the editor allow editing the **platform assignment** (in case auto-detection was wrong)?

**Pass Criteria**
- [ ] Editing a title and saving immediately reflects in the carousel info panel
- [ ] Setting a custom art image displays it in the carousel and extracts its color DNA
- [ ] A manually-edited game is not overwritten by a subsequent scrape run

---

### T5.7 — Visual QA pass: Cartridge should now look like the design mockup `[UI]`

Side-by-side comparison of the running app with the design. Fix all visual discrepancies.

**Pass Criteria**
- [ ] A person shown only the app describes it as "like a console UI"
- [ ] No elements are misaligned or using fallback system fonts
- [ ] Animations are smooth and feel intentional — not instant or laggy
- [ ] All glass/blur elements display correctly in WebView2

---

## Phase 6 — RetroAchievements Integration

---

### T6.1 — Build the RetroAchievements login UI and credential storage `[UI]` `[API]`

In Settings, add a "RetroAchievements" section. Store credentials in the OS secure store — never in SQLite as plaintext.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should RA login be surfaced during **onboarding** (Phase 10), in **Settings only**, or both?
> 2. If the user doesn't have a RetroAchievements account, should Cartridge show a link to create one?

**Pass Criteria**
- [ ] Valid credentials show the user's RA username and point total after login
- [ ] Invalid credentials show a clear error — not a crash or silent failure
- [ ] Credentials survive app restart (persisted in secure storage)
- [ ] Credentials are NOT readable in the SQLite database file

---

### T6.2 — Fetch achievement list per game from RetroAchievements API `[API]` `[DB]`

Call the RA `GetGameInfoAndUserProgress` endpoint. Cache the response in `achievements_cache`. Refresh when the user returns from playing.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the cache refresh **every time** the user returns from a session, or only if the session lasted longer than a minimum (e.g., 5 minutes)?
> 2. Should Cartridge also display **global unlock rates** per achievement (e.g., "Unlocked by 12% of players"), or just the user's personal unlock status?

**Pass Criteria**
- [ ] For a known RA-supported game, the correct achievement list is returned
- [ ] Already-unlocked achievements are correctly marked
- [ ] A game not on RetroAchievements returns an empty list — not an error
- [ ] Cache is used on second call — no redundant API request

---

### T6.3 — Build the achievement list UI in the game details overlay `[UI]`

Add an "Achievements" section with badge icon, title, description, and point value.

> 🤔 **Decisions Required — Ask the user:**
> 1. For **locked** achievements, should the title and description be **visible but desaturated** (current plan) or **hidden/replaced with "???"** until unlocked?
> 2. Should achievements be sorted: (a) unlocked first then locked, (b) by point value (highest first), or (c) in the default RA order?

**Pass Criteria**
- [ ] Achievement list renders correctly with real RA data
- [ ] Unlocked and locked achievements are visually distinct per the confirmed behavior
- [ ] Badge images load without broken image icons
- [ ] Point total matches the value shown on the RetroAchievements website

---

### T6.4 — Build the achievement unlock toast notification `[UI]`

When the user returns to Cartridge, show queued toast notifications for each newly unlocked achievement.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Toast display duration:** This plan shows each toast for 4 seconds. Do you want longer (e.g., 6s) or shorter?
> 2. If the user unlocked many achievements in one session (e.g., 10+), should Cartridge: (a) show all as individual toasts in sequence, (b) show the first 3 then a "+7 more" summary, or (c) show a single summary toast?

**Pass Criteria**
- [ ] After unlocking a new achievement and returning, the toast appears
- [ ] Multiple unlocks behave per the confirmed option above
- [ ] No toast appears for achievements already unlocked before the session
- [ ] Toast slides in from the bottom-right

---

### T6.5 — Build the global achievements screen `[UI]`

Build the Achievements view showing total points, mastery count, recent unlocks feed, and per-game breakdown. Reads from local cache — no live API call required.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the recent unlocks feed show the last **10**, **20**, or a different number of entries?
> 2. Should mastered games be **featured at the top** of the breakdown, or sorted the same as non-mastered?

**Pass Criteria**
- [ ] Total points match the sum of all unlocked achievement points in the cache
- [ ] Recent unlocks list is sorted chronologically (newest first)
- [ ] A game with all achievements unlocked shows a "Mastered" badge
- [ ] Screen loads within 200ms (reading from local cache, not network)

---

### T6.6 — Show RA progress on carousel cards `[UI]`

Update the completion progress bar to include RA achievement completion as an input.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Progress formula:** This plan uses 50% playtime-based + 50% achievement-based. Do you confirm, or prefer: 100% achievement-based only, 100% playtime-based only, or a different ratio?
> 2. For games with **no RA data**, should the progress bar show playtime-based progress, or be hidden entirely?

**Pass Criteria**
- [ ] A game with 50% of achievements unlocked reflects this per the confirmed formula
- [ ] A fully mastered game has a visible indicator on its carousel card
- [ ] Games with no RA data are handled per the confirmed behavior

---

### T6.7 — Add Hardcore Mode toggle and visual indicator `[UI]`

Show the hardcore mode toggle in game details. Show a badge on the card when enabled. Show a confirmation dialog on first enable.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Card badge style:** For hardcore mode, should the badge be: (a) a red/gold "HC" text badge, (b) a skull or flame icon, or (c) a colored border on the entire card?
> 2. Should the confirmation dialog also warn that **cheats are disabled** in hardcore mode?

**Pass Criteria**
- [ ] Enabling hardcore mode shows the confirmed indicator immediately
- [ ] Confirmation dialog appears on first enable
- [ ] Launching in hardcore mode correctly sets `cheevos_hardcore_mode_enable = true` in the config

---

## Phase 7 — Social Backend

> The backend infrastructure. Get the data model and security right here — it is extremely difficult to fix later.

---

### T7.1 — Create all Supabase tables from the schema document `[DB]` `[INFRA]`

Create all tables in Supabase from T0.3 and commit the SQL as a migration file.

**Pass Criteria**
- [ ] All tables exist in Supabase — verifiable in the Table Editor
- [ ] Migration SQL file is committed and reproducible on a fresh Supabase project
- [ ] All foreign key constraints are in place

---

### T7.2 — Configure Row Level Security (RLS) policies for all tables `[DB]` `[HARD]`

Enable RLS on every table and write access policies.

> ⚠️ Do not proceed past this task until security is verified. A mistake here is a production data leak.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Presence visibility:** Should any logged-in user see any other user's presence (current plan), or **friends only**?
> 2. **Profile visibility:** Should full profile stats be visible to: (a) anyone, (b) friends only, or (c) only the user themselves?
> 3. Should there be a **"private mode"** where a user can hide their presence from everyone?

**Pass Criteria**
- [ ] User A cannot read User B's messages — verified with a direct API call using User A's JWT
- [ ] Presence is visible per the confirmed policy above
- [ ] An unauthenticated request returns zero rows on all protected tables
- [ ] RLS policy SQL is committed to version control

---

### T7.3 — Implement user registration and authentication `[API]` `[INFRA]`

Use Supabase Auth. On successful registration, create a row in `profiles` via database trigger. Store the session in the OS secure store.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Auth methods:** This plan uses email + password only. Do you also want **social logins** (e.g., Google, Discord) from the start?
> 2. Should **email verification** be required before a new account can use social features?
> 3. Should users log in with **email or username**, or email only?

**Pass Criteria**
- [ ] Registering creates rows in both `auth.users` and `profiles`
- [ ] Logging out and restarting auto-logs back in with stored credentials
- [ ] Expired access tokens are silently refreshed — user is never logged out unexpectedly
- [ ] An incorrect password shows a clear error message

---

### T7.4 — Build the presence broadcasting system `[API]` `[DB]`

Upsert presence on launch, update on game launch/exit, update to offline on app close. Handle force-kill via a scheduled edge function.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Presence statuses:** This plan uses: `online`, `in-game`, `away`, `offline`. Do you want to add a **`do-not-disturb`** status that suppresses friend notifications?
> 2. **Auto-away:** Should the app automatically set the user to `away` after a period of idle (no input)? If so, how long? (Suggested: 10 minutes. Or disable entirely.)
> 3. The edge function sets users offline if `updated_at` is more than **3 minutes old**. Do you want a different threshold?

**Pass Criteria**
- [ ] Opening the app shows the account as "online" when queried from another device
- [ ] Launching a game shows the correct game name on another device within 5 seconds
- [ ] Closing the app normally shows "offline" immediately
- [ ] Force-killing shows "offline" within the confirmed cleanup window

---

### T7.5 — Build the friends system `[API]`

Write Supabase RPC functions for: send, accept, decline, and remove friend requests.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should there be a **block** feature in v1? (Blocked users can't send requests or messages)
> 2. Should there be a **cap** on the maximum number of friends per account? (Suggested: no cap for v1)

**Pass Criteria**
- [ ] Sending a friend request creates a pending friendship row
- [ ] Accepting creates an accepted row
- [ ] Sending a duplicate request returns an error — not a duplicate row
- [ ] After removal, neither user sees the other in their friends list

---

### T7.6 — Set up Supabase Realtime subscriptions `[API]` `[HARD]`

Enable Realtime on `presence`, `messages`, and `friendships`. Write a `useRealtimeSubscriptions` hook.

**Pass Criteria**
- [ ] User A launching a game is reflected in User B's friends list within 2 seconds — no manual refresh
- [ ] User B sending a message appears in User A's chat within 2 seconds
- [ ] Closing and reopening re-establishes all subscriptions within 3 seconds
- [ ] No duplicate messages or presence updates from repeated events

---

### T7.7 — Build the activity feed write logic `[API]` `[DB]`

Write Supabase database triggers that auto-write to `activity_feed`. Set up a scheduled cleanup function.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Activity types:** This plan generates events for: session ended, achievement unlocked, new friend accepted. Do you want additional types (e.g., game mastered, level up)?
> 2. **Feed retention:** This plan auto-deletes entries older than **30 days**. Do you prefer a different retention period?
> 3. Should users be able to **manually delete** individual feed entries?

**Pass Criteria**
- [ ] After a play session, an activity row appears in the feed with the correct payload
- [ ] Activity feed entries are only visible to friends (RLS enforced)
- [ ] The cleanup function exists as a Supabase scheduled function

---

### T7.8 — Build the direct messaging backend `[API]`

Write queries for `getMessageThread` and `sendMessage`. Implement content validation and "last read" tracking.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Message character limit:** This plan caps messages at 500 characters. Do you want a different limit?
> 2. Should messages support **basic markdown formatting** (bold, italic) in v1, or plain text only?
> 3. Should sent messages be **deletable** by the sender? If so, for sender only, or both parties?

**Pass Criteria**
- [ ] Message thread loads the last 50 messages in correct chronological order
- [ ] A message over the confirmed limit is rejected — not silently truncated
- [ ] Unread message count is calculable from the data

---

### T7.9 — Load test the social backend with simulated concurrent users `[INFRA]`

Simulate 50 concurrent users with active Realtime subscriptions. Measure latency, connections, and query times. Document results.

**Pass Criteria**
- [ ] 50 simulated concurrent users does not exceed Supabase free tier connection limits
- [ ] Average message latency under 500ms during the test
- [ ] Test results are documented as a file in the repo
- [ ] Scaling plan documented: at what user count does each Supabase tier become necessary?

---

## Phase 8 — Social UI

> The social features become visible. Every social element must feel native to the console UI — not like a chat app bolted onto an emulator.

---

### T8.1 — Build the authentication screens (login + register) `[UI]`

Build login and registration screens matching the Cartridge aesthetic — dark glass panels centered on the animated background.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the login screen show on **every launch** when not logged in, or only be accessible from Settings (app works fully without an account)?
> 2. Should there be a **"forgot password"** flow in v1, or just a link to reset externally?

**Pass Criteria**
- [ ] Screens match the glass aesthetic — not a generic white form
- [ ] Submitting empty fields shows per-field validation errors inline
- [ ] Successful login transitions to the library within 1 second
- [ ] Skipping login shows the library with social features visibly disabled (greyed out, not hidden)
- [ ] Controller-navigable: Tab key and D-pad move focus between form fields

---

### T8.2 — Wire the profile panel with real user data `[UI]`

The profile tab loads real data: avatar, display name, handle, level/XP, stats, and recent activity feed.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Level formula:** This plan uses every 500 RA points = 1 level. Do you confirm, or want a different rate?
> 2. **Level titles:** Do you want predefined tier titles (e.g., "Rookie", "Retro Fan", "Cartridge Veteran", "Legend"), or just a number (Level 12)?
> 3. Should the profile show a **RetroAchievements profile link** or any external link?

**Pass Criteria**
- [ ] Profile panel shows the logged-in user's real username and stats
- [ ] Activity feed shows the last 10 real activities
- [ ] Level calculation matches for a user with a known RA point total (manually verified)
- [ ] Restarting the app shows the same profile data

---

### T8.3 — Wire the friends list with real-time presence `[UI]`

The Friends tab loads the friends list grouped by status. Realtime subscription updates the list without a full reload.

**Pass Criteria**
- [ ] Friends list shows real friends with accurate statuses
- [ ] Friend launching a game updates their status within 3 seconds — no manual refresh
- [ ] Pending requests section appears only when there are pending requests
- [ ] Accepting a request moves the user to the friends list immediately

---

### T8.4 — Build the social presence sidebar with real data `[UI]`

The social sidebar shows real online friends, capped at the confirmed number. Clicking an avatar opens the side panel to that friend's chat.

**Pass Criteria**
- [ ] Real friend avatars appear with correct initials and status colors
- [ ] Clicking an avatar opens the side panel with that friend's chat focused
- [ ] If the user has no friends, the sidebar is hidden — not showing empty slots

---

### T8.5 — Build the in-app direct messaging UI `[UI]`

Build the chat thread view inside the side panel. Realtime subscription shows incoming messages instantly.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the message input support **emoji** via a picker, or just keyboard input?
> 2. Should there be a **typing indicator** ("m0ssfire is typing…") in v1?
> 3. Should there be **message reactions** (emoji reacts) in v1, or plain messages only?

**Pass Criteria**
- [ ] Sending a message appears in the thread immediately (optimistic update)
- [ ] Receiving a message while the thread is open shows it without a refresh
- [ ] Unread badge appears when a message arrives while a different thread is open
- [ ] Thread is scrollable — scrolling up loads the previous 50 messages
- [ ] Controller-compatible: D-pad can navigate between the message list and input

---

### T8.6 — Build the "Find Friends" search and add flow `[UI]`

In the Friends tab, add an "Add Friend" button that opens a debounced username search.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should users be searchable by **username only**, or also by display name?
> 2. Should there be a **"suggested friends"** section (e.g., users who play the same games) in v1, or just manual search?

**Pass Criteria**
- [ ] Searching a known username returns that user in results
- [ ] The current logged-in user does not appear in their own search results
- [ ] Already-friended users show a different state — not an Add button
- [ ] Sending a request to the same person twice shows "Request Sent" on second attempt

---

### T8.7 — Build the notification system `[UI]`

Build a notification bell in the top bar with an unread count badge and actionable dropdown.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Notification types:** This plan includes: new friend request, new message, friend unlocked an achievement in a shared game. Do you want any additional types (e.g., friend started playing a game you love)?
> 2. Should notifications also trigger an **OS-level Windows toast notification**, or in-app only?

**Pass Criteria**
- [ ] Receiving a friend request increments the bell badge
- [ ] Clicking a friend request notification and accepting works without opening the full panel
- [ ] After reading all notifications, badge disappears
- [ ] Notification state persists across app restarts

---

### T8.8 — Social UI integration QA pass `[HARD]`

Full end-to-end test of the social system using two real devices or accounts.

**Pass Criteria**
- [ ] Complete friend add flow works from Device A to Device B
- [ ] Both users can see each other's "Now Playing" status in real time
- [ ] Chat is functional in both directions
- [ ] A fresh user can find, add, and message a friend within 3 minutes without instructions
- [ ] Social elements match the visual design language of the rest of the app

---

## Phase 9 — In-Game Overlay *(Stretch Goal)*

> Treat this as a bonus phase. Ship without it if timeline is tight.

---

### T9.1 — Create a second Tauri window for the overlay `[RUST]`

Create a second named Tauri window (`overlay`) that is transparent, always-on-top, skip-taskbar, and not-focusable.

> ⚠️ This may not work in WebView2 with all fullscreen RetroArch configs. Test before building further overlay features.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Overlay position:** This plan defaults to the **top-right corner**. Do you prefer top-left, bottom-right, or user-configurable?
> 2. **Overlay default size:** This plan uses 300×400px. Do you confirm, or want different dimensions?

**Pass Criteria**
- [ ] Overlay window appears in the confirmed corner when a game is launched
- [ ] Overlay is transparent — no black or white background
- [ ] RetroArch remains in focus — clicking in RetroArch does not focus the overlay
- [ ] Overlay disappears when RetroArch is closed

---

### T9.2 — Build the overlay UI `[UI]`

Build the overlay React app (separate entry point from the main app). Collapsed by default, expands on hotkey press.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Overlay content:** This plan shows: Now Playing + up to 3 friend statuses + latest unread message. Is this right, or do you want to show/hide different things?
> 2. In its **collapsed state**, should the overlay be: (a) invisible until hotkey pressed, (b) a small always-visible icon, or (c) a thin info bar?

**Pass Criteria**
- [ ] Overlay shows correct friend statuses (reading from Supabase presence)
- [ ] Collapsed and expanded states both look intentional and polished
- [ ] Text is readable over both dark and light game backgrounds

---

### T9.3 — Implement global hotkey to toggle the overlay `[RUST]`

Register a global hotkey using Tauri's global shortcut plugin. Works even when RetroArch is focused. Allow rebinding in Settings.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Default hotkey:** This plan uses `Ctrl+Shift+C`. Do you confirm, or want a different default? (Must not conflict with common RetroArch shortcuts)

**Pass Criteria**
- [ ] Pressing the confirmed hotkey while RetroArch is in focus toggles the overlay
- [ ] Hotkey does not conflict with common RetroArch defaults
- [ ] Rebound hotkey persists across app restarts

---

### T9.4 — Show achievement unlock toast in the overlay `[UI]`

Poll the RA API during gameplay to detect newly unlocked achievements and show a toast in the overlay window.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Poll interval:** This plan polls every **30 seconds**. Do you want more frequent (15s) or less frequent (60s) polling?

**Pass Criteria**
- [ ] Unlocking an achievement triggers the overlay toast within the confirmed poll interval + ~5 seconds
- [ ] Toast appears without the user needing to toggle the overlay open
- [ ] Toast auto-dismisses after the duration confirmed in T6.4

---

### T9.5 — Overlay compatibility test across game types `[HARD]`

Test the overlay in: windowed, fullscreen exclusive, fullscreen borderless, and with different RetroArch graphic drivers.

**Pass Criteria**
- [ ] Overlay works correctly in at least borderless fullscreen mode
- [ ] Compatibility matrix is documented as a file in the repo
- [ ] Settings shows a note about required RetroArch configuration if needed

---

## Phase 10 — Polish & Ship

> The difference between a beta and a product. Do not skip this phase.

---

### T10.1 — Build the onboarding flow for new users `[UI]`

On first launch, walk the user through: add ROM folder → create account (or skip) → connect RetroAchievements (or skip).

> 🤔 **Decisions Required — Ask the user:**
> 1. Should each onboarding step have a **"Skip"** option (current plan), or should any steps be mandatory?
> 2. Should the onboarding flow come **before or after** the controller setup screen (T4.7)?
> 3. Are there any **additional steps** you want in the onboarding flow (e.g., choose accent color, set a display name)?

**Pass Criteria**
- [ ] A fresh install presents the onboarding flow before the library
- [ ] Skipping all steps leads to a functional app
- [ ] Completing onboarding with a ROM folder ends with games visible in the carousel
- [ ] Onboarding never shows again after completion

---

### T10.2 — Build the Settings screen `[UI]`

Build a full Settings screen accessible from the top nav.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Settings sections:** This plan includes: General, Appearance, Library, Emulator, Accounts, Overlay, About. Are there any to add or remove?
> 2. Under **Appearance**, should the user choose from **preset accent colors** or use a **full color picker**?
> 3. Should Settings be a **full-screen overlay** or a **slide-in panel**?

**Pass Criteria**
- [ ] Every changed setting persists — survives app restart
- [ ] Changing the accent color immediately updates all UI elements using it
- [ ] Removing a ROM folder removes those games from the DB and carousel
- [ ] Settings screen is fully controller-navigable

---

### T10.3 — Performance optimization: virtualize the game library for large collections `[UI]` `[HARD]`

Implement virtual scrolling on any list that could exceed 100 items using `react-virtual`. Target: 60fps on 1000+ games.

**Pass Criteria**
- [ ] Adding 1000 games and opening search results shows no frame drops
- [ ] Chrome DevTools Performance shows no "long tasks" during list scroll
- [ ] Memory usage does not grow continuously while scrolling

---

### T10.4 — Implement global error handling and user-facing error states `[UI]`

Add a React Error Boundary at root. All Tauri command calls must have try/catch with meaningful error messages.

> 🤔 **Decisions Required — Ask the user:**
> 1. For **fatal errors** (e.g., database corruption), should Cartridge: (a) show an error screen with a "Reset Library" button, (b) automatically attempt repair, or (c) open the app data folder so the user can fix it manually?

**Pass Criteria**
- [ ] Simulating a network failure during scraping shows a friendly message, not a crash
- [ ] Launching a game with a missing ROM shows "ROM file not found" — not a crash
- [ ] A Supabase outage keeps the app functional in offline mode — local library still works
- [ ] No raw error stack traces are ever shown to the user

---

### T10.5 — Implement crash reporting `[INFRA]`

Integrate a crash reporting service for both Rust and React. Add an opt-in prompt during onboarding.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Crash reporting service:** This plan uses Sentry (free tier). Do you confirm, or prefer Bugsnag, Datadog, or no third-party service (just local log files)?
> 2. Should crash reporting be **opt-in** (user must enable it) or **opt-out** (enabled by default, user can disable)?

**Pass Criteria**
- [ ] Intentionally triggering a JS error causes it to appear in the confirmed service's dashboard
- [ ] Users who opt out send no events
- [ ] File paths in error reports are anonymized

---

### T10.6 — Set up code signing and build the Windows installer `[INFRA]` `[HARD]`

Build an installer. Set up GitHub Actions to auto-build on every push to `main`.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Installer type:** Tauri supports MSI and NSIS (`.exe`). Do you want MSI (current plan), NSIS, or both?
> 2. Should the installer offer a **"Launch on Windows startup"** option, or never auto-start?
> 3. Do you have a **code signing certificate**, or should this beta be self-signed (users will see a SmartScreen warning)?

**Pass Criteria**
- [ ] Installer runs on a clean Windows 10/11 machine with no prior Cartridge installation
- [ ] App appears in "Add or Remove Programs" with correct name and version
- [ ] Uninstalling removes the app but NOT the user's library data in `appDataDir`
- [ ] GitHub Actions CI produces a downloadable build artifact on every commit to `main`

---

### T10.7 — Implement auto-updater `[INFRA]`

Configure Tauri's built-in updater pointing to a GitHub Releases endpoint. Silently check on startup. Show a notification banner if an update is available.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should updates **download and install automatically** in the background (then prompt to restart), or require the user to click "Download & Install"?
> 2. Should the app check for updates **every launch**, **once a day**, or **once a week**?

**Pass Criteria**
- [ ] Publishing a new GitHub release with a higher version triggers the update notification
- [ ] Update installs and app restarts to the new version
- [ ] Clicking "Later" dismisses the notification until the next launch
- [ ] Update check failure (no internet) is silent — no error shown

---

### T10.8 — Focus management and accessibility audit `[UI]`

Audit the full app for focus management. Every action possible with a mouse must also be possible with keyboard + controller.

**Pass Criteria**
- [ ] Navigate the entire app — library, search, settings, social, game launch — using only a controller, zero mouse
- [ ] At no point is focus lost or invisible
- [ ] Opening and closing every modal returns focus to the correct element

---

### T10.9 — External beta test with 5+ fresh users `[HARD]`

Give the installer to 5 people with no instructions. Observe or record them using it. Fix every issue that 2+ users encountered.

> 🤔 **Decisions Required — Ask the user:**
> 1. Do you have **5 people available** for the beta test? If not, how many, and should the pass criteria be adjusted?
> 2. Should beta testers be **retro gaming enthusiasts**, or general users with no emulation background?

**Pass Criteria**
- [ ] All 5 users successfully add a ROM folder and launch a game without help
- [ ] At least 4 of 5 users find the social features without being told where they are
- [ ] Every issue that 2+ users encountered has a corresponding fix commit
- [ ] Test session notes or recordings are saved for reference

---

### T10.10 — Write the public release README and documentation `[INFRA]`

Write a README and CHANGELOG for v0.1.0-beta.

> 🤔 **Decisions Required — Ask the user:**
> 1. Should the GitHub repository be **public from the start**, or private until the beta is ready to share?
> 2. Is there a **project website or landing page**, or is GitHub the only public presence for now?

**Pass Criteria**
- [ ] README renders correctly on GitHub with no broken image links
- [ ] A new user reading only the README can install and set up Cartridge without external help
- [ ] Legal/attribution section acknowledges all third-party software used

---

### T10.11 — Publish the v0.1.0-beta GitHub release `[INFRA]`

Tag the commit and let GitHub Actions attach the installer to the release.

> 🤔 **Decisions Required — Ask the user:**
> 1. **Version tag:** Do you confirm `v0.1.0-beta`, or prefer a different convention (e.g., `v0.1.0`, `0.1-beta`, `early-access-1`)?
> 2. **Where will you announce the release?** (e.g., Reddit r/emulation, Discord, Twitter/X) — links should be included in the release notes.

**Pass Criteria**
- [ ] GitHub release page exists with the installer attached and downloadable
- [ ] The installer attachment is the CI-built artifact — not a local build
- [ ] Release notes clearly mark it as beta and list known issues
- [ ] Downloading and installing from GitHub works on a machine that has never had Cartridge installed

---

*Cartridge — Build Task List — v0.1.0-beta scope*
