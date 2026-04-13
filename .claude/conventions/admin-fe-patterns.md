# Admin Frontend Architecture (riz-admin-fe)

## Tech Stack

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

## Routing Structure

File-based routing với TanStack Router trong `src/routes/`:

- `__root.tsx`: Root layout với NavigationProgress, Toaster, Devtools
- `_authenticated/`: Layout route cho protected pages
- `(auth)/`: Auth route group (sign-in, OAuth callbacks)
- `(errors)/`: Error pages (401, 403, 404, 500, 503)

Routes tự động generate vào `routeTree.gen.ts` bởi Vite plugin.

## State Management

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

## Folder Structure

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
