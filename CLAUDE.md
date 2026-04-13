# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Structure

Đây là một monorepo chứa 3 dự án độc lập:

- **riz-be**: Backend NestJS với Turborepo, deployed trên AWS Lambda
- **riz-admin-fe**: Admin web application (React + Vite)
- **riz-app-v2**: React Native mobile app với Expo

Mỗi dự án có package manager riêng và có thể develop độc lập.

## Local Development Setup

### Docker Services (required)

```bash
cd riz-be/apps/nest
docker-compose up -d    # Start PostgreSQL + Redis
docker-compose down     # Stop containers
```

| Container  | Port             | Mô tả         |
| ---------- | ---------------- | ------------- |
| `postgres` | `localhost:5403` | PostgreSQL 17 |
| `redis`    | `localhost:6849` | Redis Stack   |

### Service URLs

| URL                          | Mô tả        |
| ---------------------------- | ------------ |
| `http://localhost:4000`      | Backend API  |
| `http://localhost:4000/docs` | Swagger Docs |
| `http://localhost:5173`      | Admin Panel  |

## Common Commands

### Backend (riz-be)

```bash
cd riz-be
pnpm install           # Install dependencies
pnpm dev              # Start development (via turbo -> nest start:local:watch)
pnpm build            # Build all apps (via turbo)
pnpm lint             # Run linting
pnpm format           # Format code with prettier

# Development (inside apps/nest)
cd apps/nest
pnpm start:local:watch        # Start dev server with hot reload (.env.local)
pnpm start:local:debug        # Start with debugger

# Database (inside apps/nest, requires .env.local)
npx prisma migrate dev        # Run migrations
npx prisma generate          # Generate Prisma client
npx prisma studio           # Open Prisma Studio
npx prisma db seed         # Seed database
pnpm prisma:reset:local      # Reset DB + reseed (destructive)

# Testing (inside apps/nest)
pnpm test                    # Run all tests (.env.spec)
pnpm test -- --testPathPattern="todo"  # Run single test file
pnpm test -- --testNamePattern="Create" # Run tests matching name
pnpm test:e2e               # Run e2e tests
pnpm test:watch              # Watch mode
```

**Environment Files** (trong `apps/nest/`):

- `.env.local`: Local development
- `.env.spec`: Testing environment
- `.env.dev`: Development/staging

**QUAN TRỌNG**: KHÔNG BAO GIỜ đọc file `.env` — chúng là sensitive và gitignored. Sử dụng `.env.example` để tham khảo schema.

### Admin Frontend (riz-admin-fe)

```bash
cd riz-admin-fe
bun install           # Install dependencies
bun dev              # Start dev server on port 5173
bun build            # Build for production
bun serve            # Preview production build
bun test             # Run tests with Vitest
bun lint             # Run ESLint
bun format           # Format with Prettier
bun check            # Format + lint with fixes

# Shadcn components
pnpx shadcn@latest add button  # Add UI components
```

### Mobile App (riz-app-v2)

```bash
cd riz-app-v2
bun install                    # Install dependencies
bun dev                       # Start Expo dev server
bun android                   # Run on Android emulator
bun ios                      # Run on iOS simulator (Mac only)
bun web                      # Run in browser

# EAS Build
bun eas-android             # Build preview for Android
bun eas-ios                 # Build preview for iOS
bun eas-update             # Push OTA update

# Native development
bun development-local-android  # expo run:android
bun development-local-ios      # expo run:ios
```

## Backend Architecture (riz-be)

### Monorepo Structure

Sử dụng **Turborepo** để quản lý monorepo với 2 apps:

- `apps/nest`: NestJS application với domain libraries
- `apps/cdk`: AWS CDK infrastructure (S3, Lambda, API Gateway)

### NestJS Domain Libraries

Backend được tổ chức theo modular architecture với các domain libraries trong `apps/nest/libs/`:

- `auth`: Authentication & authorization (Passport, JWT, Firebase, OAuth)
- `user`: User management
- `profile`: User profiles (multi-tenant design - users share profiles)
- `project`: Project management
- `post`: Post/content management
- `comment`: Comment system
- `reaction`: Reactions/likes
- `bookmark`: Bookmark functionality
- `notification`: Notification system
- `follow`: Follow/unfollow system
- `spatial-assets`: 3D/spatial assets
- `storage`: File storage (AWS S3)
- `mailer`: Email service (SendGrid)
- `core`: Shared utilities, decorators, filters, interceptors, guards
- `helper`: Utility functions
- `spec`: Testing utilities

### Tech Stack

- **Framework**: NestJS v10 với TypeScript 5.5.4
- **Database**: PostgreSQL với Prisma ORM v6.2.1
- **Caching**: Redis (ioredis + cache-manager)
- **Authentication**: Passport (JWT, Firebase, Apple, Google OAuth)
- **Cloud**: AWS (Lambda, S3, API Gateway) via AWS CDK
- **Deployment**: Dual-mode (local Express + AWS Lambda với @vendia/serverless-express)

### Multi-tenant Design Pattern

**QUAN TRỌNG**: Hệ thống được thiết kế để nhiều users có thể connect và share cùng một profile. Hầu hết các business models sẽ relate tới `profile` thay vì `user`.

