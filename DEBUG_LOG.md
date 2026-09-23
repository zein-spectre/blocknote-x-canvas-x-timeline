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

## Fix for Sequential Timeline Toggle Not Applying
**Date:** 2026-09-23
**Problem:** Toggling "Sequential Timeline" ON in the UI did not compress the timeline to only show events (years like 1921, 1922 still rendered).
**Root Cause:** `TimelineApp.jsx` contained a `handleUpdateTimeline` function that manually destructured all allowed properties from the `SettingsModal` updates. `useSequentialScale` was missing from the destructuring list and `nextFile` construction, causing the setting to be silently dropped before being saved to Appwrite.
**Fix:** Added `useSequentialScale` to the destructuring arguments and `nextFile` object in `TimelineApp.jsx`. Additionally, added `useCalendar` to the auto-save `useEffect` dependency array in `SettingsModal.jsx` to ensure changes to the Calendar mode immediately auto-save.
**Verification:** Subagent confirmed via UI screenshot that with Sequential Timeline ON, only 1920 and 1936 are rendered on the axis.

## Fix: 1926 Not Appearing as Tick Label in Sequential Timeline Mode
**Date:** 2026-09-23
**Problem:** With Sequential Timeline ON and an event at 09/20/1926, the tick "1926" did not appear as a labeled tick on the axis.
**Root Cause (Multi-layer):**
1. Event dates are stored as fractional years (e.g. `1926.7191` for Sept 20, 1926). The `uniqueYears` Set was storing the raw fractional value instead of the integer year, so `1926.7191` never matched an integer tick candidate.
2. The tick label proximity filter (`showLabel = labelLeft >= lastLabelRight + MIN_LABEL_GAP`) was suppressing the 1926 label because it was too close to 1920 and 1936 at the default zoom level.
3. The tick proximity guard (`if (px < lastTickPx + tickGapForYear(...))`) was also skipping ticks too close together.
**Fix:**
- Applied `Math.floor()` to all dates/times before adding to `yearSet` so only integer years are used.
- In Sequential mode, bypassed the tick proximity filter so all sequential ticks always render.
- In Sequential mode, bypassed the label proximity filter so all sequential labels always show.
**Verification:** Browser confirmed 1920 M, 1926 M, and 1936 M all appear as labeled ticks on the axis.

## Session: BlockNote Fragmentation & Save Button Debugging
**Date:** 2026-09-23

### Problem 1: "Simpan Perubahan" Button Not Visible

**Issue:** User reported the Save button at the bottom of the right panel in Edit mode was not visible or reachable. Could not scroll to it.

**Failed Attempt 1 — `position: sticky`:**
- Applied `position: sticky; bottom: 0` to `.rp-action-bar`.
- Result: Failed. The sticky element was clipped by the parent `overflow: auto` container. The button never stayed in view.

**Failed Attempt 2 — `position: absolute; bottom: 0`:**
- Applied `position: absolute; bottom: 0; left: 0; right: 0` to `.rp-action-bar`. Made `.right-panel` `position: relative`.
- Result: Failed. `.right-panel` had no bounded height — it grew with content. So `bottom: 0` of the absolute bar mapped to a position below the viewport. Button was completely off-screen.

**Failed Attempt 3 — `overflow: visible` on BlockNoteWrapper:**
- Changed `overflow: auto` to `overflow: visible` on the div wrapping `<BlockNoteView>` in `BlockNoteWrapper.jsx`, hoping to prevent nested scroll conflicts.
- Result: Failed. This was a blind guess based on no evidence. It broke BlockNote's read-only rendering — the Note section appeared as an empty white box in Preview mode. Reverted.

**Root Cause (Discovered):**
The actual root cause was that `.app-shell` used `height: 100vh` while being placed inside a CMS layout that already consumed ~65px for the top navigation bar. This caused the entire timeline and its panels to overflow by ~65px below the visible screen. The Save button was always there — it was simply rendered off-screen.

Additionally, `TimelinePage.tsx` had `margin: "-24px 0"` which shifted the layout further, and `height: "calc(100vh - 65px)"` which was a failed attempt to compensate that did not account for all layers.

**Partial Fix Applied (not browser-verified):**
- Removed `position: absolute` from `.rp-action-bar` → reverted to `flex-shrink: 0` (normal flex item).
- Changed `.app-shell` height: `100vh` → `100%`.
- Changed `.timeline-scroll` width/height: `100vw/100vh` → `100%`.
- Removed `py-6` from `<main>` in `App.tsx`.
- Changed `TimelinePage.tsx` to `height: "100%"`, removed `margin: "-24px 0"`.
- Changed `TimelineViewPage.tsx` to `height: "100%"`, removed `margin: "-24px 0"`.

---

### Problem 2: Button Colors (Simpan Perubahan / Batal)

**Issue:** After Save button became somewhat visible, it appeared gray/white with white text — completely illegible.

**Root Cause:** The inline style used `background: 'var(--brand-primary)'`. The CSS variable `--brand-primary` is NOT defined anywhere in the timeline engine's CSS system. The browser fell back to transparent/inherited background.

**Fix Applied:** Replaced `var(--brand-primary)` with hardcoded `#1f2937` (dark gray). Button text color on Batal changed from `var(--text-muted)` to `#1f2937`. This was applied without browser verification.

---

### Problem 3: Note Content Missing in Preview After Save

**Issue:** After clicking Save and then Publish, opening the Preview showed an empty white box in the Note section instead of the note content.

**Root Cause:** The `overflow: visible` change from Failed Attempt 3 (see above) was still in place when the user tested Preview. It broke BlockNote's internal read-only rendering. This was NOT a data/save issue — the content was intact in Appwrite.

**Fix Applied:** Reverted `overflow: visible` → `overflow: auto` in `BlockNoteWrapper.jsx`.

---

### Problem 4: Fragmented BlockNote Architecture (UNRESOLVED)

**Issue:** User discovered that the Timeline Note editor (BlockNoteWrapper.jsx) lacks Upload Image capability that the Article editor has.

**Root Cause:** Two independent BlockNote instances were created for different parts of the app:
1. `src/components/Editor.tsx` — Article editor. Configured with `uploadFile`, math, mentions.
2. `src/timeline-engine/components/BlockNoteWrapper.jsx` — Timeline Note. Configured with math only, missing `uploadFile` and mentions.

**Partial Fix Applied:** Added `uploadFile` Appwrite handler to `BlockNoteWrapper.jsx`. NOT verified.

**UNRESOLVED — Architectural Decision Needed:**
The user decided the correct fix is a full architectural overhaul: one shared BlockNote factory/hook that all editor surfaces (Article, Timeline Note, Canvas Note) use. This ensures feature parity across all formats. No code changes should be made without designing this shared architecture first.
