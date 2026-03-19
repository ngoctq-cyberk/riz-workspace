# PLAN: Force Update & Maintenance Mode Feature

## Overview

Implement a **Tiered Update System** and **Maintenance Mode** that allows admins to control app access via server-side configuration with 4 levels of update prompts:

| Level         | Behavior                                        |
| ------------- | ----------------------------------------------- |
| `none`        | Không show gì                                   |
| `optional`    | Soft dialog, dễ dismiss, show 1 lần/version     |
| `recommended` | Soft dialog, nhấn mạnh hơn, show mỗi lần mở app |
| `required`    | Force dialog, không tắt được                    |

**Project Type:** MOBILE + BACKEND (Full-stack feature)

---

## Success Criteria

- [ ] Admin can set iOS/Android version requirements via Backend API
- [ ] Admin can set update level: `none`, `optional`, `recommended`, `required`
- [ ] Admin can enable maintenance mode with custom message
- [ ] Mobile app checks version on startup (silent, non-blocking API call)
- [ ] **Maintenance mode:** Blocking dialog, cannot be dismissed, no actions (user must close and reopen app later)
- [ ] **Maintenance priority:** If maintenance ON, ignore all update levels (show only maintenance)
- [ ] **Required level:** Blocking dialog, cannot be dismissed without updating
- [ ] **Optional/Recommended level:** Dialog can be dismissed with "Later" button
- [ ] **Optional level:** Only shows once per new version (persisted in MMKV)
- [ ] **Recommended level:** Shows every cold start
- [ ] **Global display:** Dialog shows on all screens, persists during navigation
- [ ] Tapping "Update Now" opens correct Store with error handling
- [ ] API failure: App continues normally (fail silently, no retry)
- [ ] Backend uses in-memory cache (60s TTL) to reduce DB queries

---

## Tech Stack

| Layer   | Technology          | Notes                                                       |
| ------- | ------------------- | ----------------------------------------------------------- |
| Backend | NestJS + Prisma     | Existing `AppSettings` table with in-memory cache (60s TTL) |
| Mobile  | Expo (React Native) | `expo-constants` for version, `expo-linking` for Store URLs |
| Storage | MMKV                | Persist dismissed version for `optional` level              |
| API     | REST                | New `/app-config` endpoint (public, no auth, silent call)   |

---

## Schema Design

### AppSettings Entry

```json
// Key: "app_version_control"
// ValueType: "json"
// Group: "mobile"
{
  "ios": {
    "storeVersion": "1.2.0",
    "updateLevel": "optional", // "none" | "optional" | "recommended" | "required"
    "storeUrl": "https://apps.apple.com/app/id123456789"
    // NOTE: No updateMessage - use i18n text in app for multi-language support
  },
  "android": {
    "storeVersion": "1.2.0",
    "updateLevel": "optional",
    "storeUrl": "https://play.google.com/store/apps/details?id=com.rizfe"
    // NOTE: No updateMessage - use i18n text in app for multi-language support
  },
  "maintenance": {
    "enabled": false,
    "message": "Hệ thống đang bảo trì. Dự kiến hoàn thành lúc 14:00." // Keep: admin-specific message, can include time/details
  }
}
```

### Update Level Behaviors

```
┌─────────────────────────────────────────────────────────────────────┐
│                        UPDATE LEVEL MATRIX                          │
├──────────────┬───────────────┬───────────────┬─────────────────────┤
│ Level        │ Dismissible?  │ Show When?    │ Persistence         │
├──────────────┼───────────────┼───────────────┼─────────────────────┤
│ none         │ N/A           │ Never         │ N/A                 │
│ optional     │ ✅ Yes        │ Once/version  │ MMKV (lastDismissed)│
│ recommended  │ ✅ Yes        │ Every start   │ Memory only         │
│ required     │ ❌ No         │ Always        │ N/A (blocking)      │
└──────────────┴───────────────┴───────────────┴─────────────────────┘
```

---

## File Structure

```
riz-be/
├── apps/nest/libs/
│   └── settings/                              # EXTEND EXISTING MODULE
│       └── src/
│           ├── index.ts                       # UPDATE: Export new services/DTOs
│           ├── settings.module.ts             # UPDATE: Add AppConfigController
│           ├── settings.service.ts            # EXISTING (unchanged)
│           ├── app-config.service.ts          # NEW: App version & maintenance logic
│           ├── app-config.controller.ts       # NEW: Public API endpoint
│           └── dto/
│               ├── app-config.response.dto.ts # NEW: Response DTO
│               └── update-level.enum.ts       # NEW: Enum definition

riz-app-v2/
├── lib/
│   └── api/
│       └── app-config.ts                      # NEW: API client
├── hooks/
│   ├── use-app-config.ts                      # NEW: React Query hook
│   └── use-foreground-config-check.ts         # NEW: Foreground re-check hook
├── components/
│   └── app-update/                            # NEW folder
│       ├── ForceUpdateModal.tsx               # Blocking modal (required)
│       ├── SoftUpdateModal.tsx                # Dismissible modal (optional/recommended)
│       ├── MaintenanceModal.tsx               # Maintenance message
│       ├── version-utils.ts                   # Version comparison logic
│       └── update-storage.ts                  # MMKV persistence
└── app/
    └── _layout.tsx                            # MODIFY: Add version check + foreground check
```

---

## Task Breakdown

### Phase 0: Codebase Exploration Results ✅

**Status:** COMPLETED - All assumptions verified against actual codebase

---

#### Backend Findings

**✅ SettingsService API (VERIFIED):**
- Location: `riz-be/apps/nest/libs/settings/src/settings.service.ts`
- `get(key: string): Promise<string | null>` ✅
- `set(key: string, value: string, group?: string, userId?: bigint): Promise<void>` ✅
- Module is `@Global()` - available everywhere without imports
- **Note:** Service returns raw Prisma results, does NOT use `th.toInstanceSafe()`

**✅ Prisma AppSettings Schema (VERIFIED):**
- Location: `riz-be/apps/nest/prisma/schema.prisma` (lines 590-605)
- Fields: `id`, `key` (unique), `value` (String), `valueType`, `group`, `updatedById`, timestamps ✅
- **Warning:** `value` is `String` not `Text` (may have varchar limits, but sufficient for JSON config)

**⚠️ Auth Pattern (DIFFERENT FROM ASSUMPTIONS):**
- **NO `@Public()` decorator exists** in codebase
- **Use instead:** `@UseGuards(JwtOptionalGuard)` from `@app/auth/guards/jwt-optional.guard`
- For admin routes: Class-level decorators pattern
  ```typescript
  @Controller('admin/...')
  @UseGuards(JwtGuard)
  @Roles('ADMIN', 'SUPERADMIN')
  @ExcludeProfile()
  @ApiBearerAuth()
  ```

