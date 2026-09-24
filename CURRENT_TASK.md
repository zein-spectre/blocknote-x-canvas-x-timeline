# CURRENT_TASK.md

## Status: COMPLETED

## Previous Problem (RESOLVED)
**Cannot scroll down on Article Page (`/article/:id`)** — This has been fixed. `PublicReader.tsx` contains an `onWheel` handler and `key={id}` re-mount. `App.tsx` uses `<main className="flex-1">` which allows natural document scrolling.
**Canvas Article Page Searchability** — Excalidraw's "Find on canvas" failed to find custom `embeddable` Article Pages. Fixed by removing the Article Page embeddable button and forcing users to link to articles exclusively via `@` mentions inside standard Excalidraw text blocks, which Excalidraw can natively search.

## What Is Currently Working
1. ✅ Article page (`/article/:id`) — scrolls via window, mention chips navigate correctly, content refreshes on route change.
2. ✅ Admin editor (`/admin/edit/:id`) — mention autocomplete (`@`), mention chip navigation, content switching with correct key.
3. ✅ Timeline page (`/timeline/:id`) — viewport-anchored layout, save/publish buttons visible, sequential timeline ticks display correctly.
4. ✅ Canvas page (`/canvas/:id`) — mention chips route properly. Article integration relies solely on standard `@` mentions for perfect compatibility with Excalidraw's "Find on canvas" search. Extraneous Excalidraw UI (Export, Library, Social Links) is hidden.
5. ✅ BlockNoteWrapper.jsx — has `uploadFile`, `noteMention` schema, `@` suggestion menu, `onOpenNote` module-scope fallback.
6. ✅ AdminDashboard — uses `Promise.allSettled` for graceful degradation.

## Pending Architectural Issue (UNRESOLVED)
**Fragmented BlockNote Architecture** — Two independent BlockNote instances exist:
- `src/components/Editor.tsx` — Article editor (canonical)
- `src/timeline-engine/components/BlockNoteWrapper.jsx` — Timeline Note editor

Both have been manually kept in feature parity (uploadFile, math, noteMention, paste handler, onOpenNote), but there is no single shared factory/hook. Any future feature added to one must be manually copied to the other. The architectural decision to create a shared BlockNote factory is documented but not yet implemented.

**No code changes should be made to this until user provides explicit direction.**

## Next Step
**COMPLETED** — All documented canvas search bugs and UI cleanup tasks are complete. Awaiting user's next instructions.

