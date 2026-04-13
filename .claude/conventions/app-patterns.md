# Mobile App Architecture (riz-app-v2)

## Tech Stack

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

## Navigation Structure

File-based routing với Expo Router trong `app/`:

- `_layout.tsx`: Root layout với providers (GestureHandler, QueryProvider, ThemeProvider)
- `(protected)/`: Protected routes với authentication guard
- `onboarding/`: Onboarding flow
- `profile/`: Profile routes
- `settings/`: Settings routes

Protected routes redirect to `/login` nếu chưa authenticated.

## Key Features

1. **Feed System**: Masonry grid layout với category filtering
2. **Community/Social**: Post creation, comments, reactions, bookmarks
3. **Creator Studio**: Project management dashboard
4. **Project Creation**: 2D và 3D project creation (Three.js integration)
5. **Profile System**: User profiles với tabs (Works, Saved, Services, About)
6. **Onboarding**: Email + OTP authentication flow
7. **Media Management**: Camera roll, image cropping, video upload

## Folder Structure

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
