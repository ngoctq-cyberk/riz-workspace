# FFmpeg Usage Coding Standard

This document defines the coding standards for using FFmpeg in Blixt Editor.

## Overview

FFmpeg operations **MUST** be performed in the main process, not the renderer process. All FFmpeg interactions should go through the `VideoUtil` utility class and IPC channels.

## Core Principles

1. **Main Process Only**: FFmpeg can only be executed in the main process (Node.js environment)
2. **Use VideoUtil**: Always use `VideoUtil` class for FFmpeg operations
3. **IPC for Renderer**: Renderer process must use IPC to request FFmpeg operations
4. **Progress Reporting**: Use VideoUtil's progress callback for long-running operations

---

## File Locations

### VideoUtil Class

**File**: `src/main/utils/video.util.ts`

This is the **ONLY** place where direct FFmpeg execution should occur.

**Available Methods**:
```typescript
class VideoUtil {
  // Get FFmpeg binary path
  static ffmpegPath(): string;

  // Create FFmpeg process with arguments
  static makeFfmpegProcess(args: string[]): FfmpegProcess;

  // Execute FFmpeg process with optional progress callback
  static execFfmpegProcess(
    ffmpeg: FfmpegProcess,
    opts?: { onProgress?: (data: { seconds?: number; speedX?: number }) => void }
  ): Promise<void>;

  // Parse HH:MM:SS.mmm to seconds
  static parseHmsToSeconds(hms: string): number | undefined;

  // Fast probe video duration
  static fastProbeDurationSec(inputPath: string): Promise<number | undefined>;
}
```

### IPC Handlers

**File**: `src/main/services/ipc.service.ts`

FFmpeg-related IPC handlers should be registered in `IpcService.initVideoAPI()`.

### IPC Channels

**File**: `src/types/ipc.type.ts`

Define FFmpeg-related channels in `IpcVideoChannel` enum.

---

## Patterns

### Pattern 1: Execute FFmpeg in Main Process (Service Layer)

**When**: Implementing video processing in main process services

```typescript
// src/main/services/video-processing.service.ts
import { VideoUtil } from '@/main/utils/video.util';

export class VideoProcessingService {
  /**
   * Mux video and audio using FFmpeg
   */
  static async muxVideoAudio(
    videoPath: string,
    audioPath: string,
    outputPath: string,
    onProgress?: (percent: number) => void
  ): Promise<void> {
    // Build FFmpeg arguments
    const args = [
      '-i', videoPath,
      '-i', audioPath,
      '-c', 'copy',
      '-shortest',
      outputPath
    ];

    // Create FFmpeg process
    const ffmpeg = VideoUtil.makeFfmpegProcess(args);

    // Execute with progress reporting
    await VideoUtil.execFfmpegProcess(ffmpeg, {
      onProgress: (data) => {
        if (onProgress && data.seconds) {
          // Convert seconds to percent if you know duration
          const percent = (data.seconds / totalDuration) * 100;
          onProgress(percent);
        }
      }
    });
  }
}
```

### Pattern 2: Expose FFmpeg Operation via IPC

**Step 1**: Define IPC Channel

```typescript
// src/types/ipc.type.ts
export enum IpcVideoChannel {
  // ... existing channels ...
  VideoMuxAudio = 'video-mux-audio',
}
```

**Step 2**: Register IPC Handler

```typescript
// src/main/services/ipc.service.ts
import { IpcVideoChannel } from '@/types/ipc.type';
import { VideoProcessingService } from '@/main/services/video-processing.service';

export class IpcService {
  static init = () => {
    // ... existing inits ...
    IpcService.initVideoAPI();
  };

  private static initVideoAPI = () => {
    // ... existing handlers ...

    ipcMain.handle(
      IpcVideoChannel.VideoMuxAudio,
      async (
        _,
        videoPath: string,
        audioPath: string,
        outputPath: string
      ): Promise<void> => {
        try {
          await VideoProcessingService.muxVideoAudio(
            videoPath,
            audioPath,
            outputPath,
            (percent) => {
              // Send progress to renderer if needed
              mainWindow.webContents.send(
                IpcVideoChannel.VideoExportProgress,
                percent
              );
            }
          );
        } catch (error) {
          console.error('Failed to mux audio:', error);
          throw error;
        }
      }
    );
  };
}
```

**Step 3**: Expose in Preload

