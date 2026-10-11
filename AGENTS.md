# Project notes

## What this repo is

A single static page, the extension-less file `8424` — a "Milestone Satisfaction
Survey" built with inline HTML/CSS/JS. No framework, no build step, no backend,
no database, no required environment variables.

## Running it (Base44 sandbox)

- `docker compose -f docker-compose.base44.yml up -d`
- Entry point: host port **3000** → nginx (`nginx:alpine`) on port 80.
- `8424` is bind-mounted at `/usr/share/nginx/html/index.html`, so `/` serves it
  with the correct `text/html` type (the extension-less file would otherwise be
  downloaded rather than rendered). nginx's stock config does the rest — no
  custom config file needed.
- Only that single file is mounted, deliberately: the repo directory is mode
  `700` on the sandbox host, and nginx's unprivileged worker cannot traverse it,
  so a directory bind mount yields `403 Forbidden` for every request. The file
  itself is world-readable, so mounting it alone works.
- Consequence: if extra asset files (css/js/images) are added later, they need
  their own read-only volume entries in `docker-compose.base44.yml`.
- Because the file is served straight from the bind mount, **edits to `8424`
  show up on a plain page reload** — no rebuild and no `reload_preview` needed.
  Only compose/volume changes need a container restart.
- `BASE44_PREVIEW_MODE` is intentionally not wired into any service: there is no
  application code that would branch on it.

## Verify it works

```
curl -s http://localhost:3000/ | grep -c 'Milestone Satisfaction Survey'
docker compose -f docker-compose.base44.yml ps
```

The compose healthcheck runs the same content probe inside the container.

## Known gap (not a setup issue)

The submit handler in `8424` POSTs to a Google Apps Script Web App placeholder:
`https://script.google.com/macros/s/YOUR-GOOGLE-SCRIPT-URL/exec`. Until that URL
is replaced with a real Apps Script deployment, the page renders and the form
works but submitting fails. The URL is hardcoded in the page, not an environment
variable, so no secret needs to be configured for it.