@.claude/conventions/backend-patterns.md

### Testing Conventions

@.claude/conventions/testing-patterns.md

## Admin Frontend Architecture (riz-admin-fe)

@.claude/conventions/admin-fe-patterns.md

## Mobile App Architecture (riz-app-v2)

@.claude/conventions/app-patterns.md

## Development Guidelines

### TypeScript

- Luôn declare type cho variables và functions
- Avoid `any` - tạo types cần thiết
- Use JSDoc cho public classes/methods
- One export per file
- Không để blank lines trong function

### Code Style

- Use English cho tất cả code và documentation
- Follow existing patterns trong codebase
- Prefer modular architecture
- Keep components focused và single-responsibility

### Exception Handling

- Use exceptions cho unexpected errors
- Khi catch exception, chỉ để: fix expected problem, add context, hoặc dùng global handler
- Backend có global filters trong `libs/core/` cho exception handling

### Git Workflow

Tất cả 3 projects đều sử dụng:

- **Husky** cho git hooks
- **Commitlint** với conventional commits
- **Prettier** + **ESLint** integration
- Lint-staged cho pre-commit hooks

### Before Completing Coding Tasks

**QUAN TRỌNG**: Sau khi hoàn thành coding (tạo/sửa file `.ts`, `.tsx`, `.js`, `.jsx`), PHẢI chạy lint fix trước khi kết thúc response:

```bash
# Cho riz-admin-fe
cd riz-admin-fe && bunx eslint --fix <files>

# Cho riz-app-v2  
cd riz-app-v2 && bunx eslint --fix <files>

# Cho riz-be
cd riz-be && npx eslint --fix <files>
```

Chỉ chạy lint trên các files đã thay đổi trong task hiện tại. Review kết quả và fix nếu có lỗi còn lại.

## Cyberk Flow Workflow

Khi implement hoặc resume một change:

1. Kiểm tra `cyberk-flow/changes/` cho existing change directories (`bun run cf changes`)
2. Nếu có matching change, đọc `workflow.md` để xác định current gate/state
3. Resume từ correct gate — `workflow.md` trên disk là source of truth, KHÔNG phải conversation history

<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **riz** (476914 symbols, 711980 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## When Debugging

1. `gitnexus_query({query: "<error or symptom>"})` — find execution flows related to the issue
2. `gitnexus_context({name: "<suspect function>"})` — see all callers, callees, and process participation
3. `READ gitnexus://repo/riz/process/{processName}` — trace the full execution flow step by step
4. For regressions: `gitnexus_detect_changes({scope: "compare", base_ref: "main"})` — see what your branch changed

## When Refactoring

- **Renaming**: MUST use `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` first. Review the preview — graph edits are safe, text_search edits need manual review. Then run with `dry_run: false`.
- **Extracting/Splitting**: MUST run `gitnexus_context({name: "target"})` to see all incoming/outgoing refs, then `gitnexus_impact({target: "target", direction: "upstream"})` to find all external callers before moving code.
- After any refactor: run `gitnexus_detect_changes({scope: "all"})` to verify only expected files changed.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Tools Quick Reference

| Tool | When to use | Command |
|------|-------------|---------|
| `query` | Find code by concept | `gitnexus_query({query: "auth validation"})` |
| `context` | 360-degree view of one symbol | `gitnexus_context({name: "validateUser"})` |
| `impact` | Blast radius before editing | `gitnexus_impact({target: "X", direction: "upstream"})` |
| `detect_changes` | Pre-commit scope check | `gitnexus_detect_changes({scope: "staged"})` |
| `rename` | Safe multi-file rename | `gitnexus_rename({symbol_name: "old", new_name: "new", dry_run: true})` |
| `cypher` | Custom graph queries | `gitnexus_cypher({query: "MATCH ..."})` |

## Impact Risk Levels

| Depth | Meaning | Action |
|-------|---------|--------|
| d=1 | WILL BREAK — direct callers/importers | MUST update these |
| d=2 | LIKELY AFFECTED — indirect deps | Should test |
| d=3 | MAY NEED TESTING — transitive | Test if critical path |

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/riz/context` | Codebase overview, check index freshness |
| `gitnexus://repo/riz/clusters` | All functional areas |
| `gitnexus://repo/riz/processes` | All execution flows |
| `gitnexus://repo/riz/process/{name}` | Step-by-step execution trace |

## Self-Check Before Finishing

Before completing any code modification task, verify:
1. `gitnexus_impact` was run for all modified symbols
2. No HIGH/CRITICAL risk warnings were ignored
3. `gitnexus_detect_changes()` confirms changes match expected scope
4. All d=1 (WILL BREAK) dependents were updated

## Keeping the Index Fresh

After committing code changes, the GitNexus index becomes stale. Re-run analyze to update it:

```bash
npx gitnexus analyze
```

If the index previously included embeddings, preserve them by adding `--embeddings`:

```bash
npx gitnexus analyze --embeddings
```

To check whether embeddings exist, inspect `.gitnexus/meta.json` — the `stats.embeddings` field shows the count (0 means no embeddings). **Running analyze without `--embeddings` will delete any previously generated embeddings.**

> Claude Code users: A PostToolUse hook handles this automatically after `git commit` and `git merge`.

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->