```typescript
// src/main/preload.ts
import { IpcVideoChannel } from '@/types/ipc.type';

const videoAPI = {
  // ... existing methods ...

  muxAudio: (
    videoPath: string,
    audioPath: string,
    outputPath: string
  ) => ipcRenderer.invoke(IpcVideoChannel.VideoMuxAudio, videoPath, audioPath, outputPath),
};

contextBridge.exposeInMainWorld('videoAPI', videoAPI);

export type VideoAPI = typeof videoAPI;
```

**Step 4**: Add TypeScript Declaration

```typescript
// src/renderer/preload.d.ts
import {
  VideoAPI,
  // ... other imports
} from '@/main/preload';

declare global {
  interface Window {
    // ... existing ...
    videoAPI: VideoAPI;
  }
}
```

**Step 5**: Use in Renderer

```typescript
// src/renderer/services/video-export.service.ts
export class VideoExportService {
  async muxAudioToVideo(
    videoPath: string,
    audioPath: string,
    outputPath: string
  ): Promise<void> {
    try {
      await window.videoAPI.muxAudio(videoPath, audioPath, outputPath);
      console.log('Audio muxed successfully');
    } catch (error) {
      console.error('Failed to mux audio:', error);
      throw error;
    }
  }
}
```

### Pattern 3: Working with Temporary Files

**When**: FFmpeg needs file paths (not in-memory data)

```typescript
// src/renderer/services/video-export.service.ts
export class VideoExportService {
  /**
   * Convert data URL to temporary file for FFmpeg processing
   */
  private async dataUrlToTempFile(
    dataUrl: string,
    filename: string
  ): Promise<string> {
    const response = await fetch(dataUrl);
    const blob = await response.blob();
    const arrayBuffer = await blob.arrayBuffer();

    // Use IPC to write file in main process
    const tempPath = await window.videoAPI.createTempFile(
      filename,
      Buffer.from(arrayBuffer)
    );

    return tempPath;
  }

  async processVideoWithAudio(
    videoDataUrl: string,
    audioDataUrl: string
  ): Promise<string> {
    // 1. Convert data URLs to temp files
    const videoPath = await this.dataUrlToTempFile(videoDataUrl, 'video.mp4');
    const audioPath = await this.dataUrlToTempFile(audioDataUrl, 'audio.webm');
    const outputPath = await window.videoAPI.getTempPath('output.mp4');

    // 2. Use FFmpeg via IPC
    await window.videoAPI.muxAudio(videoPath, audioPath, outputPath);

    // 3. Cleanup temp files
    await window.videoAPI.deleteTempFile(videoPath);
    await window.videoAPI.deleteTempFile(audioPath);

    return outputPath;
  }
}
```

---

## Anti-Patterns (DON'T DO THIS)

### ❌ Never Execute FFmpeg Directly in Renderer

```typescript
// ❌ WRONG - This will NOT work in renderer process
import { exec } from 'child_process'; // Node.js module not available in renderer!

export class VideoExportService {
  async muxAudio(videoPath: string, audioPath: string) {
    // This will fail - child_process not available in renderer
    exec(`ffmpeg -i ${videoPath} -i ${audioPath} output.mp4`);
  }
}
```

**Why**: Renderer process runs in browser context, not Node.js. No access to `child_process`.

### ❌ Never Import VideoUtil in Renderer

```typescript
// ❌ WRONG - VideoUtil is main process only
import { VideoUtil } from '@/main/utils/video.util';

export class VideoExportService {
  async getFFmpegPath() {
    return VideoUtil.ffmpegPath(); // Will fail at runtime
  }
}
```

**Why**: VideoUtil imports Node.js modules (`child_process`, `fs`, etc.) which are not available in renderer.

### ❌ Never Use require('child_process') in Renderer

```typescript
// ❌ WRONG - Even with require, this is not safe
export class VideoExportService {
  async muxAudio() {
    const { exec } = require('child_process'); // Security risk + doesn't work
    exec('ffmpeg ...');
  }
}
```

**Why**: Even if contextIsolation is disabled, this is a **security vulnerability** and not recommended.

---

## Common FFmpeg Operations

### 1. Mux Video and Audio

```typescript
// Main process
const args = [
  '-i', videoPath,
  '-i', audioPath,
  '-c', 'copy',        // Copy streams without re-encoding
  '-shortest',         // Finish when shortest stream ends
  outputPath
];

const ffmpeg = VideoUtil.makeFfmpegProcess(args);
await VideoUtil.execFfmpegProcess(ffmpeg);
```

### 2. Convert Video Format

