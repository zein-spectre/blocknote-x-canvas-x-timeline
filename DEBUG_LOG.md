# Debug Log

*Persistent debugging history for tracking failed attempts, evidence, eliminated causes, successful fixes, root causes, and verification.*

## Advanced Mention Modal Overhaul
**Date**: 2026-09-24

**Issue**: 
The native BlockNote mention menu (triggered by `@`) was a single long list. With many files (Article Pages, Canvases, Timelines), this became cluttered and difficult to navigate. The user wanted a custom popup with tabs, search, and multi-select capabilities.

**Evidence/Failed Approaches**:
- Using the default `SuggestionMenuController` only allows a simple dropdown list.
- Attempting to show all collections in one dropdown would cause UX issues (infinite scrolling without categorization).

**Root Cause**:
BlockNote's default suggestion menu is not designed for complex, multi-tab, multi-select linking across different database collections.

**Final Fix**:
1. Modified `Editor.tsx` to intercept the `@` trigger using `SuggestionMenuController`, returning only two options: "Create new Article Page" and "Add Existing Article Page".
2. Created a custom `MentionModal.tsx` that fetches from Articles, Canvases, and Timelines collections.
3. Implemented a tabbed interface (Article, Canvas, Timeline) with independent search bars.
4. Added multi-select checkbox logic and a "Mention" button to insert multiple `noteMention` nodes at once, separated by commas.
5. Integrated `MentionModal` into `Editor.tsx` to spawn when "Add Existing Article Page" is selected.

**Verification**:
Typing `@` now offers the custom trigger. Clicking it opens the advanced modal, allowing cross-collection, searchable, multi-select mentions that insert correctly into the editor.

## Timeline Tag Not Found Bug
**Date**: 2026-09-24

**Issue**: 
Tags added to timeline elements (specifically Era) were not displaying in the left Sidebar under "ALL TAGS", which showed "No tags found".

**Evidence/Failed Approaches**:
- Tags were successfully saved and visible in the Right Panel.
- Sidebar showed "No tags found" despite elements having tags.

**Root Cause**:
In `Sidebar.jsx`, the tag aggregation logic (`useMemo` for `allTags`) had a strict guard: `if (element.type !== "event" && element.type !== "span") return;`. This explicitly excluded the `era` type, so any tags assigned to eras were ignored by the Sidebar.

**Final Fix**:
Removed the restrictive type check. The code now iterates over all elements and properly counts tags regardless of whether the element is an event, span, or era.

**Verification**:
Tags applied to "Abad Kekosongan" (an Era) now successfully show up in the Sidebar under "ALL TAGS".

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

## Fix: Article Mention Click Does Not Navigate
**Date:** 2026-09-23
**Problem:** User clicked the "📝 Artikel 2" mention chip in Article 1 (both in `/admin/edit/:id` edit mode and in `/article/:id` public reader) — nothing happened. No navigation, no console error.

**Root Cause (two layers):**
1. **`PublicReader.tsx` (`/article/:id`)** did not pass `onOpenNote` to the `<Editor>`. The mention chip rendered with `cursor: pointer` but the click handler was gated on `if (onOpenNote)` — undefined → silent no-op.
2. **`React Context` is unreliable as the carrier for the `onOpenNote` callback into ProseMirror NodeViews.** BlockNote renders inline content (including custom `noteMention` chips) through Tiptap/ProseMirror NodeViews, which sit in a separate React render tree. In practice the `[DIAG] 1` console.log placed in the `NoteMention` render does not fire when the user clicks — meaning the React subtree that mounts inside the NodeView never re-resolves `useContext` from the Provider that wraps `<BlockNoteView>`. Pure React Context is the wrong transport.

