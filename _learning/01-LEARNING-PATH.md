# Learning Path

Use this order. It starts with easiest files and builds toward the most complex one.

## Chapter 0: Project Orientation
Read:
1. `_learning/00-ARCHITECTURE.md`
2. [package.json guide](guides/package.json.guide.md)

Checkpoint:
- You can explain what the app does end-to-end.
- You understand which libraries are used and why.
- You know where UI, API, and persistence code lives.

## Chapter 1: App Shell Basics
Read:
1. [src/app/layout.js guide](guides/src/app/layout.js.guide.md)
2. [src/app/page.js guide](guides/src/app/page.js.guide.md)
3. [src/app/globals.css guide](guides/src/app/globals.css.guide.md)

Checkpoint:
- You understand Next.js App Router layout/page composition.
- You can trace how global styles shape the UI structure.
- You can point to where fonts, metadata, and root shell are configured.

## Chapter 2: Utility Layer Foundations
Read:
1. [src/lib/workspaceUtils.js guide](guides/src/lib/workspaceUtils.js.guide.md)
2. [src/lib/markdown.js guide](guides/src/lib/markdown.js.guide.md)
3. [src/lib/defaultWorkspace.js guide](guides/src/lib/defaultWorkspace.js.guide.md)

Checkpoint:
- You can explain recursive tree building and breadcrumb generation.
- You can explain markdown parsing/serialization in this project.
- You understand the default domain model shape for pages/blocks/users.

## Chapter 3: Persistence and Offline Data
Read:
1. [src/lib/sqlite.js guide](guides/src/lib/sqlite.js.guide.md)
2. [src/lib/offlineStore.js guide](guides/src/lib/offlineStore.js.guide.md)

Checkpoint:
- You can explain how server persistence works from startup to write.
- You can describe why IndexedDB queue/drafts improve reliability.
- You can identify tradeoffs of full-state blob persistence.

## Chapter 4: API Route Layer
Read:
1. [src/app/api/workspace/route.js guide](guides/src/app/api/workspace/route.js.guide.md)
2. [src/app/api/workspace/version/route.js guide](guides/src/app/api/workspace/version/route.js.guide.md)
3. [src/app/api/search/route.js guide](guides/src/app/api/search/route.js.guide.md)
4. [src/app/api/import/markdown/route.js guide](guides/src/app/api/import/markdown/route.js.guide.md)
5. [src/app/api/export/markdown/route.js guide](guides/src/app/api/export/markdown/route.js.guide.md)
6. [src/app/api/export/html/route.js guide](guides/src/app/api/export/html/route.js.guide.md)

Checkpoint:
- You can trace how each endpoint receives input and returns output.
- You can distinguish read/write/version/import/export responsibilities.
- You can point out current validation and missing validation.

## Chapter 5: State Management Core
Read:
1. [src/lib/workspaceStore.js guide](guides/src/lib/workspaceStore.js.guide.md)

Checkpoint:
- You can explain every major store slice.
- You understand immutable updates for deeply nested data.
- You can add a new action safely without mutating old state.

## Chapter 6: Main UI Orchestration
Read:
1. [src/components/WorkspaceApp.js guide](guides/src/components/WorkspaceApp.js.guide.md)

Checkpoint:
- You can trace user action -> store update -> API sync.
- You understand why there are many `useEffect` hooks and what each one does.
- You can explain drag/drop, command palette, comments, version restore, and import flow.

## Chapter 7: Engineering Judgment
Read:
1. `_learning/02-GLOSSARY.md`
2. `_learning/03-PATTERNS.md`

Checkpoint:
- You can name patterns this codebase uses and where.
- You can identify anti-patterns and propose safer alternatives.
- You can suggest a next refactor roadmap with tradeoffs.

## Suggested Practice Schedule
- Day 1: Chapters 0-2
- Day 2: Chapters 3-4
- Day 3: Chapter 5
- Day 4: Chapter 6
- Day 5: Chapter 7 + implement one improvement

## Final Intern Challenge
After finishing all chapters, implement these in order:
1. Add a "duplicate page" action in store + UI.
2. Add API input validation for at least one route using a schema library (none is installed yet; `zod` is a common choice).
3. Replace one `window.prompt` interaction with a real modal component.
4. Add a basic test for `markdownToBlocks`. The project has no test runner or `test` script yet, so start by adding one (Node's built-in `node --test` works if the test does not depend on the Next.js `@/` import alias).

## Bug-Hunt Exercises (Goal / Check)
Each of these starts from a gap documented in `00-ARCHITECTURE.md` section 11. Do them after Chapter 6.

1. **Goal:** see the cross-tab state sync fail. Read `applyRemoteState` (`src/lib/workspaceStore.js:366-387`); the early return on line 370 drops any message whose `meta.updatedAt` is not strictly newer.
   **Check:** open the app in two tabs, rename a page in tab A, wait a few seconds. Tab B shows A's presence and typing, but the sidebar title in B does not change until you reload B.
2. **Goal:** make the sync work by giving every local edit a newer clock. One option: have the autosave effect (`src/components/WorkspaceApp.js:568-598`) read `updatedAt` from the `PUT` response (`src/app/api/workspace/route.js:54` returns it) and write it into `meta`.
   **Check:** repeating exercise 1 updates tab B's sidebar without a reload.
3. **Goal:** stop losing queued saves. In `flushSyncQueue` (`WorkspaceApp.js:442-456`), only call `removeQueuedSync` when `response.ok`.
   **Check:** with the API temporarily throwing in `PUT`, edit a page; the entry in the `syncQueue` store of the `notion-clone-offline` IndexedDB database (DevTools > Application > IndexedDB) survives a reload instead of disappearing.
4. **Goal:** make "Saved" honest. `decrementPendingSaves` (`src/lib/workspaceStore.js:417-418`) always stamps `lastSavedAt`; split it so the catch branch at `WorkspaceApp.js:592-593` does not.
   **Check:** with the network offline in DevTools, editing leaves the header on a label you add for queued saves (today it can only show `Saving...`, `Saved <time>` or `Unsaved`, `WorkspaceApp.js:829-833`) instead of `Saved <time>`.
5. **Goal:** make `GET /api/workspace` a pure read. `getWorkspaceState` (`src/lib/sqlite.js:184-193`) inserts a seeded workspace for an unknown id.
   **Check:** `GET /api/workspace?workspaceId=does-not-exist` returns a 404 (or an explicit empty response), and `SELECT COUNT(*) FROM workspace_state` in `.data/workspace.sqlite` is unchanged afterwards.
