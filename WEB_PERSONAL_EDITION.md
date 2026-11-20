# OpenClipper Web – Personal Edition (porting workspace)

This is a **web-porting workspace** based on the supplied OpenClipper source tree.
It is not yet a completed browser-only Clipper. The original desktop implementation is
preserved so its working native processing pipeline is not accidentally broken.

## Current state

- The repository already has a Vite web build (`npm run build:web`).
- The React frontend can be served in a normal browser.
- The Clipper feature is deliberately guarded by `ClipperTauriGate`.
- Important processing and file-system operations are implemented through Tauri/Rust commands.
- Therefore, a successful web build does **not** mean the full Clipper workflow works in a browser.

## Recommended personal architecture

Keep the React frontend and the Rust processing code. Run a small local service on the same
Windows PC and have the browser communicate with it over `http://127.0.0.1:<port>`.

This keeps heavy video processing local and avoids cloud hosting/API charges. It is technically
a browser UI plus a local companion service, not a pure browser-only app.

### Porting phases

1. **Inventory native calls**: identify every `@tauri-apps/api`, plugin, and `invoke()` usage
   in the Clipper feature and its shared services.
2. **Define a local HTTP API**: replace each required Tauri command with an authenticated
   localhost endpoint. Keep the existing Rust implementation behind those endpoints where
   practical.
3. **File handling**: replace native path assumptions with upload/session IDs and local service
   file references. Never expose arbitrary filesystem paths to browser requests.
4. **Media URLs**: serve preview assets from a restricted local media endpoint.
5. **Models**: keep model download/cache logic local; verify each model's separate licence.
6. **Browser adapter**: add a `web` implementation for the Clipper services and retain the
   existing Tauri adapter for desktop.
7. **Enable the web gate only after** project creation, transcription, reframing, rendering,
   export, and project reload all work through the local service.
8. **Regression tests**: test large uploads, cancellation, interrupted jobs, Unicode filenames,
   long recordings, and GPU fallback.

## Run the existing web frontend

Requirements: Node.js LTS and npm.

```powershell
npm ci
npm run dev
```

Then open the localhost URL printed by Vite.

To build the existing web frontend:

```powershell
npm run build:web
```

**Important:** these commands run/build the existing web frontend only. The Clipper feature is
still blocked by its Tauri gate until the local-service adapter is implemented.

## Important source files to start from

- `src/features/clipper/pages/clipper-tauri-gate.component.tsx`
  - Explicitly blocks Clipper outside Tauri. Do not simply remove this gate: downstream native
    commands would fail.
- `src/features/clipper/`
  - Clipper UI, hooks, and feature services.
- `src/shared/utils/platform.util.ts`
  - Platform detection.
- `src-tauri/src/commands/`
  - Native command implementations that need to be exposed via the local service.
- `src-tauri/src/`
  - Rust processing, model, media, and persistence implementation.
- `vite.config.ts`
  - Existing web build configuration.
- `package.json`
  - `build:web` is already present.

## Security for personal localhost use

- Bind the service only to `127.0.0.1`, not `0.0.0.0`.
- Require a random per-run token for mutating endpoints.
- Validate `Origin` and reject unexpected hosts.
- Restrict file operations to a dedicated application data directory.
- Do not accept arbitrary command lines or arbitrary filesystem paths from the browser.
- Set upload size limits and clean up temporary files.
- Do not expose the local service to the public internet.

## Licence notes

The supplied repository includes its own licence file. Preserve required notices when modifying
or redistributing the source. Check third-party libraries, downloaded models, and model weights
under their own licences; the repository's licence does not automatically cover them.