**💡 Architecture Decision:**
- Plan originally suggested creating new `libs/app-config` module
- **Recommendation:** Extend existing `libs/settings` module instead (already global, handles AppSettings table)
- Simpler, less boilerplate, consistent with codebase

---

#### Mobile Findings

**✅ React Query (VERIFIED):**
- Version: `@tanstack/react-query` v5.90.12
- Setup: `integrations/tanstack-query/query-provider.tsx` with MMKV persistence
- **Pattern:** Uses `onError` callbacks in mutations (v5 compatible) ✅
- **Note:** No global error handling configured (minimal setup)

**✅ NativeWind Theme (VERIFIED):**
- Version: `nativewind` v4.2.1
- Config: `tailwind.config.js` with all required tokens
- Tokens available: `background`, `foreground`, `muted`, `muted-foreground` ✅
- CSS variables defined in `global.css` ✅

**✅ Assets (VERIFIED):**
- Icon path: `assets/images/icon.png` exists (89713 bytes) ✅
- Configured in `app.json` line 7

**✅ Expo Packages (VERIFIED):**
- `expo-constants` ~18.0.12 installed ✅
- `expo-linking` ~8.0.10 installed ✅

**❌ Missing Dependencies:**
- `semver` package NOT installed - must add before Task 2.2
- `@types/semver` NOT installed - must add before Task 2.2

---

#### Action Items Before Implementation

- [x] Verify SettingsService API signatures
- [x] Verify Prisma schema structure
- [x] Find auth decorator pattern
- [x] Verify React Query version
- [x] Verify NativeWind theme tokens
- [x] Verify asset paths
- [x] Identify missing dependencies
- [ ] **TODO:** Install semver packages (Task 2.2 dependency)
- [ ] **TODO:** Update all code examples to use `JwtOptionalGuard` instead of `@Public()`

---

### Phase 1: Backend - App Config Module (P0)

#### Task 1.1: Create Update Level Enum & Response DTO

- **Agent:** `backend-specialist`
- **Skills:** `api-patterns`, `clean-code`
- **Priority:** P0 (Foundation)
- **Dependencies:** None
- **Estimated Time:** 10 min

**INPUT:**

- Schema design from above
- 4 update levels: none, optional, recommended, required

**OUTPUT:**

Create files in `riz-be/apps/nest/libs/settings/src/dto/`:

```typescript
// dto/update-level.enum.ts
export enum UpdateLevel {
  NONE = "none",
  OPTIONAL = "optional",
  RECOMMENDED = "recommended",
  REQUIRED = "required",
}

// dto/app-config.response.dto.ts
import { ApiProperty } from '@nestjs/swagger';
import { UpdateLevel } from './update-level.enum';

export class PlatformVersionDto {
  @ApiProperty({ example: '1.2.0' })
  storeVersion: string;

  @ApiProperty({ enum: UpdateLevel, example: UpdateLevel.OPTIONAL })
  updateLevel: UpdateLevel;

  @ApiProperty({ example: 'https://apps.apple.com/app/id123456789' })
  storeUrl: string;
  // NOTE: No updateMessage - handled by i18n in mobile app
}

export class MaintenanceDto {
  @ApiProperty({ example: false })
  enabled: boolean;

  @ApiProperty({ required: false, example: 'Hệ thống đang bảo trì. Dự kiến hoàn thành lúc 14:00.' })
  message?: string;
}

export class AppConfigResponseDto {
  @ApiProperty({ type: PlatformVersionDto })
  ios: PlatformVersionDto;

  @ApiProperty({ type: PlatformVersionDto })
  android: PlatformVersionDto;

  @ApiProperty({ type: MaintenanceDto })
  maintenance: MaintenanceDto;
}
```

**VERIFY:**

- DTOs compile without errors
- Enum values are correct
- Swagger decorators present

---

#### Task 1.2: Implement AppConfig Service

- **Agent:** `backend-specialist`
- **Skills:** `api-patterns`, `database-design`
- **Priority:** P0
- **Dependencies:** Task 1.1
- **Estimated Time:** 20 min

**INPUT:**

- `SettingsService` for reading/writing `AppSettings`
- Key: `app_version_control`

**OUTPUT:**

Create `riz-be/apps/nest/libs/settings/src/app-config.service.ts`:

```typescript
// app-config.service.ts
import { Injectable } from '@nestjs/common';
import { SettingsService } from './settings.service';
import { AppConfigResponseDto } from './dto/app-config.response.dto';
import { UpdateLevel } from './dto/update-level.enum';

@Injectable()
export class AppConfigService {
  private cache: { data: AppConfigResponseDto; timestamp: number } | null = null;
  private readonly CACHE_TTL = 60_000; // 60 seconds

  constructor(private readonly settings: SettingsService) {}

  async getConfig(): Promise<AppConfigResponseDto> {
    const now = Date.now();

    // Return cache if valid
    if (this.cache && now - this.cache.timestamp < this.CACHE_TTL) {
      return this.cache.data;
    }

    // Fetch fresh data from DB
    const raw = await this.settings.get("app_version_control");
    const config = raw
      ? (JSON.parse(raw) as AppConfigResponseDto)
      : this.getDefaultConfig();

    // Update cache
    this.cache = { data: config, timestamp: now };

    return config;
  }

  private getDefaultConfig(): AppConfigResponseDto {
    return {
      ios: {
        storeVersion: "0.0.0",
        updateLevel: UpdateLevel.NONE,
        storeUrl: "",
      },
      android: {
        storeVersion: "0.0.0",
        updateLevel: UpdateLevel.NONE,
        storeUrl: "",
      },
      maintenance: {
        enabled: false,
        message: "",
      },
    };
  }

  async updateConfig(
    config: AppConfigResponseDto,
    userId: bigint,
  ): Promise<void> {
    await this.settings.set(
      "app_version_control",
      JSON.stringify(config),
      "mobile",
      userId,
    );

    // Invalidate cache immediately
    this.cache = null;
  }
}
```

**VERIFY:**

```bash
# Service compiles
cd riz-be/apps/nest && pnpm build

# Unit test patterns (optional):
# 1st request: Cache MISS → Query DB
# 2nd request within 60s: Cache HIT → No DB query
# Admin updates config → Cache invalidated → Next request rebuilds cache
```

---

#### Task 1.3: Create Public App Config Endpoint

- **Agent:** `backend-specialist`
- **Skills:** `api-patterns`, `database-design`
- **Priority:** P0
- **Dependencies:** Task 1.1, Task 1.2
- **Estimated Time:** 20 min

**INPUT:**

- `SettingsService` for reading/writing `AppSettings`
- Key: `app_version_control`

**OUTPUT:**

