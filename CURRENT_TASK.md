# Current Task: Settings Cleanup and Simplification

**STATUS: COMPLETED**

## Current Problem
The Settings menu in the Timeline engine was bloated with technical and unused options (e.g. Wiki, Spreadsheet, Map views, Disable Groups, Keep Selection, Hide Decimals) which cluttered the UI and confused the user. The goal was to remove all non-essential toggles, hardcode their behaviors to the optimal states, and flatten the settings menu.

## Goal
Streamline `SettingsModal.jsx` by removing redundant toggles, hardcoding their payload properties to preferred defaults, and merging all remaining settings into a single, tab-less panel for better UX.

## Current State
- [x] Removed Wiki, Spreadsheet, Maps toggles (locked to off).
- [x] Removed Branch Ordering (locked to later-first).
- [x] Removed Connect Event Lines (locked to off/full-length).
- [x] Removed Hide Decimals (locked to true).
- [x] Removed Disable Groups (locked to false).
- [x] Removed Keep Selection (locked to false).
- [x] Removed Show Popular Tags (locked to true).
- [x] Deleted sidebar navigation and merged `general` and `appearance` into a single view.
- [x] Verified successful build after all removals.

## Exact Next Step
- Phase 1 (Settings Cleanup) complete! Next step is the visual redesign of the timeline.

## Things that must not be repeated
- Do not expose highly technical rendering configuration toggles to standard users unless explicitly requested.
