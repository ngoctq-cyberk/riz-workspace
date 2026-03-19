# Force Update & Maintenance Mode - Implementation Summary

## Tổng quan

Đã implement thành công tính năng **Force Update & Maintenance Mode** theo plan chi tiết trong [PLAN-force-update.md](./PLAN-force-update.md). Tính năng cho phép admin kiểm soát việc cập nhật app và bật chế độ bảo trì từ server.

## ✅ Completed Tasks

### Phase 1: Backend - App Config Module

**1.1 DTOs và Enums** ✅
- ✅ `dto/update-level.enum.ts` - 4 update levels (NONE, OPTIONAL, RECOMMENDED, REQUIRED)
- ✅ `dto/app-config.response.dto.ts` - DTOs cho iOS, Android, Maintenance

**1.2 AppConfigService** ✅
- ✅ `app-config.service.ts` - Service với in-memory cache (60s TTL)
- ✅ `getConfig()` - Lấy config từ cache hoặc DB
- ✅ `updateConfig()` - Update config và invalidate cache
- ✅ Default config khi chưa có data

**1.3 Public API Endpoint** ✅
- ✅ `app-config.controller.ts` - Public endpoint `GET /app-config`
- ✅ Sử dụng `JwtOptionalGuard` (no auth required)
- ✅ Swagger documentation

**1.4 Settings Module Updates** ✅
- ✅ Update `settings.module.ts` - Thêm controller và service
- ✅ Update `index.ts` - Export types mới

**1.5 Seed Default Config** ✅
- ✅ Thêm `seedAppVersionControl()` vào `prisma/seed.ts`
- ✅ Default config với iOS 0.2.1, Android 1.1.5
- ✅ Update level = "none" (không show dialog)

### Phase 2: Mobile - Version Check System

**2.1 Dependencies** ✅
- ✅ Install `semver` ^7.7.3
- ✅ Install `@types/semver` ^7.7.1

**2.2 API Client** ✅
- ✅ `lib/api/app-config.ts` - API client với types
- ✅ `getAppConfig()` function

**2.3 Version Utilities** ✅
- ✅ `components/app-update/version-utils.ts`
- ✅ `isVersionOutdated()` - Semver comparison
- ✅ `getCurrentAppVersion()` - Get từ Expo Constants

**2.4 Update Storage (MMKV)** ✅
- ✅ `components/app-update/update-storage.ts`
- ✅ `getLastDismissedVersion()` - Lấy version đã dismiss
- ✅ `setLastDismissedVersion()` - Lưu version đã dismiss
- ✅ `shouldShowOptionalUpdate()` - Logic hiển thị optional update

**2.5 React Query Hooks** ✅
- ✅ `hooks/use-app-config.ts` - Hook với caching strategy (5min staleTime)
- ✅ `hooks/use-foreground-config-check.ts` - Foreground re-check (debounced 5min)
- ✅ Silent error handling (không block app)

**2.6 Modal Components** ✅
- ✅ `components/app-update/ForceUpdateModal.tsx` - Required level (blocking)
- ✅ `components/app-update/SoftUpdateModal.tsx` - Optional/Recommended (dismissible)
- ✅ `components/app-update/MaintenanceModal.tsx` - Maintenance mode (blocking)

**2.7 Root Layout Integration** ✅
- ✅ Update `app/_layout.tsx` với version check logic
- ✅ Maintenance priority (highest)
- ✅ Version comparison với semver
- ✅ Dialog state management
- ✅ Render 3 modals at root level

**2.8 i18n Translations** ✅
- ✅ Thêm 8 translation keys vào `en.json`
- ✅ Thêm 8 translation keys vào `vi.json`
- ✅ Update modal components để sử dụng flat keys

### Phase 3: Admin API

**3.1 Admin API Endpoint** ✅
- ✅ `admin-app-config.controller.ts` - Admin-only endpoints
- ✅ `GET /admin/app-config` - Get config (admin view)
- ✅ `PUT /admin/app-config` - Update config
- ✅ Role-based access control (ADMIN, SUPERADMIN)

## 📁 Files Created/Modified

### Backend (riz-be)

**Created:**
```
riz-be/apps/nest/libs/settings/src/
├── dto/
│   ├── update-level.enum.ts          # NEW
│   └── app-config.response.dto.ts    # NEW
├── app-config.service.ts              # NEW
├── app-config.controller.ts           # NEW
└── admin-app-config.controller.ts     # NEW
```

**Modified:**
```
riz-be/apps/nest/libs/settings/src/
├── settings.module.ts                 # UPDATED: Add controllers & services
├── index.ts                           # UPDATED: Export new types
└── prisma/seed.ts                     # UPDATED: Add app_version_control seed
```

