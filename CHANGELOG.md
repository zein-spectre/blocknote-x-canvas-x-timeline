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

### Fixed
- Fixed missing timeline axis issue where events were floating loosely on the page.
- Fixed UI contrast bug by correctly copying and importing the `themes/` directory, restoring the `parchment_v2` styling.
