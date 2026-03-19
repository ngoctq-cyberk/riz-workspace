# FSD Layers

**Layer Hierarchy (Top to Bottom)**:
1. `app/` (Next.js Routing Shell)
2. `screens/` (Page Composition)
3. `widgets/` (Reusable Blocks)
4. `features/` (Mutations)
5. `entities/` (Queries)
6. `shared/` (Infrastructure)

## app/
**Routing Shell**.
- Contains: `page.tsx`, `layout.tsx`, providers.
- Role: Route definitions, metadata, global context. No business logic.

## screens/
**Page Composition**.
- Naming: `screens/{page-name}`
- Role: Assemble widgets, features, and entities into a full page.
- Scope: One screen = One route.

## widgets/
**Reusable UI Blocks**.
- Naming: `widgets/{block-name}` (e.g., `Header`, `Sidebar`, `PostFeed`)
- Role: Large, self-contained UI sections used on **multiple** pages.
- Rule: If used on only one page → keep in `screens`.

## features/
**User Actions (Mutations)**.
- Naming: `features/{action}-{entity}` (e.g., `create-post`, `delete-comment`)
- Role: Handle **writes** (create/update/delete).
- Rule: Must contain mutation logic. Pure UI goes to `shared` or `entities`.

## entities/
**Business Data (Queries)**.
- Naming: `entities/{entity-name}` (e.g., `user`, `post`, `product`)
- Role: Handle **reads** (GET/Search). Shared UI components (cards, rows).
- Rule: No mutations here.

## shared/
**Infrastructure**.
- Role: Reusable code without business domain knowledge.
- Contains: `shadcn/` (CLI-managed components), `ui/` (custom components), `api/` (clients), `lib/` (utilities), `providers/`.
- `api/`, `lib/`, `providers/` have `index.ts` barrels — import via `@/shared/api`, `@/shared/lib`, `@/shared/providers`.
- `shadcn/` and `ui/` do NOT have barrels — import directly: `@/shared/shadcn/button`, `@/shared/ui/loader`.
