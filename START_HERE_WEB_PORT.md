# Start here: browser port

1. Read `WEB_PERSONAL_EDITION.md`.
2. Run `npm ci` and `npm run dev` to confirm the existing frontend works.
3. Inspect `src/features/clipper/pages/clipper-tauri-gate.component.tsx`.
4. Trace Clipper feature service calls into `src-tauri/src/commands/`.
5. Implement the local API described in `docs/web-personal/LOCAL_API_CONTRACT.md`.
6. Do not remove the Tauri gate until the web adapter supports the full workflow.

This ZIP is intentionally a safe porting baseline, not a falsely advertised finished browser
clipper. The desktop implementation is preserved intact.