```typescript
// app-config.service.ts
@Injectable()
export class AppConfigService {
  private cache: { data: AppConfigResponseDto; timestamp: number } | null =
    null;
  private readonly CACHE_TTL = 60_000; // 60 seconds

  constructor(private readonly settings: SettingsService) {}

  async getConfig(): Promise<AppConfigResponseDto> {
    const now = Date.now();

    // Return cache if valid
    if (this.cache && now - this.cache.timestamp < this.CACHE_TTL) {
      return this.cache.data;
    }

    // Fetch fresh data from DB
    const raw = await this.settings.get("app_version_control");
    const config = raw
      ? (JSON.parse(raw) as AppConfigResponseDto)
      : this.getDefaultConfig();

    // Update cache
    this.cache = { data: config, timestamp: now };

    return config;
  }

  private getDefaultConfig(): AppConfigResponseDto {
    return {
      ios: {
        storeVersion: "0.0.0",
        updateLevel: UpdateLevel.NONE,
        storeUrl: "",
      },
      android: {
        storeVersion: "0.0.0",
        updateLevel: UpdateLevel.NONE,
        storeUrl: "",
      },
      maintenance: {
        enabled: false,
        message: "",
      },
    };
  }

  async updateConfig(
    config: AppConfigResponseDto,
    userId: bigint,
  ): Promise<void> {
    await this.settings.set(
      "app_version_control",
      JSON.stringify(config),
      "mobile",
      userId,
    );

    // Invalidate cache immediately
    this.cache = null;
  }
}
```

**VERIFY:**

```bash
# Unit test service
npm run test -- --grep AppConfigService

# Test cache behavior
# 1st request: Cache MISS → Query DB
# 2nd request within 60s: Cache HIT → No DB query
# Admin updates config → Cache invalidated → Next request rebuilds cache
```

---

#### Task 1.3: Create Public App Config Endpoint

- **Agent:** `backend-specialist`
- **Skills:** `api-patterns`, `clean-code`
- **Priority:** P0
- **Dependencies:** Task 1.2
- **Estimated Time:** 15 min

**INPUT:**

- `AppConfigService`
- No authentication required (public endpoint)

**OUTPUT:**

Create `riz-be/apps/nest/libs/settings/src/app-config.controller.ts`:

```typescript
// app-config.controller.ts
import { Controller, Get, UseGuards } from '@nestjs/common';
import { ApiOkResponse, ApiOperation, ApiTags } from '@nestjs/swagger';
import { JwtOptionalGuard } from '@app/auth/guards/jwt-optional.guard';
import { AppConfigService } from './app-config.service';
import { AppConfigResponseDto } from './dto/app-config.response.dto';

@ApiTags('App Config')
@Controller("app-config")
export class AppConfigController {
  constructor(private readonly service: AppConfigService) {}

  @Get()
  @UseGuards(JwtOptionalGuard) // Public endpoint - no auth required
  @ApiOperation({ summary: 'Get app version config and maintenance status' })
  @ApiOkResponse({ type: AppConfigResponseDto })
  async getConfig(): Promise<AppConfigResponseDto> {
    return this.service.getConfig();
  }
}
```

**VERIFY:**

```bash
# Start server and test endpoint
curl http://localhost:3000/api/v1/app-config
# Should return JSON with ios, android, maintenance fields
# updateLevel should be one of: none, optional, recommended, required
```

---

#### Task 1.4: Update Settings Module to Include AppConfig

- **Agent:** `backend-specialist`
- **Skills:** `api-patterns`, `clean-code`
- **Priority:** P0
- **Dependencies:** Task 1.2, Task 1.3
- **Estimated Time:** 10 min

**INPUT:**

- Existing `SettingsModule` in `libs/settings/src/settings.module.ts`
- New `AppConfigService` and `AppConfigController`

**OUTPUT:**

Update `riz-be/apps/nest/libs/settings/src/settings.module.ts`:

```typescript
// settings.module.ts
import { Global, Module } from '@nestjs/common';
import { SettingsService } from './settings.service';
import { AppConfigService } from './app-config.service';
import { AppConfigController } from './app-config.controller';

@Global()
@Module({
  controllers: [AppConfigController], // ADD: Register controller
  providers: [SettingsService, AppConfigService], // ADD: AppConfigService
  exports: [SettingsService, AppConfigService], // ADD: Export for use in other modules
})
export class SettingsModule {}
```

Update `riz-be/apps/nest/libs/settings/src/index.ts` to export new types:

```typescript
// index.ts
export * from './settings.module';
export * from './settings.service';
export * from './app-config.service';
export * from './app-config.controller';
export * from './dto/update-level.enum';
export * from './dto/app-config.response.dto';
```

**VERIFY:**

```bash
# Compile and verify module loads
cd riz-be/apps/nest && pnpm build

# Test endpoint is accessible
pnpm start:local:watch
curl http://localhost:3000/api/v1/app-config
```

---

#### Task 1.5: Seed Default App Config

- **Agent:** `backend-specialist`
- **Skills:** `database-design`
- **Priority:** P1
- **Dependencies:** Task 1.3
- **Estimated Time:** 10 min

**INPUT:**

- Current app versions (iOS: 0.2.1, Android: 1.1.5 from app.config.ts)
- Store URLs for Riz app

**OUTPUT:**

- Migration or seed script to insert default `app_version_control` setting
- Default `updateLevel: "none"` (no prompts initially)

**VERIFY:**

```bash
# Query database for the setting
SELECT * FROM "AppSettings" WHERE key = 'app_version_control';
```

---

### Phase 2: Mobile - Version Check System (P1)

#### Task 2.1: Create API Client for App Config

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `clean-code`
- **Priority:** P1
- **Dependencies:** Task 1.4
- **Estimated Time:** 10 min

**INPUT:**

- Existing `lib/axios.ts` setup
- API response structure with `updateLevel` enum

**OUTPUT:**

```typescript
// lib/api/app-config.ts
import { api } from "../axios";

export type UpdateLevel = "none" | "optional" | "recommended" | "required";

export interface PlatformVersion {
  storeVersion: string;
  updateLevel: UpdateLevel;
  storeUrl: string;
  // NOTE: No updateMessage - use i18n text in app
}

export interface AppConfigResponse {
  ios: PlatformVersion;
  android: PlatformVersion;
  maintenance: {
    enabled: boolean;
    message?: string;
  };
}

export async function getAppConfig(): Promise<AppConfigResponse> {
  const { data } = await api.get("/app-config");
  return data;
}
```

**VERIFY:**

- TypeScript compiles
- Function is exported

---

#### Task 2.2: Create Version Comparison Utility

- **Agent:** `mobile-developer`
- **Skills:** `clean-code`
- **Priority:** P1
- **Dependencies:** Install semver package first (see below)
- **Estimated Time:** 10 min

**DEPENDENCIES (INSTALL FIRST):**

```bash
# Add semver package (NOT currently installed in project)
cd riz-app-v2
bun add semver
bun add -d @types/semver
```

**INPUT:**

- Semantic versioning format (MAJOR.MINOR.PATCH)

**OUTPUT:**

