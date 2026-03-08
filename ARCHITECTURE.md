# Cartridge — Architecture Decision Record

> **Status:** Approved  
> **Date:** 2026-03-08  
> **Author:** Project owner  
> **Rule:** Every technology choice below is locked for v1. Deviation requires updating this document first.

---

## 1. Purpose

Cartridge is a desktop game-library frontend and launcher. It scans local ROM collections, fetches rich metadata and box art from online sources, launches best-in-class emulators, tracks play time and achievements, and exposes social features (friends, presence, activity feed) via a cloud backend — all inside one polished, animated native window.

---

## 2. Technology Stack

### 2.1 Desktop Framework — **Tauri v2**

| Attribute | Detail |
|---|---|
| Language | Rust (backend) + TypeScript (frontend) |
| Renderer | WebView2 (Windows) · WebKitGTK (Linux) |
| Bundle size | ~10 MB (vs ~150 MB Electron) |

**Rationale:** Tauri v2 provides a Rust core for high-performance filesystem scanning, process management (launching emulators), and OS integration, while its WebView layer enables the fully custom, animation-rich UI demanded by the design. Electron was rejected because of its bundled Chromium overhead; Qt and native frameworks were rejected because they cannot match the CSS/animation design fidelity required. No lighter or more suitable alternative exists for this combination — Tauri v2 is confirmed.

### 2.2 UI Framework — **React 18 + TypeScript**

**Rationale:** React 18 has the broadest ecosystem of animation (Framer Motion, GSAP), virtualized-list, and UI component libraries needed for the coverflow carousel, animated backgrounds, and GPU-composited transitions. TypeScript strict mode catches interface mismatches across Rust ↔ frontend boundaries at compile time. Vue and Svelte were considered but lack equivalent animation-library depth and were rejected.

### 2.3 Styling — **Tailwind CSS v3 + Framer Motion**

**Rationale:** Tailwind provides design-token-driven utility classes that keep component styles co-located and consistent; Framer Motion drives all orchestrated enter/exit animations, layout transitions, and the coverflow physics. This combination enables the "sophisticated, stylish" aesthetic requirement without a bespoke CSS architecture.

### 2.4 Build Tool — **Vite**

**Rationale:** Vite is the default bundler for Tauri React projects and provides sub-second hot-reload during development. No alternative is needed.

### 2.5 State Management — **Zustand**

**Rationale:** Zustand is minimal, TypeScript-first, and supports slice-based organization without the boilerplate of Redux. It handles the three global slices: library state, UI state (active platform, focused game, nav tab), and session state.

### 2.6 Local Database — **SQLite via tauri-plugin-sql**

**Rationale:** All library data (games, platforms, play sessions, achievements cache, save states) lives in a local SQLite file inside Tauri's `appDataDir`. This works fully offline, has zero latency for reads, and is portable. The database file is named `cartridge.db`.

### 2.7 Cloud Backend — **Supabase**

**Rationale:** Supabase provides Postgres, real-time subscriptions (for friend presence and activity feed), authentication (magic link + OAuth), and storage (avatar uploads) as a single managed service with a generous free tier. Firebase was rejected due to its NoSQL model being poorly suited to relational social data; self-hosted options were rejected to minimize operational burden in v1.

---

## 3. Target Platforms

| Platform | v1 Support |
|---|---|
| Windows 10/11 (x86-64) | ✅ Primary |
| Linux (x86-64, glibc-based distros) | ✅ Primary |
| macOS | ❌ Out of scope for v1 |
| ARM / mobile | ❌ Out of scope for v1 |

Both Windows and Linux are first-class citizens from day one. Platform-specific CI jobs will gate releases for both targets.

---

## 4. Emulator Strategy

### 4.1 Philosophy

Cartridge bundles and manages **best-in-class standalone emulators** for each supported platform. "Best-in-class" means highest accuracy and active maintenance as of the date of this document. The user may switch between available emulators per-platform if they prefer a different option — Cartridge stores this preference in `cartridge.db` and passes it through when launching a game.

RetroArch is **not** used as the primary emulator layer. Individual standalone emulators are preferred because they have more direct update paths, cleaner CLI interfaces, and better per-platform accuracy records. RetroArch cores may be offered as optional alternatives for platforms where no suitable standalone emulator exists.

### 4.2 Emulator Table

