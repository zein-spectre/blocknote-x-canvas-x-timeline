# CURRENT_TASK.md

## Status: COMPLETED — FINALISASI PERTAMA

**Finalized:** 2026-09-24

## What Is Currently Working
1. ✅ Article page (`/article/:id`) — scrolls via window, mention chips navigate correctly, content refreshes on route change.
2. ✅ Admin editor (`/admin/edit/:id`) — mention autocomplete (`@`), mention chip navigation, content switching with correct key.
3. ✅ Timeline page (`/timeline/:id`) — viewport-anchored layout, save/publish buttons visible, sequential timeline ticks display correctly.
4. ✅ Canvas page (`/canvas/:id`) — mention chips route properly. Article integration relies solely on standard `@` mentions for perfect compatibility with Excalidraw's "Find on canvas" search. Extraneous Excalidraw UI (Export, Library, Social Links, Dark mode toggle) is hidden.
5. ✅ BlockNoteWrapper.jsx — has `uploadFile`, `noteMention` schema, `@` suggestion menu, `onOpenNote` module-scope fallback.
6. ✅ AdminDashboard — uses `Promise.allSettled` for graceful degradation.
7. ✅ Mention click — ALL mention clicks (Admin, Preview, Canvas) open in a new tab. No same-tab navigation risk.
8. ✅ Timeline Preview (`/view/timeline/:id`) — renders correctly with proper `calc(100vh - 65px)` height. Empty timeline fallback prevents blank screen.
9. ✅ Timeline span/event/era rendering — `file.start`/`file.end` empty string edge case fixed; elements no longer get clamped to year 0 when bounds are unset.

## Pending Architectural Issue (UNRESOLVED)
**Fragmented BlockNote Architecture** — Two independent BlockNote instances exist:
- `src/components/Editor.tsx` — Article editor (canonical)
- `src/timeline-engine/components/BlockNoteWrapper.jsx` — Timeline Note editor

Both have been manually kept in feature parity (uploadFile, math, noteMention, paste handler, onOpenNote), but there is no single shared factory/hook. Any future feature added to one must be manually copied to the other. The architectural decision to create a shared BlockNote factory is documented but not yet implemented.

**No code changes should be made to this until user provides explicit direction.**

## Next Step
Awaiting user's next instructions for Phase 2.
