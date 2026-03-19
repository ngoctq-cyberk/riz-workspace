---
name: cyberk-fsd-fe
description: Definitive guide for building Next.js apps with FSD architecture and ORPC. Use when adding new features, creating entities, or deciding where code belongs.
---

# CyberK FSD Frontend Skill

**Architecture**: Feature Sliced Design (FSD) + Next.js App Router.
**State/API**: ORPC + React Query (TanStack Query).

## Quick Decision Tree
1. **Adding a Page?** → Create `screens/{name}`.
2. **Adding a Reusable UI Block?** (Header, Sidebar) → Create `widgets/{name}`.
3. **Adding a User Action?** (Create, Update, Delete) → Create `features/{action}-{entity}`.
4. **Adding Business Data?** (Read-only, Search) → Create `entities/{name}`.
5. **Adding Generic UI?** (Button, Input) → Use `shared/shadcn` (shadcn CLI-managed, direct import).

## Core References (Read These)
- **[Layers Definition](references/fsd-layers.md)**: What goes where.
- **[Code Patterns](references/fsd-segments.md)**: How to write Queries & Mutations.
- **[Import Rules](references/fsd-import-rules.md)**: Strict dependency rules.
- **[Next.js Conventions](references/nextjs-conventions.md)**: Route groups and specific files.

## Critical Implementation Rules
1. **Server Components First**: Default to RSC. Use `"use client"` only for interactivity.
2. **Strict Imports**: Never import from the same layer (e.g., entity importing another entity). Pass props or use slots.
3. **ORPC Pattern**: 
    - Queries defined in `entities/{name}/api/*.queries.ts`.
    - Mutations defined in `features/{name}/api/use-*.ts`.
