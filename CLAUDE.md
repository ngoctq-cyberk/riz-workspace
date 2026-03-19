# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Monorepo Structure

Đây là một monorepo chứa 3 dự án độc lập:

- **riz-be**: Backend NestJS với Turborepo, deployed trên AWS Lambda
- **riz-admin-fe**: Admin web application (React + Vite)
- **riz-app-v2**: React Native mobile app với Expo

Mỗi dự án có package manager riêng và có thể develop độc lập.

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
pnpm test:e2e               # Run e2e tests
```

**Environment Files** (trong `apps/nest/`):

- `.env.local`: Local development
- `.env.spec`: Testing environment
- `.env.dev`: Development/staging

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

### Prisma Schema Convention

```typescript
// Tất cả entities phải có:
id         BigInt   @id @default(autoincrement())
createdAt  DateTime @default(now())
updatedAt  DateTime @updatedAt
profileId  BigInt?  // Optional, related to profile
```

### Entity Structure

Entities nằm trong `libs/*/entities/*.entity.ts`:

```typescript
import { Expose, Type } from "class-transformer";
import { ApiProperty } from "@nestjs/swagger";

export class TodoEntity {
  @ApiProperty()
  @Expose()
  id: bigint;

  @ApiProperty()
  @Expose()
  createdAt: Date;

  @ApiProperty()
  @Expose()
  updatedAt: Date;

  @ApiProperty()
  @Expose()
  title: string;

  @ApiProperty()
  @Expose()
  profileId: bigint;

  // Relations section (at bottom)
  @ApiProperty({ type: () => ProfileEntity })
  @Type(() => ProfileEntity)
  @Expose()
  profile: ProfileEntity;
}
```

### DTO Conventions

**Create DTO** (`create-*.dto.ts`):

- Không cần `profileId` field (sẽ lấy từ user)
- Có validation decorators (class-validator)
- Có ApiProperty decorators

**Update DTO** (`edit-*.dto.ts`):

- Sử dụng `PickType` để pick fields từ CreateDTO
- Sử dụng `PartialType` để make optional

```typescript
import { PartialType, PickType } from "@nestjs/swagger";
import { CreateTodoDto } from "./create-todo.dto";

class _UpdateTodoDto extends PickType(CreateTodoDto, [
  "title",
  "description",
]) {}
export class UpdateTodoDto extends PartialType(_UpdateTodoDto) {}
```

**Query DTO** (`query-*.dto.ts`):

- Hỗ trợ where, sort, select, include, skip, take
- Number fields phải có `@Type(() => Number)`

### Service Layer

Services phải include `user` parameter trong create/update/delete methods:

```typescript
async createTodo(dto: CreateTodoDto, user: User) {
  const todo = await this.prisma.todo.create({
    data: {
      ...dto,
      profileId: user.profileId, // Attach to user's profile
    },
  })
  return th.toInstanceSafe(TodoEntity, todo)
}

async updateTodo(id: bigint, dto: UpdateTodoDto, user: User) {
  return await this.prisma.todo.update({
    where: { id, profileId: user.profileId }, // Security: only update own data
    data: dto,
  })
}
```

### Testing Conventions

- Sử dụng Jest framework
- Follow Arrange-Act-Assert pattern
- Naming: `inputX`, `mockX`, `actualX`, `expectedX`
- Write unit tests for public functions
- Write e2e tests for each API module
- Test file pattern: `*.spec.ts`

```bash
# Chạy từ riz-be/apps/nest
pnpm test                              # Chạy tất cả tests
pnpm test -- --testPathPattern="todo"  # Chạy tests có "todo" trong path
pnpm test -- --testNamePattern="Create" # Chạy tests có "Create" trong tên
pnpm test:watch                        # Watch mode
```

## Admin Frontend Architecture (riz-admin-fe)

### Tech Stack

- **Framework**: React 19 với TypeScript
- **Build Tool**: Vite 6
- **Package Manager**: Bun
- **Routing**: TanStack Router (file-based routing)
- **State Management**:
  - TanStack Query (server state với IndexedDB persistence)
  - Zustand (client state với localStorage persistence)
- **Styling**: Tailwind CSS v4
- **UI Components**: Radix UI + Shadcn/ui
- **Forms**: React Hook Form + Zod validation
- **HTTP Client**: Axios với token refresh interceptors
- **i18n**: React Intl

### Routing Structure

File-based routing với TanStack Router trong `src/routes/`:

- `__root.tsx`: Root layout với NavigationProgress, Toaster, Devtools
- `_authenticated/`: Layout route cho protected pages
- `(auth)/`: Auth route group (sign-in, OAuth callbacks)
- `(errors)/`: Error pages (401, 403, 404, 500, 503)

Routes tự động generate vào `routeTree.gen.ts` bởi Vite plugin.

### State Management

**Zustand (Client State)**:

- `auth.store.ts`: Quản lý accessToken/refreshToken, persist to localStorage
- Custom pattern với `createControlledStore` wrapper
- Hooks: `useIsAuthenticated`, `useAccessToken`, `useRefreshToken`

**TanStack Query (Server State)**:

- Query caching với IndexedDB persistence (via idb-keyval)
- Global error handling trong query cache
- Auto retry (disabled cho 401/403)
- 10s default stale time

**Token Refresh Service**:

- Proactive token refresh (check expiration trước requests)
- Axios interceptors cho automatic retry on 401
- Integrated với Zustand auth store

### Folder Structure

```
src/
├── components/          # Reusable UI components
│   ├── ui/             # Shadcn UI components
│   └── layout/         # Layout components (AuthenticatedLayout, AppSidebar)
├── features/           # Feature-based modules (domain-driven)
│   ├── auth/
│   ├── dashboard/
│   └── errors/
├── routes/            # File-based routing
├── stores/            # Zustand stores
├── integrations/      # Third-party integrations (TanStack Query, React Intl)
├── lib/               # Shared libraries (axios, utils)
├── context/           # React Context (theme, search)
└── hooks/             # Custom React hooks
```

## Mobile App Architecture (riz-app-v2)

### Tech Stack

- **Framework**: React Native với Expo ~54.0
- **Routing**: Expo Router (file-based)
- **Styling**: TailwindCSS + NativeWind v4
- **State Management**: Zustand + TanStack Query
- **UI**: React Native Reusables, Lucide icons
- **Forms**: React Hook Form + Zod
- **Animations**: React Native Reanimated ~4.1
- **3D Graphics**: Three.js + React Three Fiber + React Three Drei
- **Media**: Expo Image, Expo Video, Shopify Skia
- **Storage**: MMKV + Expo Secure Store
- **Package Manager**: Bun v1.3.5

### Navigation Structure

File-based routing với Expo Router trong `app/`:

- `_layout.tsx`: Root layout với providers (GestureHandler, QueryProvider, ThemeProvider)
- `(protected)/`: Protected routes với authentication guard
- `onboarding/`: Onboarding flow
- `profile/`: Profile routes
- `settings/`: Settings routes

Protected routes redirect to `/login` nếu chưa authenticated.

### Key Features

1. **Feed System**: Masonry grid layout với category filtering
2. **Community/Social**: Post creation, comments, reactions, bookmarks
3. **Creator Studio**: Project management dashboard
4. **Project Creation**: 2D và 3D project creation (Three.js integration)
5. **Profile System**: User profiles với tabs (Works, Saved, Services, About)
6. **Onboarding**: Email + OTP authentication flow
7. **Media Management**: Camera roll, image cropping, video upload

### Folder Structure

```
app/                    # Expo Router routes
screens/               # Screen implementations
components/            # Reusable UI components
lib/                   # Core libraries (api, storage, constants, 3d)
hooks/                 # Custom React hooks
services/              # Business logic services
store/                 # Zustand stores
types/                 # TypeScript types
utils/                 # Utility functions
integrations/          # Third-party integrations
```

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

### Git Workflow

Tất cả 3 projects đều sử dụng:

- **Husky** cho git hooks
- **Commitlint** với conventional commits
- **Prettier** + **ESLint** integration
- Lint-staged cho pre-commit hooks
