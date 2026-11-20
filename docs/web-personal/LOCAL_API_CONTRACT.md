# Proposed local API contract

This is a design contract for the browser port, not an implemented server.

Base URL: `http://127.0.0.1:<ephemeral-port>/api/v1`

The companion service should bind to loopback only and issue a random session token at startup.
The browser should obtain the token through a same-origin bootstrap page or a one-time local
pairing step. Never put the token in a URL.

## Initial endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/health` | Service version and readiness |
| GET | `/capabilities` | Available transcription/rendering backends |
| POST | `/jobs` | Create a processing job from an uploaded source |
| POST | `/jobs/{id}/analyze` | Run transcription/highlight analysis |
| GET | `/jobs/{id}` | Read job state and progress |
| GET | `/jobs/{id}/events` | Server-sent progress events |
| GET | `/jobs/{id}/clips` | List generated clip candidates |
| POST | `/jobs/{id}/clips/{clipId}/render` | Render selected clip |
| GET | `/media/{opaqueId}` | Stream a preview/export asset |
| DELETE | `/jobs/{id}` | Cancel and clean up a job |

## Data rules

- Use opaque IDs, not local paths, in browser-visible JSON.
- Accept uploads as multipart form data with strict size limits.
- Validate file type by inspecting content, not only filename extension.
- Store all job files below one application-owned data directory.
- Make job creation idempotent where possible.
- Report progress as structured states (`queued`, `transcribing`, `analyzing`,
  `reframing`, `rendering`, `complete`, `failed`, `cancelled`).
- Return structured errors with stable error codes.
- Add cancellation and cleanup semantics before enabling batch processing.

## First vertical slice

Implement and test in this order:

1. `GET /health`
2. Upload a short test video and create a job
3. Generate a transcript using the existing local transcription code
4. Return timestamped transcript segments
5. Return manually selected clip ranges
6. Render one clip with the existing FFmpeg pipeline
7. Serve the result to the browser
8. Only then connect AI highlight selection and batch processing