```typescript
// components/app-update/version-utils.ts
import { valid, gt } from "semver";
import Constants from "expo-constants";

/**
 * Compare two semantic versions to check if current version is outdated
 * @param current - Current app version (e.g., "1.0.0")
 * @param required - Required minimum version from server (e.g., "1.2.0")
 * @returns true if current version is older than required version
 */
export function isVersionOutdated(current: string, required: string): boolean {
  // Normalize versions (remove 'v' prefix if exists)
  const normalizedCurrent = valid(current.replace(/^v/, ""));
  const normalizedRequired = valid(required.replace(/^v/, ""));

  // If either version is invalid, fail-safe by not blocking the user
  if (!normalizedCurrent || !normalizedRequired) {
    if (__DEV__) {
      console.warn("[VersionUtils] Invalid version format:", {
        current,
        required,
      });
    }
    return false;
  }

  // Compare versions: returns true if required > current
  return gt(normalizedRequired, normalizedCurrent);
}

export function getCurrentAppVersion(): string {
  return Constants.expoConfig?.version ?? "0.0.0";
}
```

**VERIFY:**

```typescript
// Unit tests
expect(isVersionOutdated("1.0.0", "1.0.1")).toBe(true);
expect(isVersionOutdated("1.0.1", "1.0.0")).toBe(false);
expect(isVersionOutdated("1.1.0", "1.0.9")).toBe(false);
expect(isVersionOutdated("2.0.0", "1.9.9")).toBe(false);

// Edge cases with semver
expect(isVersionOutdated("1.0.0-beta", "1.0.0")).toBe(true); // pre-release < release
expect(isVersionOutdated("v1.0.0", "1.0.1")).toBe(true); // handles 'v' prefix
expect(isVersionOutdated("invalid", "1.0.0")).toBe(false); // fail-safe
```

---

#### Task 2.3: Create Update Storage Utility (MMKV)

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `clean-code`
- **Priority:** P1
- **Dependencies:** None
- **Estimated Time:** 15 min

**INPUT:**

- MMKV storage (existing in project)
- Need to persist: last dismissed version for `optional` level

**OUTPUT:**

```typescript
// components/app-update/update-storage.ts
import { MMKV } from "react-native-mmkv";

const storage = new MMKV({ id: "app-update" });
const DISMISSED_VERSION_KEY = "lastDismissedVersion";

export function getLastDismissedVersion(): string | null {
  return storage.getString(DISMISSED_VERSION_KEY) ?? null;
}

export function setLastDismissedVersion(version: string): void {
  storage.set(DISMISSED_VERSION_KEY, version);
}

export function shouldShowOptionalUpdate(storeVersion: string): boolean {
  const lastDismissed = getLastDismissedVersion();
  // Show if never dismissed, or dismissed version is different from current store version
  return lastDismissed !== storeVersion;
}
```

**VERIFY:**

- Dismiss version 1.2.0 → Don't show for 1.2.0 again
- New storeVersion 1.3.0 published → Show again

---

#### Task 2.4: Create React Query Hook for App Config

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `nextjs-react-expert`
- **Priority:** P1
- **Dependencies:** Task 2.1
- **Estimated Time:** 10 min

**INPUT:**

- `getAppConfig` API function
- Existing React Query patterns in the app

**OUTPUT:**

```typescript
// hooks/use-app-config.ts
import { useQuery } from "@tanstack/react-query";
import { getAppConfig, AppConfigResponse } from "@/lib/api/app-config";

export function useAppConfig() {
  return useQuery<AppConfigResponse>({
    queryKey: ["app-config"],
    queryFn: getAppConfig,

    // Cache config for 5 minutes (matches foreground check interval)
    staleTime: 5 * 60 * 1000,
    gcTime: 10 * 60 * 1000, // Keep in cache for 10 minutes

    // NO RETRY - fail silently if API call fails
    retry: false,

    // NO REFETCH - one-time check at app startup
    refetchOnWindowFocus: false,
    refetchOnMount: false,
    refetchOnReconnect: false,

    // Silent error handling - don't throw to UI
    useErrorBoundary: false,

    onError: (error) => {
      // Log for debugging but don't block the app
      if (__DEV__) {
        console.warn(
          "[AppConfig] Failed to fetch, app will continue normally:",
          error,
        );
      }
      // Optional: Send to error monitoring service (Sentry, Firebase, etc.)
    },
  });
}
```

**VERIFY:**

- Hook returns data/error/isLoading correctly
- Check React Query devtools
- API failure: App continues normally, no error thrown
- No automatic retries on failure

---

#### Task 2.5: Create ForceUpdateModal Component (Required Level)

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `frontend-design`
- **Priority:** P1
- **Dependencies:** Task 2.2
- **Estimated Time:** 30 min

**INPUT:**

- Design requirements:
  - Full-screen blocking modal
  - Cannot be dismissed (no backdrop press, no back button)
  - App logo at top
  - "Update Available" title (i18n: `update.title`)
  - Message from i18n (i18n: `update.message`)
  - Single "Update Now" button with gradient (i18n: `update.button.updateNow`)

**OUTPUT:**

```typescript
// components/app-update/ForceUpdateModal.tsx
import { Modal, View, Text, BackHandler, Image, Alert } from 'react-native';
import { Button } from '@/components/ui/button';
import * as Linking from 'expo-linking';
import { useEffect } from 'react';
import { useTranslation } from 'react-i18next';

interface Props {
  visible: boolean;
  storeUrl: string;
}

export function ForceUpdateModal({ visible, storeUrl }: Props) {
  const { t } = useTranslation();

  // Block hardware back button on Android
  useEffect(() => {
    if (!visible) return;
    const handler = BackHandler.addEventListener('hardwareBackPress', () => true);
    return () => handler.remove();
  }, [visible]);

  const handleUpdate = async () => {
    try {
      const supported = await Linking.canOpenURL(storeUrl);
      if (!supported) {
        Alert.alert(
          t('common.error'),
          t('update.error.cannotOpenStore')
        );
        return;
      }
      await Linking.openURL(storeUrl);
    } catch (error) {
      console.error('[ForceUpdateModal] Failed to open store:', error);
      Alert.alert(
        t('common.error'),
        t('update.error.failedToOpenStore')
      );
    }
  };

  return (
    <Modal
      visible={visible}
      animationType="fade"
      transparent={false}
      statusBarTranslucent
      presentationStyle="fullScreen"
    >
      <View className="flex-1 justify-center items-center bg-background p-6">
        {/* App Logo */}
        <Image source={require('@/assets/images/icon.png')} className="w-24 h-24 mb-8" />

        {/* Title */}
        <Text className="text-2xl font-bold text-foreground mb-4">
          {t('update.title')}
        </Text>

        {/* Message - from i18n, not server */}
        <Text className="text-center text-muted-foreground mb-8">
          {t('update.message')}
        </Text>

        {/* Update Button */}
        <Button onPress={handleUpdate} className="w-full">
          {t('update.button.updateNow')}
        </Button>
      </View>
    </Modal>
  );
}
```