**Fix:**
1. `src/pages/PublicReader.tsx` — added `onOpenNote={(noteId) => navigate(\`/article/${noteId}\`)}` so the public reader has a working handler in scope.
2. Added a **module-scope fallback handler** (`let __noteClickHandler`) in all three BlockNote factories: `src/components/Editor.tsx`, `src/timeline-engine/components/BlockNoteWrapper.jsx`, `src/pages/BlockNoteTemplate.tsx`. Each component sets it in a `useEffect(() => { __noteClickHandler = onOpenNote; return cleanup })` tied to its `onOpenNote` prop. The `NoteMention` render falls back: `const handler = onOpenNote || __noteClickHandler`. Context still works when it propagates; module-scope is the safety net.
3. `BlockNoteWrapper.jsx` was already missing `onOpenNote` plumbing entirely — added the prop and wrapped `<BlockNoteView>` in the existing `<EditorContext.Provider>` for symmetry, even though the fallback carries the load.

**Why the fix works:**
- Module-scope `let` is set synchronously when the React component mounts (via `useEffect`), so by the time any NodeView is created the handler is already there. NodeViews read it on every render via closure.
- `useEffect` cleanup compares identity and clears only if the same instance — prevents one unmounting Editor from clobbering another's handler.
- `cursor: pointer` was already in place — it only lacked a handler.

**Verification:**
- Vite HMR reloaded all four files (5174) — confirmed via `curl http://localhost:5174/src/...` returning the new code with `__noteClickHandler`, `useEffect`, and `onOpenNote` navigation.
- Browser verification not performed in this session (no headless browser available locally). User must click a mention chip in `/admin/edit/{artikel-1-id}` and in `/article/{artikel-1-id}` and confirm both navigate to `/admin/edit/{artikel-2-id}` and `/article/{artikel-2-id}` respectively.
- Logically equivalent to the `[DIAG] 1` console.log path that the original developer placed but never saw fire.

## Fix: Navigation Worked But Content Stayed Stale (Follow-up)
**Date:** 2026-09-23
**Problem:** After the previous fix, clicking a mention chip did navigate (title switched to "Artikel 2"), but the rendered content remained "Ini percoban sih." — the content of Artikel 1. Same shape as a problem documented earlier.

**Root Cause:**
`<Editor initialContent={content} ...>` reads `initialContent` exactly once at mount via `useCreateBlockNote`. When `useEffect([id])` fires after the route changes and updates `content` to Artikel 2's content, the `<Editor>` instance from Artikel 1 was still mounted with its stale editor state. State updates to `content` did not propagate into the BlockNote document because the editor was already initialized.

**Fix (`src/pages/PublicReader.tsx`):**
1. Added `key={id}` to the `<Editor>` so React unmounts and remounts it whenever the URL `:id` changes — guarantees `initialContent` is re-read.
2. Added `setLoading(true); setError(""); setTitle(""); setContent("");` at the top of the `useEffect` so the previous article's stale values are wiped before the fetch starts. Without this, between the route change and the fetch completing, the previous article's title/content would still be visible.

**Why the fix works:**
- `key={id}` is React's idiomatic "re-mount when this changes" lever; BlockNote's `useCreateBlockNote` runs again and re-parses `initialContent` into a fresh document.
- Resetting state in `useEffect` prevents a flash of the previous article's content while the new fetch is in flight.

**Verification:**
- `curl http://localhost:5174/src/pages/PublicReader.tsx` shows the compiled JSX with `, id, false, ...` as the third argument to `_jsxDEV(Editor, ...)` — that is the `key={id}` prop, confirmed live by Vite HMR.
- Browser-side verification still requires the user: load `/article/{artikel-1-id}`, click "📝 Artikel 2", confirm URL becomes `/article/{artikel-2-id}` and rendered content matches Artikel 2 (not "Ini percoban sih.").

**Things to watch:**
- A brief "Loading article..." flash is now expected between route change and fetched content. This is the desired trade-off — better than stale content.
- AdminEditorPage already has `key={\`${id}-${content?.slice(0, 50)}\`}` so the admin edit flow was not affected by this bug.

## Attempt: Custom `onWheel` handler in PublicReader (INTERRUPTED)
**Date:** 2026-09-23
**What was tried:** Adding `onWheel` handler to the content wrapper div in PublicReader that detects when BlockNote's internal scroll is at its boundary and then propagates to `window.scrollBy()`.
**Result:** INTERRUPTED by user. NOT applied. User explicitly forbade this approach before any file was modified.
**Why it was stopped:** User said "JANGAN SAMPAI KAMU BERHALUSINASII!!!!" — refused to allow further code changes without proper checkpoint.

