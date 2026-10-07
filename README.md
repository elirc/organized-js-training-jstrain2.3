# Real-Time Collaborative Document & Knowledge Base

A Notion-style collaborative workspace built with Next.js (App Router), SQLite, Zustand, dnd-kit, and Fuse.js.

## Stack

- Next.js 16 + React 19
- JavaScript only (no TypeScript)
- SQLite (`better-sqlite3`) for persistence
- Zustand for client state management
- dnd-kit for block drag/drop reordering
- Fuse.js for full-text quick find
- IndexedDB for offline draft + sync queue

## Implemented Features

- Block-based editor:
  - paragraph, heading1-3, bullet, number, callout, code, divider, database blocks
  - per-block type switching
  - drag-and-drop block reordering
  - inline comments on blocks with threaded replies and resolve/unresolve
- Page hierarchy:
  - nested pages with recursive sidebar tree
  - expand/collapse tree navigation
  - breadcrumbs
  - move page between parents
  - page icon + cover image URL
- Collaboration mechanics:
  - cross-tab live state sync via `BroadcastChannel`
  - presence ("X is viewing this page")
  - live typing indicators per block
  - conflict resolution by latest `meta.updatedAt` state (see Known limitations: in practice this blocks most cross-tab state updates)
- Offline support:
  - workspace draft stored in IndexedDB
  - failed saves (network error or non-OK status) are queued in IndexedDB and replayed on the `online` event and on the next page load
  - debounced autosave (650 ms)
- Databases:
  - inline database block with typed properties
  - row editing
  - table, board, calendar and gallery view rendering; the seeded `list` view has no renderer of its own and shows the table
  - row templates exist in the seed data, but no UI creates rows from them
- Search:
  - fuzzy API search (Fuse.js, top 30 matches) across page titles and block text, including trashed pages
  - command palette (`Cmd/Ctrl + K`)
  - recent pages shown in palette when query is empty
- Sharing/permissions:
  - per-page privacy mode (`private`, `workspace`, `public`) stored on the page; no route reads or enforces it
- Import/export:
  - import Markdown into a new page
  - export page as Markdown
  - export page as HTML
- Favorites/recent:
  - favorite page toggles
  - recent page tracking
- Trash:
  - soft delete + restore
- Version history:
  - snapshot capture
  - list snapshots per page
  - diff summary and restore

## Data Model Coverage

SQLite schema includes:

- `workspaces`
- `users`
- `workspace_members`
- `pages`
- `blocks`
- `databases`
- `database_properties`
- `database_views`
- `database_view_filters`
- `database_view_sorts`
- `comments`
- `comment_threads`
- `page_versions`
- `templates`
- `shared_links`
- `favorites`
- `page_activity`

The app persists current operational state in `workspace_state` for fast load/sync. Only two tables are ever written: `workspace_state` (the whole workspace as one JSON blob) and `page_versions` (snapshots). The other 16 tables are created by `ensureSchema` in `src/lib/sqlite.js` and stay empty. The database file is `.data/workspace.sqlite`, created on the first API request.

## API Endpoints

- `GET /api/workspace` load workspace state
- `PUT /api/workspace` save workspace state
- `POST /api/workspace` create page snapshot version
- `GET /api/workspace/version?pageId=...` list page versions
- `GET /api/workspace/version?versionId=...` get one version payload
- `PATCH /api/workspace/version` restore a version
- `GET /api/search?q=...` full-text search
- `POST /api/import/markdown` import Markdown page
- `GET /api/export/markdown?pageId=...` export page markdown
- `GET /api/export/html?pageId=...` export page HTML

## Run

```bash
npm install
npm run dev
```

Open `http://localhost:3000`.

## Verification

```bash
npm run lint
npm run build
```

There are no automated tests in this repository and no `test` script. `better-sqlite3` is a native module, so `npm install` must be able to fetch or build its binary for your Node version.

## Known limitations

Checked against the code on 2026-10-06:

- **Cross-tab state sync rarely applies.** `applyRemoteState` (`src/lib/workspaceStore.js:366-387`) ignores a broadcast unless its `meta.updatedAt` is newer than the receiving tab's. Editing never changes `meta.updatedAt` on the client; only the server stamps it on `PUT`, and the client does not read that value back. Two tabs loaded from the same saved state therefore ignore each other's edits. Presence and typing indicators do work across tabs.
- **The IndexedDB draft is almost never used.** The bootstrap effect (`src/components/WorkspaceApp.js:458-490`) prefers the draft only when its `meta.updatedAt` is newer than the server's, which the point above makes nearly impossible. If the server is unreachable at load, the `fetch` rejects and the app shows an error instead of the draft.
- **Queued saves are dropped on HTTP errors.** `flushSyncQueue` (`WorkspaceApp.js:442-456`) removes an item after any HTTP response, including a 500. Only network failures stop the replay.
- **"Saved" can mean "queued".** `decrementPendingSaves` sets `lastSavedAt` in a `finally` block, so the header shows a save time even when the save failed and was queued.
- **Last write wins on the server.** Every autosave `PUT` replaces the whole workspace blob, so two tabs saving in turn overwrite each other.
- **No authentication.** `actorId` is taken from the request body, and any `workspaceId` passed to a `GET` creates a new seeded workspace row (`src/lib/sqlite.js:184-193`).
- **Unused dependencies.** `prisma`, `@prisma/client`, `yjs`, `y-websocket`, `uuid`, `date-fns` and `lucide-react` are listed in `package.json` but not imported anywhere in `src/`.

## About this snapshot

This repository is a snapshot of [elirc/jstrain2.3](https://github.com/elirc/jstrain2.3) taken before that repository's rewrite. The maintained repository keeps this version of the code under `docs/original/` and adds an `astraupskill/` course and automated tests, neither of which exists here. The README and `_learning/` docs in this snapshot were re-checked against this code on 2026-10-06.