### Mobile (riz-app-v2)

**Created:**
```
riz-app-v2/
├── lib/api/
│   └── app-config.ts                  # NEW: API client
├── hooks/
│   ├── use-app-config.ts              # NEW: React Query hook
│   └── use-foreground-config-check.ts # NEW: Foreground check hook
└── components/app-update/             # NEW folder
    ├── AppUpdateProvider.tsx          # NEW: Wrapper provider component
    ├── version-utils.ts
    ├── update-storage.ts
    ├── ForceUpdateModal.tsx
    ├── SoftUpdateModal.tsx
    ├── MaintenanceModal.tsx
    └── index.ts                       # Barrel export
```

**Modified:**
```
riz-app-v2/
├── app/_layout.tsx                    # UPDATED: Wrap with AppUpdateProvider
├── integrations/react-intl/locales/
│   ├── en.json                        # UPDATED: Add 8 keys
│   └── vi.json                        # UPDATED: Add 8 keys
└── package.json                       # UPDATED: Add semver deps
```

## 🚀 API Endpoints

### Public Endpoints

```
GET /app-config
```
**Response:**
```json
{
  "ios": {
    "storeVersion": "1.2.0",
    "updateLevel": "optional",
    "storeUrl": "https://apps.apple.com/app/id123456789"
  },
  "android": {
    "storeVersion": "1.2.0",
    "updateLevel": "optional",
    "storeUrl": "https://play.google.com/store/apps/details?id=com.rizfe"
  },
  "maintenance": {
    "enabled": false,
    "message": ""
  }
}
```

### Admin Endpoints (Requires Auth)

```
GET  /admin/app-config       # Get current config
PUT  /admin/app-config       # Update config
```

**Auth:** Bearer token với role ADMIN hoặc SUPERADMIN

## 🎯 Update Level Behaviors

| Level | Dismissible? | Show When? | Persistence |
|-------|-------------|-----------|-------------|
| `none` | N/A | Never | N/A |
| `optional` | ✅ Yes | Once/version | MMKV (lastDismissed) |
| `recommended` | ✅ Yes | Every start | Memory only |
| `required` | ❌ No | Always | N/A (blocking) |

## 🔥 Priority Order

1. **Maintenance Mode** (highest) - Blocks app, shows only maintenance dialog
2. **Force Update** (required) - Blocks app if version outdated
3. **Recommended Update** - Shows every cold start, dismissible
4. **Optional Update** - Shows once per version, dismissible

## 🧪 Testing Guide

### 1. Backend Testing

```bash
# Build backend
cd riz-be/apps/nest
pnpm build

# Run seed (nếu chưa có data)
npx prisma db seed

# Start dev server
pnpm start:local:watch

# Test public endpoint (không cần auth)
curl http://localhost:3000/api/v1/app-config

# Test admin endpoint (cần auth token)
curl -H "Authorization: Bearer $ADMIN_TOKEN" \
  http://localhost:3000/api/v1/admin/app-config
```

### 2. Mobile Testing

```bash
# Build và chạy app
cd riz-app-v2
bun dev

# Run trên Android/iOS
bun android
bun ios
```

### 3. Test Cases

| Test Case | Steps | Expected Result |
|-----------|-------|----------------|
| **Maintenance ON** | 1. Set `maintenance.enabled: true` via admin API<br>2. Kill & reopen app | MaintenanceModal shows (blocking) |
| **Required Update** | 1. Set `updateLevel: "required"`, `storeVersion: "99.0.0"`<br>2. Kill & reopen app | ForceUpdateModal shows (cannot dismiss) |
| **Recommended Update** | 1. Set `updateLevel: "recommended"`, `storeVersion: "99.0.0"`<br>2. Kill & reopen app<br>3. Dismiss modal<br>4. Kill & reopen app | Modal shows → dismiss → modal shows again |
| **Optional Update** | 1. Set `updateLevel: "optional"`, `storeVersion: "99.0.0"`<br>2. Kill & reopen app<br>3. Dismiss modal<br>4. Kill & reopen app | Modal shows → dismiss → no modal on reopen |
| **Foreground Check** | 1. Open app<br>2. Background for 6+ minutes<br>3. Admin enables maintenance<br>4. Return to app | MaintenanceModal appears |
| **API Failure** | 1. Stop backend server<br>2. Open app | App works normally (silent fail) |

### 4. Admin API Testing với cURL

