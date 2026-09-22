# Timelines Project Overview

## Tech Stack
- Frontend: React 19, Vite
- Desktop: Electron 41
- Libraries: Leaflet (react-leaflet) for map view, marked/dompurify for Markdown rendering, isomorphic-git for git sync, html2canvas/fflate.

## Commands
- **Install:** `npm install`
- **Run (Dev):** `npm run dev` (Web only) or `npm run electron:dev` (Desktop)
- **Run (Viewer):** `npm run dev:viewer`
- **Build (Desktop):** `npm run electron:build`
- **Build (Viewer):** `npm run build:viewer`
- **Test:** `npm test`
- **Lint:** `npm run lint`

## Important Project Rules
- The project is local-first. Data is stored in `.timeline` and `.md` files.
- Ensure cross-platform compatibility (macOS, Windows, Linux).
- Maintain concise and clear codebase logic, respecting React and Electron patterns.