```typescript
const args = [
  '-i', inputPath,
  '-c:v', 'libx264',   // Video codec
  '-c:a', 'aac',       // Audio codec
  '-b:v', '2M',        // Video bitrate
  '-b:a', '192k',      // Audio bitrate
  outputPath
];

const ffmpeg = VideoUtil.makeFfmpegProcess(args);
await VideoUtil.execFfmpegProcess(ffmpeg);
```

### 3. Extract Audio from Video

```typescript
const args = [
  '-i', videoPath,
  '-vn',               // No video
  '-acodec', 'copy',   // Copy audio stream
  audioPath
];

const ffmpeg = VideoUtil.makeFfmpegProcess(args);
await VideoUtil.execFfmpegProcess(ffmpeg);
```

### 4. Trim Video

```typescript
const args = [
  '-i', inputPath,
  '-ss', startTime,    // Start time (HH:MM:SS or seconds)
  '-to', endTime,      // End time
  '-c', 'copy',        // Copy streams
  outputPath
];

const ffmpeg = VideoUtil.makeFfmpegProcess(args);
await VideoUtil.execFfmpegProcess(ffmpeg);
```

### 5. Get Video Duration

```typescript
// Use VideoUtil's built-in method
const duration = await VideoUtil.fastProbeDurationSec(videoPath);
console.log(`Video duration: ${duration} seconds`);
```

---

## Progress Reporting

### With Known Duration

```typescript
const totalDuration = await VideoUtil.fastProbeDurationSec(inputPath);

const ffmpeg = VideoUtil.makeFfmpegProcess(args);

await VideoUtil.execFfmpegProcess(ffmpeg, {
  onProgress: (data) => {
    if (data.seconds && totalDuration) {
      const percent = (data.seconds / totalDuration) * 100;
      console.log(`Progress: ${percent.toFixed(1)}%`);

      if (data.speedX) {
        console.log(`Speed: ${data.speedX}x`);
      }
    }
  }
});
```

### With IPC Progress Updates

```typescript
// Main process
ipcMain.handle(IpcVideoChannel.VideoProcess, async (event, inputPath, outputPath) => {
  const totalDuration = await VideoUtil.fastProbeDurationSec(inputPath);

  const ffmpeg = VideoUtil.makeFfmpegProcess(args);

  await VideoUtil.execFfmpegProcess(ffmpeg, {
    onProgress: (data) => {
      if (data.seconds && totalDuration) {
        const percent = (data.seconds / totalDuration) * 100;

        // Send progress to renderer
        event.sender.send(IpcVideoChannel.VideoExportProgress, {
          percent,
          speed: data.speedX
        });
      }
    }
  });
});

// Renderer process
window.electron.ipcRenderer.on(
  IpcVideoChannel.VideoExportProgress,
  (_, data) => {
    console.log(`Progress: ${data.percent}%`);
    updateProgressBar(data.percent);
  }
);
```

---

## Error Handling

### Handling FFmpeg Errors

```typescript
try {
  const ffmpeg = VideoUtil.makeFfmpegProcess(args);
  await VideoUtil.execFfmpegProcess(ffmpeg);

  console.log('FFmpeg operation successful');
} catch (error) {
  console.error('FFmpeg operation failed:', error);

  // Error contains:
  // - message: Error description
  // - ffmpegCommand: The command that was executed
  // - stderr: FFmpeg error output
  // - exitCode: Process exit code

  // Handle specific errors
  if (error.message.includes('Invalid data found')) {
    throw new Error('Invalid video file format');
  } else if (error.message.includes('Permission denied')) {
    throw new Error('No permission to access file');
  } else {
    throw new Error(`FFmpeg error: ${error.message}`);
  }
}
```

---

## Best Practices

### 1. Always Use Absolute Paths

```typescript
// ✅ GOOD
const videoPath = path.resolve('/Users/name/videos/input.mp4');
const outputPath = path.resolve('/Users/name/videos/output.mp4');

// ❌ BAD
const videoPath = './input.mp4'; // Relative paths can cause issues
```

### 2. Quote Paths with Spaces

FFmpeg arguments are automatically escaped by `spawn()`, but be careful with manual string building:

```typescript
// ✅ GOOD - Using array arguments (recommended)
const args = ['-i', '/path/with spaces/video.mp4', 'output.mp4'];
const ffmpeg = VideoUtil.makeFfmpegProcess(args);

// ❌ BAD - Manual string with spaces
const command = `ffmpeg -i /path/with spaces/video.mp4 output.mp4`; // Will fail
```

### 3. Cleanup Temporary Files