**Update config để test Required update:**
```bash
curl -X PUT \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "ios": {
      "storeVersion": "99.0.0",
      "updateLevel": "required",
      "storeUrl": "https://apps.apple.com/app/id123456789"
    },
    "android": {
      "storeVersion": "99.0.0",
      "updateLevel": "required",
      "storeUrl": "https://play.google.com/store/apps/details?id=com.rizfe"
    },
    "maintenance": {
      "enabled": false,
      "message": ""
    }
  }' \
  http://localhost:3000/api/v1/admin/app-config
```

**Enable maintenance mode:**
```bash
curl -X PUT \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "ios": {
      "storeVersion": "0.2.1",
      "updateLevel": "none",
      "storeUrl": "https://apps.apple.com/app/id123456789"
    },
    "android": {
      "storeVersion": "1.1.5",
      "updateLevel": "none",
      "storeUrl": "https://play.google.com/store/apps/details?id=com.rizfe"
    },
    "maintenance": {
      "enabled": true,
      "message": "Hệ thống đang bảo trì. Dự kiến hoàn thành lúc 14:00."
    }
  }' \
  http://localhost:3000/api/v1/admin/app-config
```

## 📊 Caching Strategy

### Backend Cache
- **Type:** In-memory cache
- **TTL:** 60 seconds
- **Invalidation:** Immediate khi admin update config
- **Purpose:** Reduce DB queries

### Mobile Cache
- **Type:** React Query cache
- **staleTime:** 5 minutes (matches foreground check interval)
- **gcTime:** 10 minutes
- **Retry:** false (fail silently)
- **Refetch:** Only on foreground after 5+ minutes

## 🔒 Security Features

### Backend
- Public endpoint: `JwtOptionalGuard` (no auth required)
- Admin endpoint: `JwtGuard` + `@Roles('ADMIN', 'SUPERADMIN')`
- Config updates tracked với `updatedById`

### Mobile
- Silent error handling (không expose errors to users)
- Fail-safe approach (invalid versions → don't block user)
- Version validation với semver library

## 📝 Translation Keys

### English (en.json)
```
update_title
update_message
update_button_updateNow
update_button_later
update_error_cannotOpenStore
update_error_failedToOpenStore
maintenance_title
maintenance_defaultMessage
```

### Vietnamese (vi.json)
Same keys với Vietnamese translations.

## 🎨 UI/UX

### ForceUpdateModal (Required)
- Full-screen modal
- App logo + title + message
- Single "Update Now" button
- Blocks Android back button
- Cannot be dismissed

### SoftUpdateModal (Optional/Recommended)
- Bottom sheet style
- App logo + title + message
- "Update Now" + "Later" buttons
- Can be dismissed (tap backdrop or "Later")
- For optional: Persists to MMKV
- For recommended: Session-only

### MaintenanceModal
- Full-screen modal
- Maintenance icon (🔧) + title + message
- No action buttons (user must close app)
- Blocks Android back button
- Shows server message or default i18n message

## 🔄 Deployment Checklist

### Backend
- [ ] Run `pnpm build` - verify no errors
- [ ] Run `pnpm lint` - verify no warnings
- [ ] Run `npx prisma db seed` - seed app_version_control
- [ ] Test public endpoint `/app-config`
- [ ] Test admin endpoints với valid auth token
- [ ] Deploy to AWS Lambda
- [ ] Verify endpoint accessible from mobile app

### Mobile
- [ ] Run `npx tsc --noEmit` - verify no TypeScript errors
- [ ] Run `bun lint` - verify no ESLint warnings
- [ ] Test all 3 modals (required, optional, maintenance)
- [ ] Test foreground check (background 5+ min)
- [ ] Test API failure scenario (silent fail)
- [ ] Update Store URLs với real App Store/Play Store links
- [ ] Build và submit to stores

## 🚨 Rollback Strategy

| Issue | Rollback Action |
|-------|----------------|
| Backend deployment fails | Revert to previous container |
| Mobile crashes on launch | Set `updateLevel: "none"` in database |
| Wrong Store URL | Update `app_version_control` setting |
| Users stuck in update loop | Set `updateLevel: "optional"` so they can dismiss |

## 📖 References

- Plan Document: [PLAN-force-update.md](./PLAN-force-update.md)
- Backend Architecture: [CLAUDE.md](../CLAUDE.md) - Backend section
- Mobile Architecture: [CLAUDE.md](../CLAUDE.md) - Mobile section

## ✅ Implementation Complete

Tất cả phases đã được implement thành công:
- ✅ Phase 1: Backend - App Config Module
- ✅ Phase 2: Mobile - Version Check System
- ✅ Phase 3: Admin API

Ready for testing và deployment! 🚀