---

## Session Checkpoint Summary
**Date:** 2026-09-23
**Session goal:** Fix scroll wheel not working on Article Page (`/article/:id`).
**Status:** UNRESOLVED — all approaches failed or not applied.

**Root hypothesis (unconfirmed):** BlockNote's internal `overflow: auto` editor captures all wheel events before they propagate to the window. The `editable={false}` prop makes content non-selectable but does NOT disable the editor's event capture.

**What actually changed:** Nothing — user stopped all debugging before any code was modified.

**Next step:** User to provide new direction or clarification.

---

**Things That Must NOT Be Repeated:**
1. **Do NOT apply any source code changes without explicit user approval.**
2. **Do NOT assume cursor/pointer movement = scroll wheel movement** — they are different.
3. **Do NOT modify BlockNote's `overflow: auto`** — Attempt 6 showed it breaks rendering.
4. **Do NOT make blind CSS guesses** — each failed attempt consumed time without evidence.
5. **Do NOT assume the document is tall enough** — measure document height to confirm overflow is happening.
6. **Do NOT write code without reading the actual source files first.**
7. **Do NOT apply `onWheel` handler approach** — not approved, may break BlockNote's internal scroll.

## Fix: Cannot Scroll Down on Article Page (PublicReader)
**Date:** 2026-09-23
**Problem:** After the previous fix, the user could open `/article/{id}` and click a mention, but they were stuck — could not scroll down to see the rest of the article. Title bar of the page was fine, but page content was clipped and unreachable.

**Root Cause:**
`App.tsx` wrapped the entire routed area in `<main className="h-[calc(100vh-65px)]">` — a *fixed-height* container. The HTML document therefore had height `100vh`; anything inside `<main>` that exceeded `100vh - 65px` (e.g. a long article) overflowed visually but the document itself was not tall enough for the window scrollbar to engage. Result: clipped content, no way to scroll.

**Fix (three coordinated changes):**
1. `src/App.tsx` — `<main>` switched from `h-[calc(100vh-65px)]` to `min-h-[calc(100vh-65px)]`. Now `<main>` grows with its content; long articles make the document taller and the window scrolls naturally.
2. `src/pages/TimelinePage.tsx` — its root `<div>` previously had `height: "100%"`, which resolves to `auto` when the parent is `min-h`. Replaced with `height: "calc(100vh - 65px)"` so Timeline still fills the viewport regardless of the new parent rule.
3. `src/pages/CanvasPrototypePage.tsx`, `src/pages/CanvasViewPage.tsx`, `src/pages/BlockNoteTemplate.tsx`, `src/pages/TimelineViewPage.tsx` — verified their roots already use explicit `height: calc(100vh - ...)` or `100vh`, so they are unaffected by the `<main>` change. No edits required.

**Why the fix works:**
- `<main>` now has a *minimum* height equal to the viewport, and grows beyond it when children are tall. Document height = max(viewport, child heights). Window scroll appears whenever a child overflows.
- Timeline/Canvas pages that need a viewport-anchored canvas set their own explicit `calc(100vh - navHeight)` — they no longer rely on `<main>` being a fixed-height flex parent.

**Verification:**
- Vite HMR confirmed via `curl http://localhost:5174/src/App.tsx` → `className: "min-h-[calc(100vh-65px)]"`.
- Vite HMR confirmed via `curl http://localhost:5174/src/pages/TimelinePage.tsx` → `height: "calc(100vh - 65px)"`.
- Browser-side verification still requires the user: open any `/article/{id}` with content taller than one viewport, confirm the page scrolls and all content is reachable. Then open `/timeline/{id}` and confirm Timeline still fills the viewport (no collapse).