**VERIFY:**

- Modal renders correctly
- Cannot be dismissed by any means (backdrop, back button)
- "Update Now" button handles errors gracefully
- Shows alert if store URL cannot be opened

---

#### Task 2.6: Create SoftUpdateModal Component (Optional/Recommended Levels)

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `frontend-design`
- **Priority:** P1
- **Dependencies:** Task 2.3
- **Estimated Time:** 30 min

**INPUT:**

- Design requirements:
  - Semi-transparent backdrop
  - CAN be dismissed (backdrop press, "Later" button)
  - App logo at top
  - "Update Available" title (i18n: `update.title`)
  - Message from i18n (i18n: `update.message`)
  - Two buttons: "Update Now" (primary) + "Later" (secondary)
  - For `optional`: persist dismiss to MMKV
  - For `recommended`: dismiss only for current session

**OUTPUT:**

```typescript
// components/app-update/SoftUpdateModal.tsx
import { Modal, View, Text, Pressable, Image, Alert } from 'react-native';
import { Button } from '@/components/ui/button';
import * as Linking from 'expo-linking';
import { setLastDismissedVersion } from './update-storage';
import { useTranslation } from 'react-i18next';

interface Props {
  visible: boolean;
  storeUrl: string;
  storeVersion: string;
  updateLevel: 'optional' | 'recommended';
  onDismiss: () => void;
}

export function SoftUpdateModal({
  visible,
  storeUrl,
  storeVersion,
  updateLevel,
  onDismiss
}: Props) {
  const { t } = useTranslation();

  const handleUpdate = async () => {
    try {
      const supported = await Linking.canOpenURL(storeUrl);
      if (!supported) {
        Alert.alert(
          t('common.error'),
          t('update.error.cannotOpenStore')
        );
        return;
      }
      await Linking.openURL(storeUrl);
    } catch (error) {
      console.error('[SoftUpdateModal] Failed to open store:', error);
      Alert.alert(
        t('common.error'),
        t('update.error.failedToOpenStore')
      );
    }
  };

  const handleLater = () => {
    // For optional level: persist to MMKV so it doesn't show again for this version
    if (updateLevel === 'optional') {
      setLastDismissedVersion(storeVersion);
    }
    // For recommended: just dismiss (will show again next cold start)
    onDismiss();
  };

  return (
    <Modal visible={visible} animationType="slide" transparent>
      <Pressable
        className="flex-1 justify-end bg-black/50"
        onPress={handleLater}
      >
        <Pressable
          className="bg-background rounded-t-3xl p-6"
          onPress={(e) => e.stopPropagation()} // Prevent dismiss when tapping content
        >
          {/* Header with close button */}
          <View className="items-end mb-4">
            <Pressable onPress={handleLater}>
              <Text className="text-muted-foreground text-lg">✕</Text>
            </Pressable>
          </View>

          {/* App Logo */}
          <View className="items-center mb-4">
            <Image source={require('@/assets/images/icon.png')} className="w-16 h-16" />
          </View>

          {/* Title */}
          <Text className="text-xl font-bold text-foreground text-center mb-2">
            {t('update.title')}
          </Text>

          {/* Message - from i18n, not server */}
          <Text className="text-center text-muted-foreground mb-6">
            {t('update.message')}
          </Text>

          {/* Buttons */}
          <View className="gap-3">
            <Button onPress={handleUpdate} className="w-full">
              {t('update.button.updateNow')}
            </Button>
            <Button variant="ghost" onPress={handleLater} className="w-full">
              {t('update.button.later')}
            </Button>
          </View>
        </Pressable>
      </Pressable>
    </Modal>
  );
}
```

**VERIFY:**

- Modal renders as bottom sheet style
- Can be dismissed with backdrop tap or "Later" button
- For `optional`: After dismiss, doesn't show again for same version
- For `recommended`: After dismiss, shows again on next app restart

---

#### Task 2.7: Create MaintenanceModal Component

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `frontend-design`
- **Priority:** P1
- **Dependencies:** None
- **Estimated Time:** 20 min

**INPUT:**

- Blocking behavior like ForceUpdateModal
- Maintenance icon and message
- Cannot be dismissed (no backdrop, no back button, no actions)
- User must close app and return later

**OUTPUT:**

```typescript
// components/app-update/MaintenanceModal.tsx
import { Modal, View, Text, BackHandler } from 'react-native';
import { useEffect } from 'react';
import { useTranslation } from 'react-i18next';

interface Props {
  visible: boolean;
  message?: string;
}

export function MaintenanceModal({ visible, message }: Props) {
  const { t } = useTranslation();

  // Block hardware back button on Android (same as ForceUpdateModal)
  useEffect(() => {
    if (!visible) return;
    const handler = BackHandler.addEventListener('hardwareBackPress', () => true);
    return () => handler.remove();
  }, [visible]);

  return (
    <Modal
      visible={visible}
      animationType="fade"
      transparent={false}
      statusBarTranslucent
      presentationStyle="fullScreen"
    >
      <View className="flex-1 justify-center items-center bg-background p-6">
        {/* Maintenance Icon */}
        <View className="w-24 h-24 mb-8 items-center justify-center bg-orange-100 dark:bg-orange-900/20 rounded-full">
          <Text className="text-4xl">🔧</Text>
        </View>

        {/* Title */}
        <Text className="text-2xl font-bold text-foreground mb-4 text-center">
          {t('maintenance.title')}
        </Text>

        {/* Message - from server or fallback to i18n */}
        <Text className="text-center text-muted-foreground px-4">
          {message || t('maintenance.defaultMessage')}
        </Text>

        {/* No action buttons - user must close app and return later */}
      </View>
    </Modal>
  );
}
```

**VERIFY:**

- Modal renders with maintenance icon and message
- Cannot be dismissed by any means (blocking behavior)
- Back button blocked on Android
- No action buttons - screen is purely informational
- Uses i18n with fallback for message

---

#### Task 2.8: Integrate Version Check in Root Layout

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `clean-code`
- **Priority:** P0 (Critical)
- **Dependencies:** Task 2.4, Task 2.5, Task 2.6, Task 2.7
- **Estimated Time:** 45 min

**INPUT:**

- `app/_layout.tsx` (root layout)
- `useAppConfig` hook
- All modal components
- Update level logic

**OUTPUT:**

