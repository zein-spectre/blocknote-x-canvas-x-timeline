# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased] - 2026-09-22

### Added
- Integrated the legacy `Timeline-project-lama` engine into the modern BlockNote CMS infrastructure.
- Added `timeline_notes` collection to Appwrite to persist note data independently for timeline elements.
- Implemented `BlockNoteWrapper.jsx` to adapt the legacy `NoteEditor` interface to use the standard `<Editor />` component.

### Changed
- Replaced the experimental grid-based canvas timeline with the mature infinite-canvas implementation from the Electron app.
- Shimmed `window.electron` calls inside the timeline engine to use `appwriteApi.js`, enabling web-based persistence.
- Cleaned up `SettingsModal.jsx` by removing redundant technical toggles (Wiki, Spreadsheet, Maps, Hide Decimals, Disable Groups, Keep Selection, Popular Tags, Branch Ordering, Use Calendar, Connect Event Lines) and hardcoding their behavior to logical defaults.
- Merged the remaining settings in `SettingsModal.jsx` into a single, scrollable panel without a sidebar.
- Shifted Canvas-to-Article Page relational flow to rely purely on `@` mention system (client-side filtered via `SlashMenu`) instead of rendering complex React `embeddable` frames. This ensures Excalidraw's native "Find on canvas" engine can accurately search article titles.

### Fixed
- Fixed missing timeline axis issue where events were floating loosely on the page.
- Fixed UI contrast bug by correctly copying and importing the `themes/` directory, restoring the `parchment_v2` styling.
- Resolved Excalidraw Canvas searchability limitations by deprecating the `embeddable` custom Article Page component and stripping non-essential Excalidraw UI (e.g., Export, Library, Social Links) via strict CSS injection and `UIOptions`.

## [Finalisasi Pertama] - 2026-09-24

### Added
- All mention clicks now open target documents in a new browser tab (`window.open` with `target="_blank"`) across all surfaces: Admin editor, Article Preview, Canvas Admin, Canvas Preview. Eliminates data-loss risk from same-tab navigation.

### Changed
- `CanvasViewPage.tsx` — `handleOpenAppwriteNote` rewritten to detect document type via `Promise.any()` and open the correct `/view/...` preview URL in a new tab instead of loading content into a side panel.
- `CanvasPrototypePage.tsx` — `handleOpenAppwriteNote` rewritten similarly; opens Admin-mode URLs (`/admin/edit/`, `/canvas/`, `/timeline/`) in a new tab.
- `AdminEditorPage.tsx` — `onOpenNote` handler changed from `navigate()` to `window.open(_blank)`.
- `PublicReader.tsx` — `onOpenNote` handler changed from `navigate()` to `window.open(_blank)`.
- `CanvasPrototypePage.tsx` and `CanvasViewPage.tsx` — added `toggleTheme: false` to `UIOptions.canvasActions` to permanently remove Excalidraw's Dark mode toggle from the hamburger menu.

### Fixed
- **Timeline Preview blank screen** (`/view/timeline/:id`): Container height was `100%` resolving to `0px` under `min-height` parent. Fixed by changing to `height: "calc(100vh - 65px)"` in `TimelineViewPage.tsx`.
- **Timeline empty data crash** (`TimelineView.jsx`): When `allYears` array is empty, `Math.min/max` of spread empty array returns `±Infinity`, causing `NaN` in all pixel calculations and a silent blank render. Fixed with explicit fallback `rawMin = 0`, `rawMax = 100`.
- **Span/Era invisible in Preview but visible in Admin** (`TimelineView.jsx`, `timelineUtils.js`): `file.start`/`file.end` stored as `""` (empty string) in DB when unset. The `!= null` guard passed for empty string, treating `""` as year `0` and clamping pre-year-0 elements to width=0. Fixed by changing all guards to `!= null && !== ""`.

