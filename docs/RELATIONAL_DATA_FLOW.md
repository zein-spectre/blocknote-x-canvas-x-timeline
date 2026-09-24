# Relational Data Flow (Canvas & Article Page)

## Core Architecture

This project strictly adheres to the rule that all text notes and structured writing in the canvas exist as **Article Pages** stored in Appwrite (`articles` collection). The Canvas acts as a spatial organizer and mapping tool, but not as an independent document database.

### The Problem with Custom Embeddables (Resolved)
Initially, "Article Pages" were rendered onto the Excalidraw canvas as custom `embeddable` IFrames/React components (e.g., clicking a custom "Article Page" button). 

**Critical Flaw:** Excalidraw's native "Find on canvas" search feature *only* searches string properties inside elements with `type: "text"`. Since the `embeddable` title was fetched live from Appwrite and rendered via React DOM (not saved in Excalidraw's element JSON data), Excalidraw's search engine could not find any of the articles placed on the canvas.

### The Solution: `@` Mention System
To solve the searchability issue and simplify the UX (similar to Notion/Affine), we discarded the custom `embeddable` Article Page UI button and shifted entirely to a text-based Mention system.

**Data Flow:**
1. **Creation/Search:** The user creates a standard Excalidraw text element or rectangle and types `@`. This triggers the `SlashMenu.tsx` popover.
2. **Client-Side Filtering:** `SlashMenu` fetches articles from Appwrite and performs a flexible client-side substring search (insensitive to case), displaying matches.
3. **Storage:** When an article is selected, the title is injected into the text element (e.g., `📝 Artikel 2`). The relational data (`article_id`) is stored in the Excalidraw element's `customData.mentions` array alongside its text offset.
4. **Rendering & Interactivity:** `CanvasSceneLoader.tsx` reads `customData.mentions` and computes screen coordinates for the text offsets. It renders HTML-based blue highlight chips *on top* of the Excalidraw canvas, exactly covering the text.
5. **Searchability:** Because the text string "📝 Artikel 2" physically exists in the Excalidraw `type: "text"` element's `text` property, the native Excalidraw "Find on canvas" (magnifying glass) finds it perfectly.

## Excalidraw UI Hardening
To prevent users from straying from this paradigm, Excalidraw's UI has been aggressively pruned:
- `UIOptions` was used to disable `loadScene`, `export`, `saveToActiveFile`, `saveAsImage`, and `clearCanvas`.
- **CSS Injection (`index.css`):**
  - Social links (GitHub, Twitter, Discord) and their headers are hidden via strict selectors.
  - The "Library" feature (Book icon) and its associated sidebar tab are permanently disabled via `[role="tab"]` and `[title="Library"]` selectors, leaving only the "Find on canvas" (Search) magnifying glass.
  - The custom "Tambah Article Page Box" button was deleted from `renderTopRightUI`.

**Future AI Agent Directives:**
- Do not attempt to reintroduce custom React `embeddable` cards for Articles. The `@` mention system inside standard text nodes is the canonical way Articles exist on the Canvas.
- If you need to search or link articles, utilize the `articles` Appwrite collection.
- Excalidraw UI modifications should be done via `UIOptions` when possible, and CSS injection (`index.css`) for stubborn elements. Do not hack Excalidraw's React children.