```typescript
// app/_layout.tsx (modifications)
import { useState, useMemo, useEffect } from 'react';
import { Platform } from 'react-native';
import { useAppConfig } from '@/hooks/use-app-config';
import { ForceUpdateModal } from '@/components/app-update/ForceUpdateModal';
import { SoftUpdateModal } from '@/components/app-update/SoftUpdateModal';
import { MaintenanceModal } from '@/components/app-update/MaintenanceModal';
import { isVersionOutdated, getCurrentAppVersion } from '@/components/app-update/version-utils';
import { shouldShowOptionalUpdate } from '@/components/app-update/update-storage';

export default function RootLayout() {
  // Silent fetch - app continues normally, no loading state
  const { data: appConfig, refetch } = useAppConfig();

  // Track dismiss state for soft updates (session-only)
  const [softUpdateDismissed, setSoftUpdateDismissed] = useState(false);

  // Get platform-specific config
  const platformConfig = useMemo(() => {
    if (!appConfig) return null;
    return Platform.OS === 'ios' ? appConfig.ios : appConfig.android;
  }, [appConfig]);

  const currentVersion = getCurrentAppVersion();

  // Determine what dialog to show with MAINTENANCE PRIORITY
  const dialogState = useMemo(() => {
    // No config yet → show nothing (silent)
    if (!platformConfig || !appConfig) {
      return { type: 'none' as const };
    }

    // 🔴 PRIORITY 1: MAINTENANCE MODE (absolute priority)
    // If maintenance is ON, ignore all update levels
    if (appConfig.maintenance?.enabled) {
      return {
        type: 'maintenance' as const,
        message: appConfig.maintenance.message
      };
    }

    // 🟡 PRIORITY 2: Check if version outdated
    const isOutdated = platformConfig.storeVersion &&
      isVersionOutdated(currentVersion, platformConfig.storeVersion);

    if (!isOutdated) {
      return { type: 'none' as const };
    }

    // 🟠 PRIORITY 3: Determine update level
    switch (platformConfig.updateLevel) {
      case 'required':
        return {
          type: 'required' as const,
          storeUrl: platformConfig.storeUrl,
          // message: use i18n text in component
        };

      case 'recommended':
        // Show every cold start, but can be dismissed for session
        if (softUpdateDismissed) {
          return { type: 'none' as const };
        }
        return {
          type: 'recommended' as const,
          storeUrl: platformConfig.storeUrl,
          storeVersion: platformConfig.storeVersion,
          // message: use i18n text in component
        };

      case 'optional':
        // Check MMKV - already dismissed for this version?
        if (softUpdateDismissed || !shouldShowOptionalUpdate(platformConfig.storeVersion)) {
          return { type: 'none' as const };
        }
        return {
          type: 'optional' as const,
          storeUrl: platformConfig.storeUrl,
          storeVersion: platformConfig.storeVersion,
          // message: use i18n text in component
        };

      case 'none':
      default:
        return { type: 'none' as const };
    }
  }, [appConfig, platformConfig, currentVersion, softUpdateDismissed]);

  // Debug logging (optional, remove in production)
  useEffect(() => {
    if (__DEV__ && dialogState.type !== 'none') {
      console.log('[AppUpdate] Dialog state:', dialogState);
    }
  }, [dialogState]);

  return (
    <>
      {/* App content loads normally - NOT blocked by API call */}
      <Stack>
        {/* Your existing app navigation */}
      </Stack>

      {/* GLOBAL MODALS - render at root level, show on all screens */}

      {/* 🔴 Maintenance Modal - Highest Priority */}
      {/* NOTE: maintenance.message kept - can contain admin-specific info */}
      <MaintenanceModal
        visible={dialogState.type === 'maintenance'}
        message={dialogState.type === 'maintenance' ? dialogState.message : undefined}
      />

      {/* 🟠 Force Update Modal - Required Level */}
      {/* NOTE: No message prop - uses i18n internally */}
      <ForceUpdateModal
        visible={dialogState.type === 'required'}
        storeUrl={dialogState.type === 'required' ? dialogState.storeUrl : ''}
      />

      {/* 🟡 Soft Update Modal - Optional/Recommended Levels */}
      {/* NOTE: No message prop - uses i18n internally */}
      <SoftUpdateModal
        visible={dialogState.type === 'optional' || dialogState.type === 'recommended'}
        storeUrl={'storeUrl' in dialogState ? dialogState.storeUrl : ''}
        storeVersion={'storeVersion' in dialogState ? dialogState.storeVersion : ''}
        updateLevel={dialogState.type === 'optional' || dialogState.type === 'recommended'
          ? dialogState.type
          : 'optional'
        }
        onDismiss={() => setSoftUpdateDismissed(true)}
      />
    </>
  );
}
```

**VERIFY:**

| Test Case                               | Expected Result                                |
| --------------------------------------- | ---------------------------------------------- |
| Maintenance ON + any update level       | Show ONLY maintenance modal (highest priority) |
| `updateLevel: "required"` + outdated    | Force modal, cannot dismiss                    |
| `updateLevel: "recommended"` + outdated | Soft modal, dismiss → hidden until restart     |
| `updateLevel: "optional"` + outdated    | Soft modal, dismiss → hidden for this version  |
| `updateLevel: "none"`                   | No modal                                       |
| Version current                         | No modal regardless of level                   |
| API loading                             | App loads normally, no blocking                |
| API fails                               | App works normally (fail silently)             |
| User navigates between screens          | Dialog persists (global display)               |

---

#### Task 2.9: Create Foreground Config Check Hook

- **Agent:** `mobile-developer`
- **Skills:** `mobile-design`, `clean-code`
- **Priority:** P1
- **Dependencies:** Task 2.4
- **Estimated Time:** 20 min

**INPUT:**

- `AppState` API from React Native
- React Query's `invalidateQueries`
- 5-minute debounce interval (matches staleTime)

**RATIONALE:**

Current plan only checks on cold start. This task adds **foreground check** to ensure:

- Maintenance mode is enforced in real-time (not bypassed by backgrounding)
- Force update is enforced when user returns from background
- Debounced to 5 minutes to reduce unnecessary API calls

**OUTPUT:**

