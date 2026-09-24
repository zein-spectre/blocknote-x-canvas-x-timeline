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