**Things to watch:**
- If a new page is added with `height: 100%` at its root, it will collapse to auto under the new `<main>` rule. New pages with full-viewport layouts must use an explicit `calc(100vh - ...)` themselves — the same rule TimelinePage now follows.

**Things to watch:**
- If `onOpenNote` prop identity changes every render, `useEffect` will reassign the module-scope handler on every render — still correct, but wasteful. The two callers (`AdminEditorPage`, `PublicReader`) pass inline arrows, so this is expected. No need to memoize now.
- Module-scope state is global per tab. If two `<Editor>` instances ever mount with different `onOpenNote` props simultaneously, the last-mounted wins for any new NodeView. Currently only one Editor is visible at a time, so this is a non-issue.

## Enforcing Article Page inside Canvas
**Date:** 2026-09-24
**Problem:** The CanvasPrototypePage and CanvasViewPage had a split personality where they supported both "Appwrite Note" (Article Page) and a legacy local `notesData` implementation for `embed-` links. This violated the architecture rule that "setiap page atau note di canvas ini sudah menggunakan Article Page".
**Root Cause:** The canvas prototype originally retained local note fallback logic (`notesData` and `selectedNoteId` state) which did not hook into the central Appwrite CMS system.
**Fix:**
- Stripped all `notesData` and `selectedNoteId` state references from `CanvasPrototypePage.tsx`.
- Removed `handleContentChange` and `handleTitleChange` for local notes.
- Updated `renderEmbeddable` and `handleChange` to only process `note://` links that resolve to Appwrite.
- Removed legacy `!link.startsWith("note://embed-")` checks in both `CanvasPrototypePage.tsx` and `CanvasViewPage.tsx` since all valid links must now refer to Appwrite DB IDs.
**Verification:**
- Ran `npm run build` — encountered TS errors for unused state variables (`setSelectedNoteId` and `useEffect`). Fixed them and rebuilt successfully.
- Code now strictly enforces the "Article Page" rule within the Canvas components.

## Fix: Canvas Article Page Searchability & Excalidraw UI Hardening
**Date:** 2026-09-24
**Problem:** The user tried to use Excalidraw's built-in "Find on canvas" feature (magnifying glass) to search for "Article Page Canvas 1". Excalidraw found nothing.
**Root Cause:** The Article Page box on the canvas was rendered as an Excalidraw `embeddable` containing a custom React node. Excalidraw's native search *only* searches the `.text` property of elements with `type: "text"`. Since the title was in React state (fetched from Appwrite) and rendered over the canvas, Excalidraw's engine was completely blind to it.
**Final Fix:**
1. **Paradigm Shift:** Removed the custom "Tambah Article Page Box" button from `CanvasPrototypePage.tsx`. Forced the user to rely exclusively on typing `@` inside standard Excalidraw text elements to link to articles.
2. **Client-Side Filtering:** Updated `SlashMenu.tsx` to use `articles.filter()` (client-side substring matching) instead of `Query.search()` (Appwrite strict full-text index) to ensure `@` mentions can be found easily (e.g. typing `@art` finds `Artikel 2`).
3. **UI Hardening:** Disabled all distracting Excalidraw UI components via `index.css`:
   - Hidden Social Links (GitHub, Twitter, Discord).
   - Hidden Library Book icon via strict `[role="tab"]` and `[title="Library"]` selectors.
   - Disabled "Save to..." via `UIOptions` (`export: false`).
**Why the fix works:**
By migrating article representations completely to standard Excalidraw text nodes with the `@` mention string (e.g., `📝 Artikel 2`), the article title physically exists in the Excalidraw JSON structure. Thus, Excalidraw's native "Find on canvas" finds it flawlessly.
**Verification Performed:**
- Evaluated Excalidraw DOM rules (tested CSS roles and nth-child for Library hiding).
- User confirmed the `@` search filtering works seamlessly.
- User verified that the Library tab and Export buttons are successfully hidden from the UI.
**Important Lessons:**
- Do not fight the host framework's core features (e.g., search). If Excalidraw expects data in text elements, inject data into text elements rather than floating React `embeddables` over the top, unless you're prepared to reimplement the search feature.
