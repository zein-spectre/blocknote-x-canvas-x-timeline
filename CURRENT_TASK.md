# Current Task: BlockNote Architecture Overhaul (UNRESOLVED)

**STATUS: STOPPED — PROBLEM NOT RESOLVED**

---

## Current Problem

The BlockNote editor system in this project is **fragmented**: each feature area (Article editor, Timeline Note panel, potentially Canvas) has its **own separate BlockNote instance**, each configured independently. This means capabilities like `uploadFile`, math support, mention support, etc. can be **missing or different** from one area to another.

The user discovered this when the Note panel inside the Timeline right panel lacked the "Upload Image" tab that the Article editor has. This is a **systemic architectural flaw**, not a one-off bug.

The user's position: **The entire BlockNote system needs to be rebuilt from scratch with a unified, Notion-style architecture** — one shared page/block system that all formats (Article, Timeline Note, Canvas Note, etc.) draw from consistently.

---

## Goal

Redesign the BlockNote integration so there is **one canonical BlockNote configuration** (schema + uploadFile + extensions) that is shared across all formats. Each format-specific editor should be a thin wrapper around this shared core — not an independently configured instance.

---

## Confirmed Facts

1. **There are at least 2 separate BlockNote instances in this project:**
   - `src/components/Editor.tsx` — the main Article editor. Has `uploadFile` (Appwrite Storage), math, mention support.
   - `src/timeline-engine/components/BlockNoteWrapper.jsx` — the Timeline Note editor. Was missing `uploadFile`. Math support exists. No mention support.

2. **`uploadFile` was missing from `BlockNoteWrapper.jsx`** from the beginning (it was never configured, not broken by recent changes). Confirmed by reading source code.

3. **The `overflow: visible` "fix"** applied to `BlockNoteWrapper.jsx` in this session was a failed blind guess that temporarily broke Note rendering in Preview/ReadOnly mode. It was reverted.

4. **The `--brand-primary` CSS variable does not exist** in the timeline engine's CSS system. Using it caused the Simpan Perubahan button to render with an invisible background.

5. **The "gray area / border at the bottom"** of the timeline preview was caused by `margin: "-24px 0"` in `TimelinePage.tsx` and `TimelineViewPage.tsx`. The negative margin pulled the timeline up, leaving a white gap at the bottom.

6. **`height: 100vh`** in `.app-shell` and `.timeline-scroll` caused overflow beyond the CMS container since the CMS nav takes ~65px.

7. **`rp-action-bar` with `position: absolute; bottom: 0`** did not work because `.right-panel` had no stable bounded height. The bar fell outside the visible area. Reverted to a normal flex item.

---

## Failed Approaches (This Session)

### Attempt 1: `position: sticky` on `rp-action-bar`
- **Result**: Failed. CSS sticky does not work when the parent has `overflow: auto/hidden`.

### Attempt 2: `position: absolute; bottom: 0` on `rp-action-bar`
- **Result**: Failed. `.right-panel` grew with content so `bottom: 0` fell below viewport. Bar was completely off-screen.

### Attempt 3: `overflow: visible` on BlockNoteWrapper inner div
- **Result**: Failed. Blind guess. Broke BlockNote rendering in read-only mode. Note appeared as empty white box. Reverted.

### Attempt 4: `var(--brand-primary)` for Save button color
- **Result**: Failed. Variable is not defined in timeline CSS. Button rendered invisible (white text on transparent background).

---

## Evidence Discovered

- "Missing upload tab" is NOT a bug — it is a fundamental architectural gap. `BlockNoteWrapper.jsx` was created as a stripped-down copy without `uploadFile`.
- Correct `uploadFile` implementation exists in `src/components/Editor.tsx` lines 191–199.
- Root CSS problem: `.app-shell` used `height: 100vh` while inside a CMS container that already occupies ~65px for top nav. This caused ~65px of content to be clipped below visible area.

---

## Changes Made This Session (Partial Fixes — NOT fully verified)

| File | Change |
|------|--------|
| `06-right-panel.css` | Removed `position: absolute` from `.rp-action-bar`, reverted to flex-shrink:0 |
| `06-right-panel.css` | Removed `padding-bottom: 70px` from `.right-panel-content` |
| `03-layout.css` | Changed `.app-shell` height from `100vh` to `100%` |
| `04-timeline.css` | Changed `.timeline-scroll` width/height from `100vw/100vh` to `100%` |
| `App.tsx` | Removed `py-6` padding from `<main>` |
| `TimelinePage.tsx` | Changed height to `100%`, removed `margin: "-24px 0"` |
| `TimelineViewPage.tsx` | Changed height to `100%`, removed `margin: "-24px 0"` |
| `RightPanel.jsx` | Changed Simpan/Batal button colors to solid `#1f2937` |
| `BlockNoteWrapper.jsx` | Added `uploadFile` Appwrite handler |
| `BlockNoteWrapper.jsx` | Reverted `overflow: visible` → `overflow: auto` |

---

## Current State

- Bottom border/gap: partially addressed, NOT browser-verified.
- Simpan Perubahan button visibility: partially addressed, NOT browser-verified.
- Upload Image in Timeline Note: patched with uploadFile, NOT verified.
- Core architectural problem (fragmented BlockNote): NOT resolved. User stopped to rethink.

---

## Exact Next Step (When Resuming)

Design a **unified BlockNote architecture** before writing any code:

1. Create a single `createSharedEditor(options)` factory or `useSharedBlockNote()` hook containing:
   - Canonical schema (defaultBlockSpecs + mathBlock + inlineContentSpecs)
   - The `uploadFile` handler (Appwrite Storage)
   - The `pasteHandler` (math-aware paste)
   - Mention and other extension specs
2. All editor instances (Article, Timeline Note, Canvas Note) call this shared factory.
3. Replace `BlockNoteWrapper.jsx` with a thin wrapper calling the shared factory.
4. Test feature parity across all editor surfaces.

---

## Things That Must NOT Be Repeated

1. **Do NOT use `position: absolute; bottom: 0` on `rp-action-bar`** unless parent has explicit bounded height. Tried twice — failed both times.
2. **Do NOT use undefined CSS variables** (e.g., `var(--brand-primary)`). Always verify variable existence first.
3. **Do NOT change `overflow` on BlockNoteView wrapper to `visible`** — breaks read-only rendering.
4. **Do NOT claim success without browser verification.** All "successes" in this session were wrong assumptions.
5. **Do NOT create new per-feature BlockNote configurations** without updating the shared factory first.
