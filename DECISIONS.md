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

## 4. Hardcoding Legacy Settings
**Date:** 2026-09-22
**Context:** The legacy timeline engine contained numerous highly technical rendering options in `SettingsModal.jsx` (e.g. Wiki views, Disable Groups, Hide Decimals) that cluttered the UI.
**Decision:** We aggressively pruned the UI and hardcoded these settings to their logical "ideal" defaults within the payload before updating the `timelineRef`.
**Consequences:** 
- Drastically simplifies the settings menu to only 5 essential toggles.
- Prevents users from accidentally placing the timeline into an unreadable state.

## 5. Canvas to Article Page Relational Flow & Searchability
**Date:** 2026-09-24
**Context:** We implemented custom React `embeddable` components in Excalidraw to represent "Article Pages". However, Excalidraw's built-in "Find on canvas" search could not detect these custom components because their text existed in React state, not in Excalidraw's `type: "text"` element JSON data.
**Decision:** We entirely dropped the custom "Article Page" `embeddable` button and forced users to rely strictly on typing `@` inside standard Excalidraw text elements to link to articles. The linked text (e.g. `📝 Artikel 2`) is natively saved as a string inside Excalidraw's elements. We also aggressively hid the Library, Social Links, and Export buttons via strict CSS and `UIOptions` to enforce this streamlined UX.
**Consequences:** 
- Excalidraw's native "Find on canvas" works flawlessly and finds all referenced articles on the canvas.
- Reduces performance overhead by not rendering heavy IFrames/BlockNote wrappers for every note representation on the canvas.
- Keeps UX highly intuitive, modeling the standard Notion/Affine behavior.

## 6. Mention Navigation — Always Open in New Tab
**Date:** 2026-09-24
**Context:** Mention chips (`📝 Canvas 1`) inside the BlockNote editor were originally navigating within the same React Router tab. This caused two problems: (1) unsaved changes on the originating page (Timeline, Canvas) would be discarded silently when React unmounted the page, and (2) clicking a mention from a Preview page would incorrectly route to an Admin page.
**Decision:** All mention clicks across ALL surfaces (Admin editor, Preview/PublicReader, Canvas Admin, Canvas Preview) MUST open the target document in a new browser tab via `window.open(..., '_blank', 'noopener,noreferrer')`. React Router `navigate()` is forbidden for cross-document mention navigation.
**Consequences:**
- No data-loss risk from accidental mention clicks.
- Preview pages correctly open Preview URLs (`/view/...`) in the new tab.
- Canvas `handleOpenAppwriteNote` now uses `Promise.any()` to detect doc type before opening the correct URL.

## 7. Excalidraw Canvas Always Renders in Light Theme
**Date:** 2026-09-24
**Context:** Excalidraw's built-in Dark mode toggle was available in the hamburger menu. This creates an inconsistent UX where the canvas theme is user-controlled but the rest of the CMS (Article, Timeline) is not.
**Decision:** Disable the Dark mode toggle in Excalidraw by setting `UIOptions.canvasActions.toggleTheme: false` in both `CanvasPrototypePage.tsx` and `CanvasViewPage.tsx`. The canvas will always render in the light theme.
**Consequences:**
- Consistent visual experience across all pages of the CMS.
- Users cannot accidentally switch to dark mode on the canvas.

