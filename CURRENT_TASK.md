# Current Task: Integrate Legacy Timeline Engine

**STATUS: COMPLETED**

## Current Problem
The previous timeline canvas implementation was buggy, lost data, and didn't behave like the expected timeline app. The user requested to discard the new experiment and directly integrate the proven `Timeline-project-lama` (Electron-based) codebase, while swapping out local filesystem calls with Appwrite DB calls, and replacing the NoteEditor with BlockNote.

## Goal
Integrate the legacy Timeline app as an embedded engine within the `TimelinePage` and connect it to Appwrite and BlockNote.

## Current State
- [x] Migrate `Timeline-project-lama` core logic to `src/timeline-engine`.
- [x] Build `appwriteApi.js` shim to redirect all I/O calls to Appwrite.
- [x] Replace `NoteEditor.jsx` with `BlockNoteWrapper.jsx` to utilize the new BlockNote CMS standard.
- [x] Setup `timeline_notes` collection in Appwrite.
- [x] Patch all `window.electron.*` calls across components (`TimelineApp.jsx`, `HomePage.jsx`, `useNoteManagement.js`).
- [x] Fix missing `theme.json` and `mp4-muxer` dependencies.
- [x] Mount the engine in `TimelinePage.tsx`.
- [x] Verified successful load via browser subagent (clean console, UI rendered).

## Exact Next Step
- Phase complete! The initial integration for the timeline formatted notes has been resolved successfully.

## Things that must not be repeated
- Do not build from scratch when the user asks to integrate a previous codebase.
- Do not assume `window.electron` exists in a web build.