| Platform | Primary Emulator | Alternate (user-selectable) |
|---|---|---|
| NES | Mesen | — |
| SNES | bsnes (accuracy) | Snes9x (compatibility) |
| N64 | Ares | simple64 |
| Game Boy / GBC | mGBA | — |
| Game Boy Advance | mGBA | — |
| Nintendo DS | melonDS | — |
| Nintendo 3DS | Lime3DS | — |
| Nintendo GameCube / Wii | Dolphin | — |
| Wii U | Cemu | — |
| Nintendo Switch | **Eden** | — |
| Sega Master System / Game Gear | Emulicious | — |
| Sega Genesis / Mega Drive | BlastEm (accuracy) | Gens+ReRecording |
| Sega Saturn | Kronos | Mednafen/Beetle Saturn |
| Sega Dreamcast | Flycast | — |
| PlayStation 1 | DuckStation | — |
| PlayStation 2 | PCSX2 (nightly) | — |
| PSP | PPSSPP | — |
| PlayStation 3 | RPCS3 | — |
| Xbox 360 | Xenia Canary | — |
| Atari 2600 | Stella | — |
| Neo Geo / Arcade | FinalBurn Neo | MAME |
| PC Engine / TurboGrafx-16 | Mednafen | — |

> **Note on Eden:** Eden is the community-maintained continuation of Yuzu and is the designated Switch emulator for Cartridge. No other Switch emulator is documented or supported.

### 4.3 Emulator Management

- Emulators are **not** bundled inside the Cartridge installer. On first use of a platform, Cartridge guides the user to download or locate the emulator executable.
- Cartridge stores the resolved path in `cartridge.db` and validates it on each launch.
- Launch parameters are defined per-emulator in a versioned JSON config shipped with the app.

---

## 5. Metadata Sources

Game metadata (title, description, release year, developer, publisher, genre, player count, box art, screenshots, videos) is sourced from multiple providers in priority order:

| Priority | Provider | Used For |
|---|---|---|
| 1 (Primary) | **ScreenScraper** | Box art, media, full game details |
| 2 (Fallback) | **IGDB** | Descriptions, release data, cover art |
| 3 (Fallback) | **TheGamesDB** | Title matching, supplemental art |
| 4 (Fallback) | **MobyGames** | Older/obscure titles, screenshots |

**Rationale:** ScreenScraper has the deepest ROM-hash-matched database for retro platforms. The others are used when a title is not found in ScreenScraper or when ScreenScraper rate-limits the request. Chaining sources maximises coverage across all platforms listed in §4.2.

All fetched metadata is cached in `cartridge.db` and in a local media directory inside `appDataDir` so that API calls are only ever made once per game unless the user triggers a refresh.

---

## 6. Explicit Out-of-Scope for v1

These features will **not** ship in v1 and must not be built as part of Phase 0–8 tasks:

| Feature | Reason deferred |
|---|---|
| macOS support | Development capacity; requires separate CI pipeline and code-signing |
| In-game overlay (Phase 9) | Complex OS-level injection; post-launch stretch goal |
| Netplay / online multiplayer | Depends on per-emulator netplay APIs; future phase |
| ROM downloading or any piracy tooling | Out of scope permanently |
| Cloud save sync | Cross-platform save-state format differences; post-v1 |
| Mobile companion app | Separate codebase; post-v1 |
| Store / purchase integration | Out of scope permanently |
| ARM / Apple Silicon native builds | Post-v1 when ARM Linux gaming is mainstream |

---

## 7. Repository & Tooling Conventions

| Concern | Choice |
|---|---|
| Language | TypeScript (strict) + Rust (stable) |
| Linter | ESLint (TypeScript rules) |
| Formatter | Prettier (single quotes, no semicolons, 2-space indent) |
| Pre-commit | Husky + lint-staged |
| Package manager | npm |
| Git branching | `main` is always releasable; feature branches off `main` |
| Environment secrets | `.env.local` (gitignored) + `.env.example` (committed) |

---

## 8. Decision Log

| # | Decision | Decided By | Date |
|---|---|---|---|
| 1 | Desktop framework = Tauri v2 | Owner | 2026-03-08 |
| 2 | UI = React 18 + TypeScript | Owner | 2026-03-08 |
| 3 | Backend = Supabase | Owner | 2026-03-08 |
| 4 | v1 targets Windows + Linux | Owner | 2026-03-08 |
| 5 | Switch emulator = Eden | Owner | 2026-03-08 |
| 6 | Best-in-class standalone emulators per platform | Owner | 2026-03-08 |
| 7 | Primary metadata = ScreenScraper; fallbacks = IGDB, TheGamesDB, MobyGames | Owner | 2026-03-08 |
