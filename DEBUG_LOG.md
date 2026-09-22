# Debug Log

*Persistent debugging history for tracking failed attempts, evidence, eliminated causes, successful fixes, root causes, and verification.*

## Legacy Timeline Engine Integration
**Date**: 2026-09-22

**Issue**: 
The newly developed canvas-based timeline experiment lacked essential features (axis grid, proper absolute positioning) and diverged significantly from the original `Timeline-project-lama` UX. Furthermore, attempting to reproduce it from scratch caused frustration.

**Evidence/Failed Approaches**:
- Attempting to rebuild the timeline with a custom CSS/grid approach resulted in elements floating on an empty canvas without an axis.
- User explicitly requested to clone and adapt the previous project rather than building a new visual layer.

**Root Cause**:
Building a complex infinite-canvas UI with snap-to-grid timeline axis logic from scratch is heavily time-consuming and error-prone. The legacy Electron app already solved this, but it relied on native File I/O (`window.electron`) and a different rich-text editor (`NoteEditor`).

**Final Fix**:
1. Ported the entire `Timeline-project-lama` core into `src/timeline-engine/`.
2. Built a shim (`appwriteApi.js`) implementing `loadTimeline`, `saveTimelineToFile`, and `listTimelines` using Appwrite's Web SDK.
3. Used a `BlockNoteWrapper.jsx` to map the legacy `NoteEditor` interface directly into the BlockNote `<Editor />` to maintain CMS consistency.
4. Set up the `timeline_notes` Appwrite collection.
5. Copied missing dependencies (`mp4-muxer`) and config (`theme.json`, `themes/`).

**Why the fix works**:
The legacy application was already mature. By cleanly swapping out the persistence layer (I/O) with an API wrapper, the internal logic of the app (zoom, pan, positioning, theming) works identically on the web.

**Verification Performed**:
- Simulated a browser session using `browser_subagent`.
- Confirmed the page loaded at `http://localhost:5175/timeline/...` without console errors.
- Verified that the `parchment_v2` theme CSS variables correctly applied, rendering the left panel and canvas distinctively.

**Important Lessons**:
- When migrating legacy code, verify that all static assets and configuration files (like `themes/` and `elementIcons.js`) are copied along with components.
- Always shim external side-effects (like I/O) at the boundary layer so the core UI logic can remain untouched.

## Phase 1: Settings Cleanup
**Date**: 2026-09-22

**Issue**: 
The Timeline settings menu was bloated with technical toggles that were confusing to a standard user. It required cleaning up and hardcoding the optimal behavior for these removed settings.

**Root Cause**: 
The legacy engine was built with a wide array of Markwhen rendering toggles (e.g., Disable Groups, Hide Decimals) which are rarely useful and clutter the UX.

**Final Fix**:
1. Removed `disableGroups`, `keepSelection`, `showPopularTags`, `hideDecimals`, `eventLinesToGroupBottom`, `useWiki`, `useSpreadsheet`, `useMaps`, `branchOrdering`, `useCalendar`, and `hideSpanConnectors` from UI.
2. Hardcoded their boolean values to their logical optimal states in `timelineData.file` and the payload for `onUpdateTimelineRef`.
3. Deleted the sidebar component and merged the `general` and `appearance` settings into one simplified panel.

**Why the fix works**:
The underlying engine accepts these toggles in the payload. By hardcoding them to true/false respectively before sending the payload, we enforce the ideal UI configuration without needing the user to configure it themselves.

**Verification Performed**:
- Simulated frontend build via `npm run build` after removing each set of properties and verified that the build output succeeds with code 0.
- Assured all dependencies of the removed hooks were cleaned up (no stray variables in dependencies array).

**Important Lessons**:
- When porting a tool built for power users to a simplified UI, audit settings and aggressively hardcode the "ideal" path to reduce cognitive load.
