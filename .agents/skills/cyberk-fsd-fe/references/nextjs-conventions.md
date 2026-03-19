# Next.js Project Conventions

## Route Structure
- **Auth**: `app/(auth)/login`, `app/(auth)/signup`
- **Dashboard**: `app/(dashboard)/` (Protected layout)
- **API**: `app/api/` (Route Handlers)

## Special Files
- `page.tsx`: Route entry point.
- `layout.tsx`: Persistent wrapper (navbars, sidebars).
- `loading.tsx`: Suspense boundary fallback.
- `error.tsx`: Error boundary.

## Client vs Server
- **Default**: Server Components (RSC) everywhere.
- **Client**: Add `"use client"` ONLY when you need:
    - Event listeners (`onClick`, `onChange`)
    - Hooks (`useState`, `useEffect`)
    - Browser APIs (`localStorage`)

## Data Fetching
- **Server**: Fetch directly using `orpc` client.
- **Client**: Use `useQuery` / `useMutation` hooks from `entities` or `features`.