```typescript
// hooks/use-foreground-config-check.ts
import { useEffect, useRef } from "react";
import { AppState, AppStateStatus } from "react-native";
import { useQueryClient } from "@tanstack/react-query";

/**
 * Debounce interval for foreground checks.
 * Matches React Query staleTime to avoid redundant fetches.
 * 5 minutes is reasonable because:
 * - Backend cache is 60s (always fresh enough)
 * - Maintenance/force update changes are rare events
 * - Reduces battery and network usage
 */
const FOREGROUND_CHECK_INTERVAL = 5 * 60 * 1000; // 5 minutes

/**
 * Hook to re-check app config when app returns from background.
 *
 * Behavior:
 * - Detects when app transitions from background/inactive → active
 * - Only triggers refetch if > 5 minutes since last check
 * - Non-blocking: uses invalidateQueries which triggers background fetch
 *
 * Why needed:
 * - Cold start check alone is not sufficient
 * - User could background app, admin enables maintenance, user returns
 * - Without this hook, user would continue using app until next cold start
 */
export function useForegroundConfigCheck() {
  const queryClient = useQueryClient();
  const appState = useRef(AppState.currentState);
  const lastCheckTime = useRef(Date.now());

  useEffect(() => {
    const subscription = AppState.addEventListener(
      "change",
      (nextAppState: AppStateStatus) => {
        // Only act when transitioning FROM background/inactive TO active
        const wasBackground = appState.current.match(/inactive|background/);
        const isNowActive = nextAppState === "active";

        if (wasBackground && isNowActive) {
          const timeSinceLastCheck = Date.now() - lastCheckTime.current;

          if (timeSinceLastCheck >= FOREGROUND_CHECK_INTERVAL) {
            if (__DEV__) {
              console.log(
                "[ForegroundCheck] App foregrounded, triggering config refresh.",
                `Time since last check: ${Math.round(timeSinceLastCheck / 1000)}s`,
              );
            }

            // Invalidate cache → triggers background refetch
            // refetchType: 'active' ensures only mounted components refetch
            queryClient.invalidateQueries({
              queryKey: ["app-config"],
              refetchType: "active",
            });

            lastCheckTime.current = Date.now();
          } else if (__DEV__) {
            console.log(
              "[ForegroundCheck] App foregrounded, skipping check (debounced).",
              `Time since last check: ${Math.round(timeSinceLastCheck / 1000)}s`,
            );
          }
        }

        appState.current = nextAppState;
      },
    );

    return () => subscription?.remove();
  }, [queryClient]);
}
```

**INTEGRATION in \_layout.tsx:**

```typescript
// app/_layout.tsx (add to existing)
import { useForegroundConfigCheck } from "@/hooks/use-foreground-config-check";

export default function RootLayout() {
  // Existing hook
  const { data: appConfig, refetch } = useAppConfig();

  // NEW: Enable foreground re-checking
  useForegroundConfigCheck();

  // ... rest of the component
}
```

**VERIFY:**

| Test Case                | Steps                                                                                          | Expected Result                                             |
| ------------------------ | ---------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Basic foreground check   | 1. Open app → 2. Background for 6 min → 3. Return                                              | Config refetched, log shows "triggering config refresh"     |
| Debounce working         | 1. Open app → 2. Background for 2 min → 3. Return                                              | No refetch, log shows "skipping check (debounced)"          |
| Maintenance enforcement  | 1. Open app → 2. Background → 3. Admin enables maintenance → 4. Wait 5 min → 5. Return         | Maintenance modal appears                                   |
| Force update enforcement | 1. Open app → 2. Background → 3. Admin sets required + new version → 4. Wait 5 min → 5. Return | Force update modal appears                                  |
| Rapid app switching      | 1. Switch between apps rapidly (< 5 min intervals)                                             | Only 1 API call (debounced)                                 |
| No refetch if fresh      | 1. Open app → 2. Background for 5 min → 3. Return                                              | Query invalidated but may use fresh cache (staleTime logic) |

---

### Phase 3: Admin Panel (Optional - P2)

**NOTE:** Admin API will extend the existing `SettingsModule` with an admin controller.

#### Task 3.1: Create Admin API for Config Update

- **Agent:** `backend-specialist`
- **Skills:** `api-patterns`
- **Priority:** P2
- **Dependencies:** Task 1.2
- **Estimated Time:** 30 min

**INPUT:**

- `AppConfigService.updateConfig()` method already exists
- Auth-protected endpoint pattern from codebase
- Admin role required

**OUTPUT:**

Create `riz-be/apps/nest/libs/settings/src/admin-app-config.controller.ts`:

```typescript
// admin-app-config.controller.ts
import { Body, Controller, Get, Put, UseGuards } from '@nestjs/common';
import { ApiBearerAuth, ApiOkResponse, ApiOperation, ApiTags } from '@nestjs/swagger';
import { JwtGuard } from '@app/auth/guards/jwt.guard';
import { Roles } from '@app/core/decorators/role.decorator';
import { ExcludeProfile } from '@app/core/decorators/exclude-profile.decorator';
import { CurUser } from '@app/core/decorators/user.decorator';
import { User } from '@prisma/client';
import { AppConfigService } from './app-config.service';
import { AppConfigResponseDto } from './dto/app-config.response.dto';

@ApiTags('Admin - App Config')
@Controller('admin/app-config')
@UseGuards(JwtGuard)
@Roles('ADMIN', 'SUPERADMIN')
@ExcludeProfile()
@ApiBearerAuth()
export class AdminAppConfigController {
  constructor(private readonly service: AppConfigService) {}

  @Get()
  @ApiOperation({ summary: 'Get current app config (admin view)' })
  @ApiOkResponse({ type: AppConfigResponseDto })
  async getConfig(): Promise<AppConfigResponseDto> {
    return this.service.getConfig();
  }

  @Put()
  @ApiOperation({ summary: 'Update app config (version control & maintenance)' })
  @ApiOkResponse({ type: AppConfigResponseDto })
  async updateConfig(
    @Body() dto: AppConfigResponseDto,
    @CurUser() user: User,
  ): Promise<AppConfigResponseDto> {
    await this.service.updateConfig(dto, user.id);
    return this.service.getConfig();
  }
}
```

Update `settings.module.ts` to include admin controller:

```typescript
@Global()
@Module({
  controllers: [
    AppConfigController,
    AdminAppConfigController, // ADD
  ],
  providers: [SettingsService, AppConfigService],
  exports: [SettingsService, AppConfigService],
})
export class SettingsModule {}
```

**VERIFY:**

```bash
# Test admin endpoint (requires auth token)
curl -H "Authorization: Bearer $ADMIN_TOKEN" \
  http://localhost:3000/api/v1/admin/app-config

# Test update
curl -X PUT \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"ios":{"storeVersion":"1.2.0","updateLevel":"optional","storeUrl":"..."},...}' \
  http://localhost:3000/api/v1/admin/app-config
```

- Only ADMIN/SUPERADMIN can access ✅
- Invalid updateLevel returns 400 (class-validator) ✅
- Cache invalidates immediately after update ✅

---

#### Task 3.2: Create Admin UI for App Config

- **Agent:** `frontend-specialist`
- **Skills:** `frontend-design`
- **Priority:** P2
- **Dependencies:** Task 3.1
- **Estimated Time:** 60 min

**INPUT:**

- Existing `riz-admin-fe` patterns
- TanStack Query

**OUTPUT:**

- Settings page with:
  - **iOS Section:**
    - Version input
    - Update Level dropdown (none/optional/recommended/required)
    - Store URL input
    - Message textarea
  - **Android Section:** (same fields)
  - **Maintenance Section:**
    - Enable toggle
    - Message textarea
    - Estimated end time (optional)

**VERIFY:**

- Changes save successfully
- App reflects changes on next launch

---

## Phase X: Final Verification

### Pre-Deployment Checklist

