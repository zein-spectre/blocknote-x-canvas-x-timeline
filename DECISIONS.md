# Architectural Decisions

## 1. Timeline Engine Re-Integration
**Date:** 2026-09-22
**Context:** We attempted to build a new canvas-based timeline from scratch within the BlockNote CMS wrapper. This experiment resulted in an unpolished UX (floating elements, missing axis lines, poor zooming).
**Decision:** We discarded the experimental rewrite and instead mounted the mature, proven `Timeline-project-lama` (Electron-based) source code directly into the new web project as a monolithic module (`src/timeline-engine/`).
**Consequences:** 
- Drastically reduced development time for achieving feature parity.
- Requires maintaining a "shim" (`appwriteApi.js`) to catch and redirect legacy I/O calls intended for Electron.

## 2. Appwrite Persistence for Timeline I/O
**Date:** 2026-09-22
**Context:** The legacy timeline saved its state as `.json` files via Electron APIs.
**Decision:** We use Appwrite's Web SDK to simulate these filesystem actions. `loadTimeline`, `listTimelines`, and `saveTimelineToFile` were mapped to `getDocument`, `listDocuments`, and `updateDocument` respectively inside `appwriteApi.js`.

## 3. BlockNote Integration inside Legacy Code
**Date:** 2026-09-22
**Context:** The legacy project used its own rich-text component (`NoteEditor`). We needed it to match the main CMS experience.
**Decision:** Instead of ripping out the right-panel logic, we built `BlockNoteWrapper.jsx` to adapt the legacy `onSave` / `initialContent` prop signatures to the standard BlockNote `<Editor />` component.