```typescript
let tempVideoPath: string | null = null;
let tempAudioPath: string | null = null;

try {
  tempVideoPath = await createTempFile('video.mp4', videoData);
  tempAudioPath = await createTempFile('audio.webm', audioData);

  await window.videoAPI.muxAudio(tempVideoPath, tempAudioPath, outputPath);

} finally {
  // Always cleanup, even if operation fails
  if (tempVideoPath) await deleteTempFile(tempVideoPath);
  if (tempAudioPath) await deleteTempFile(tempAudioPath);
}
```

### 4. Validate File Existence Before Processing

```typescript
// Main process
if (!fs.existsSync(inputPath)) {
  throw new Error(`Input file not found: ${inputPath}`);
}

const stats = await fs.promises.stat(inputPath);
if (stats.size === 0) {
  throw new Error('Input file is empty');
}

// Proceed with FFmpeg operation
```

### 5. Use `-y` Flag for Overwrite

```typescript
const args = [
  '-y',              // Overwrite output file without asking
  '-i', inputPath,
  // ... other args ...
  outputPath
];
```

---

## Checklist for FFmpeg Operations

When implementing a new FFmpeg operation:

- [ ] Operation implemented in main process (NOT renderer)
- [ ] Uses `VideoUtil.makeFfmpegProcess()` and `VideoUtil.execFfmpegProcess()`
- [ ] IPC channel defined in `IpcVideoChannel` enum (if accessed from renderer)
- [ ] IPC handler registered in `IpcService.initVideoAPI()`
- [ ] Method exposed in `preload.ts` as `videoAPI` object
- [ ] TypeScript type exported (`export type VideoAPI = typeof videoAPI`)
- [ ] TypeScript declaration added in `preload.d.ts`
- [ ] All file paths are absolute
- [ ] Temporary files are cleaned up (in finally block)
- [ ] Error handling implemented
- [ ] Progress reporting added for long operations
- [ ] File existence validated before processing

---

## Security Considerations

1. **Never Trust User Input**: Always validate and sanitize file paths
2. **Use Spawn, Not Exec**: VideoUtil uses `spawn()` which is safer than `exec()`
3. **Limit File Access**: Only allow FFmpeg to access specific directories
4. **Validate File Types**: Check MIME types before processing

```typescript
// Validate file extension
const allowedExtensions = ['.mp4', '.webm', '.mov', '.avi'];
const ext = path.extname(inputPath).toLowerCase();

if (!allowedExtensions.includes(ext)) {
  throw new Error(`Unsupported file type: ${ext}`);
}
```

---

## Common Gotchas

### 1. FFmpeg Binary Not Found

**Symptom**: Error "ffmpeg: command not found" or "ENOENT"

**Solution**: Ensure `@ffmpeg-installer/ffmpeg` is installed and properly bundled:

```json
// package.json
{
  "dependencies": {
    "@ffmpeg-installer/ffmpeg": "^1.1.0"
  }
}
```

### 2. Permission Issues

**Symptom**: Error "Permission denied" when accessing files

**Solution**: Check file permissions and ensure app has necessary access rights (especially on macOS)

### 3. Output File Not Created

**Symptom**: FFmpeg completes but output file doesn't exist

**Solution**: Check stderr for errors, ensure output directory exists:

```typescript
const outputDir = path.dirname(outputPath);
await fs.promises.mkdir(outputDir, { recursive: true });
```

### 4. Audio/Video Out of Sync

**Symptom**: Audio and video drift apart after muxing

**Solution**: Use `-async 1` flag or re-encode audio:

```typescript
const args = [
  '-i', videoPath,
  '-i', audioPath,
  '-c:v', 'copy',
  '-c:a', 'aac',     // Re-encode audio
  '-async', '1',     // Sync audio to video
  outputPath
];
```

---

## Examples from Codebase

### Example 1: Video Export Service

See `src/main/services/video-export.service.ts` for comprehensive example of FFmpeg usage in main process.

### Example 2: Video Stream Export Service

See `src/main/services/video-stream-export.service.ts` for streaming video processing with FFmpeg.

---

## Summary

- ✅ **DO**: Use VideoUtil in main process
- ✅ **DO**: Expose operations via IPC for renderer access
- ✅ **DO**: Use spawn with array arguments
- ✅ **DO**: Report progress for long operations
- ✅ **DO**: Cleanup temporary files
- ❌ **DON'T**: Execute FFmpeg in renderer process
- ❌ **DON'T**: Import VideoUtil in renderer code
- ❌ **DON'T**: Use require('child_process') in renderer
- ❌ **DON'T**: Trust user input without validation