- [ ] **Backend Lint:** `cd riz-be && npm run lint`
- [ ] **Backend Tests:** `cd riz-be && npm run test`
- [ ] **Mobile Lint:** `cd riz-app-v2 && npm run lint`
- [ ] **Mobile TypeScript:** `cd riz-app-v2 && npx tsc --noEmit`

### Integration Tests

| Scenario                                                   | Expected                                       |
| ---------------------------------------------------------- | ---------------------------------------------- | --- | -------------------- | -------- |
| Maintenance ON + any update level                          | Maintenance modal ONLY (highest priority)      |
| Maintenance ON + Required update                           | Maintenance modal ONLY (ignore update)         |
| `required` + outdated                                      | Force modal, cannot dismiss, blocks app        |
| `recommended` + outdated                                   | Soft modal, dismiss, hidden until cold restart |
| `recommended` + dismiss + kill app + reopen                | Soft modal appears again                       |
| `optional` + outdated                                      | Soft modal, dismiss, hidden for this version   |
| `optional` + dismiss + new storeVersion                    | Soft modal appears again                       |
| `optional` + dismiss + same storeVersion + cold restart    | No modal                                       | \n  | `none` + any version | No modal |
| Current version + any level                                | No modal                                       |
| API fails                                                  | App works normally (fail silently, no retry)   |
| API loading                                                | App continues loading (non-blocking)           |
| Foreground after 5+ min                                    | Config refetched, modals updated if needed     |
| Foreground after < 5 min                                   | No refetch (debounced)                         |
| Background → Admin enables maintenance → Foreground 5+ min | Maintenance modal appears                      |

### Manual Testing

- [ ] iOS Simulator: All 3 modals render correctly
- [ ] Android Emulator: All 3 modals render correctly, back button blocked
- [ ] "Update Now" opens correct App Store URL with error handling
- [ ] "Update Now" opens correct Play Store URL with error handling
- [ ] Store URL error shows alert message
- [ ] MMKV persistence works for `optional` level
- [ ] Session persistence works for `recommended` level
- [ ] Maintenance modal blocks all interactions
- [ ] Maintenance priority: Shows ONLY maintenance when ON (ignores update level)
- [ ] API failure: App continues normally, no error shown
- [ ] Dialog persists during navigation between screens
- [ ] Foreground check: After 5+ min in background, config is refetched
- [ ] Foreground debounce: Rapid app switching doesn't cause multiple API calls
- [ ] Foreground maintenance: Admin enables maintenance while user in background → modal shows on return

---

## Rollback Strategy

| Issue                      | Rollback Action                                   |
| -------------------------- | ------------------------------------------------- |
| Backend deployment fails   | Revert to previous container                      |
| Mobile crashes on launch   | Set `updateLevel: "none"` in database             |
| Wrong Store URL            | Update `app_version_control` setting              |
| Users stuck in update loop | Set `updateLevel: "optional"` so they can dismiss |

---

## Risks & Mitigations

| Risk                             | Probability | Impact                       | Mitigation                                         |
| -------------------------------- | ----------- | ---------------------------- | -------------------------------------------------- |
| API call fails on cold start     | Medium      | Low (app continues normally) | Silent fail - no retry, app works without config   |
| Version comparison bug           | Low         | High                         | Use semver library + extensive unit tests          |
| MMKV read fails                  | Low         | Low                          | Fallback to showing update dialog                  |
| Store URL invalid                | Low         | Medium                       | Validate URL on save + error handling in modal     |
| User stuck in update loop        | Low         | High                         | `optional` level allows dismiss                    |
| Maintenance + critical update    | Low         | Medium                       | Maintenance takes priority (admin must disable it) |
| In-memory cache lost on redeploy | Medium      | Low (rebuilds automatically) | Cache rebuilds on first request after deploy       |

---

## Notes

- **Store URLs:**
  - iOS App Store: `https://apps.apple.com/app/id[APP_ID]`
  - Google Play Store: `https://play.google.com/store/apps/details?id=[PACKAGE_NAME]`
  - Current Android package: `com.rizfe`

- **Version Sources:**
  - iOS: `Constants.expoConfig?.version` (from app.config.ts)
  - Android: Same as iOS in Expo managed workflow

- **Fallback Behavior:**
  - If API call fails → App continues normally (fail-silent strategy)
  - No retry, no refetch on focus/reconnect
  - This prevents blocking users if server is down

- **Priority Order:**
  1. **Maintenance Mode** (highest) - Blocks app, shows only maintenance dialog
  2. **Force Update** (required) - Blocks app if version outdated
  3. **Recommended Update** - Shows every cold start, dismissible
  4. **Optional Update** - Shows once per version, dismissible

- **Update Level Use Cases:**
  - `none`: During development or when no updates needed
  - `optional`: Minor updates, bug fixes, can be skipped
  - `recommended`: Important updates, remind on every start
  - `required`: Critical/security updates, must update

- **Caching Strategy:**
  - Backend: In-memory cache with 60s TTL (reduces DB queries)
  - Mobile: React Query staleTime 5 minutes (matches foreground check interval)
  - Foreground check: Debounced to 5 minutes to prevent excessive API calls
  - Cache invalidated immediately when admin updates config
  - Max delay for enforcement: 5 minutes (when user returns from background)

- **Global Modal Display:**
  - All modals render at root layout level
  - Persist during navigation between screens
  - User cannot dismiss required/maintenance modals by any means

- **i18n Translation Keys (Required for multi-language support):**

  ```json
  // locales/en/common.json (or translation file)
  {
    "common": {
      "error": "Error"
    },
    "update": {
      "title": "Update Available",
      "message": "A new version is available. Please update to get the latest features and improvements.",
      "button": {
        "updateNow": "Update Now",
        "later": "Later"
      },
      "error": {
        "cannotOpenStore": "Cannot open store. Please update the app manually from your app store.",
        "failedToOpenStore": "Failed to open store. Please try again or update manually."
      }
    },
    "maintenance": {
      "title": "Under Maintenance",
      "defaultMessage": "We are currently performing maintenance. Please check back later."
    }
  }
  ```

  **Vietnamese translation:**

  ```json
  {
    "common": {
      "error": "Lỗi"
    },
    "update": {
      "title": "Có bản cập nhật",
      "message": "Đã có phiên bản mới. Vui lòng cập nhật để có trải nghiệm tốt nhất.",
      "button": {
        "updateNow": "Cập nhật ngay",
        "later": "Để sau"
      }
    },
    "maintenance": {
      "title": "Đang bảo trì",
      "defaultMessage": "Hệ thống đang được bảo trì. Vui lòng quay lại sau."
    }
  }
  ```

  **NOTE:** `maintenance.message` từ server có thể chứa thông tin cụ thể (thời gian hoàn thành, lý do).
  Nếu server không trả message, fallback sang `t('maintenance.defaultMessage')`.
