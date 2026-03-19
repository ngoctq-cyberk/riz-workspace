# IPC Structure Coding Standard

This document defines the coding standards for adding new IPC (Inter-Process Communication) channels in Blixt Editor.

## Overview

IPC channels are organized by **groups/domains** to maintain clean separation of concerns and improve code maintainability.

## File Locations

### 1. IPC Channel Definitions

**File**: `src/types/ipc.type.ts`

All new IPC channels MUST be defined in this file, grouped by domain using separate enums.

**Pattern**:
```typescript
export enum Ipc[Domain]Channel {
  [ActionName] = '[domain]-[action-name]',
}
```

**Existing Groups**:
- `IpcActivationChannel` - Activation/license related
- `IpcImageChannel` - Image editor related
- `IpcVideoChannel` - Video editor related
- `IpcCameraChannel` - Camera overlay related

**Adding New Group**:
```typescript
// Example: Adding App Update channels
export enum IpcAppChannel {
  CheckForUpdates = 'app-check-for-updates',
  AppWindowShown = 'app-window-shown',
}
```

### 2. IPC Handlers (Main Process)

**File**: `src/main/services/ipc.service.ts`

All IPC handlers MUST be registered in `IpcService` class, grouped by domain.

**Pattern**:
```typescript
export class IpcService {
  static init = () => {
    IpcService.initActivationAPI();
    IpcService.initImageAPI();
    IpcService.initVideoAPI();
    IpcService.init[NewDomain]API(); // Add new domain here
  };

  private static init[NewDomain]API = () => {
    ipcMain.handle(Ipc[Domain]Channel.[Action], async (_, ...args) => {
      // Handler logic
    });
  };
}
```

**Exception**: Some legacy handlers exist in `main.ts`. New handlers should be added to `ipc.service.ts`.

### 3. Preload Bridge (Renderer Access)

**File**: `src/main/preload.ts`

Exposed APIs MUST be grouped by domain as separate API objects.

**Pattern**:
```typescript
const [domain]API = {
  [method]: () => ipcRenderer.invoke(Ipc[Domain]Channel.[Action]),
};

contextBridge.exposeInMainWorld('[domain]API', [domain]API);

export type [Domain]API = typeof [domain]API;
```

**Existing API Objects**:
- `electronHandler` / `window.electron` - Legacy main handler
- `activationAPI` / `window.activationAPI` - Activation-specific
- `imageAPI` / `window.imageAPI` - Image editor specific
- `videoAPI` / `window.videoAPI` - Video editor specific

### 4. TypeScript Declarations (IDE Support)

**File**: `src/renderer/preload.d.ts`

When adding a new API object, MUST update this file to enable IDE autocompletion.

**Pattern**:
```typescript
import {
  // ... existing imports
  [Domain]API,
} from '@/main/preload';

declare global {
  interface Window {
    // ... existing
    [domain]API: [Domain]API;
  }
}
```

## Naming Conventions

### Channel Names
- Use kebab-case: `domain-action-name`
- Prefix with domain: `app-check-for-updates`, `video-export-for-id`

### Enum Members
- Use PascalCase: `CheckForUpdates`, `AppWindowShown`

### Handler Methods
- Use camelCase: `initAppAPI`, `checkForUpdates`

### API Objects
- Use camelCase with API suffix: `appAPI`, `videoAPI`

## Example: Adding New IPC Channel

### Step 1: Define Channel (src/types/ipc.type.ts)
```typescript
export enum IpcAppChannel {
  CheckForUpdates = 'app-check-for-updates',
  AppWindowShown = 'app-window-shown',
}
```

### Step 2: Add Handler (src/main/services/ipc.service.ts)
```typescript
import { IpcAppChannel } from '@/types/ipc.type';

export class IpcService {
  static init = () => {
    // ... existing inits
    IpcService.initAppAPI();
  };

  private static initAppAPI = () => {
    ipcMain.handle(IpcAppChannel.CheckForUpdates, async () => {
      // Implementation
    });
  };
}
```

### Step 3: Expose in Preload (src/main/preload.ts)
```typescript
import { IpcAppChannel } from '@/types/ipc.type';

const appAPI = {
  checkForUpdates: () => ipcRenderer.invoke(IpcAppChannel.CheckForUpdates),
};

contextBridge.exposeInMainWorld('appAPI', appAPI);

export type AppAPI = typeof appAPI;
```

### Step 4: Add TypeScript Declaration (src/renderer/preload.d.ts)
```typescript
import {
  ActivationAPI,
  ElectronHandler,
  ImageAPI,
  VideoAPI,
  AppAPI, // Add new import
} from '@/main/preload';

declare global {
  interface Window {
    electron: ElectronHandler;
    electronAPI: ElectronHandler['ipcRenderer'];
    activationAPI: ActivationAPI;
    imageAPI: ImageAPI;
    videoAPI: VideoAPI;
    appAPI: AppAPI; // Add new declaration
  }
}
```

### Step 5: Use in Renderer
```typescript
await window.appAPI.checkForUpdates();
```

## Legacy Code Notes

- `src/types/index.ts` contains legacy `IpcChannels` enum
- New channels should NOT be added to this enum
- Existing handlers in `main.ts` can remain, but new handlers should use `ipc.service.ts`

## Checklist for New IPC Channels

- [ ] Channel defined in appropriate enum in `src/types/ipc.type.ts`
- [ ] Handler registered in `src/main/services/ipc.service.ts`
- [ ] Method exposed in `src/main/preload.ts` as `[domain]API` object
- [ ] TypeScript type exported (`export type [Domain]API = typeof [domain]API`)
- [ ] TypeScript declaration added in `src/renderer/preload.d.ts`
- [ ] Channel naming follows conventions (kebab-case with domain prefix)
- [ ] Usage in renderer: `window.[domain]API.[method]()`
