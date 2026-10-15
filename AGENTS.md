# Working in this repo

## What this app is
A single self-contained static HTML file: `sandboxels.html` (the "Sandboxels"
falling-sand game, one file with all its JS inline). There is **no build step,
no backend, no database, and no credentials or environment variables** required.

`sandboxels.html` declares `<base href="https://cdn.jsdelivr.net/gh/R74nCom/sandboxels@main/">`,
so its stylesheet, icons, manifest, lang files and mods are fetched from that
CDN at runtime rather than from this repo. The game therefore needs network
access in the browser to look/behave like the real thing; the simulation logic
itself is all inline and works regardless.

## Running it
```sh
docker compose -f docker-compose.base44.yml up -d      # serves on host port 3000
curl -s http://localhost:3000/ | head -c 200           # sanity check: the game HTML
docker compose -f docker-compose.base44.yml logs web   # nginx access/error log
```
`web` is a stock `nginx:alpine` serving this checkout read-only from `/site`.
`.base44/nginx.conf` is mounted as the **whole** `/etc/nginx/nginx.conf`
(not just a conf.d snippet) because it must set `user root` — the sandbox
checkout is mode `0700`, so nginx's default worker user cannot traverse the
mount. Without that directive the server answers 403/500 ("Permission denied").

There is **no hot reload**: the page is static, so after editing
`sandboxels.html` reload the preview (nginx reads from disk per request; no
container restart or rebuild needed).

## Verifying it works
Load `/`, then:
- the material palette has ~320 `[id^="elementButton-"]` buttons and the
  `<canvas id="game">` is visible (initially `#canvasDiv` starts `display:none`
  and is revealed by the script);
- the stats line (`#stats`) shows `Pxls:0`, `30tps` and a pixel-counter that
  ticks up;
- drawing works: hover over the canvas (`mousemove`) *then* press (`mousedown`)
  and drag. The game only learns the pointer position on a *hover* move, so a
  press without a preceding move places nothing at the expected spot.
- the game reads canvas coordinates from the real mouse events on `#game`
  (`mouseClick`/`mouseMove` at the input-wiring script); it ignores nothing and
  needs no keys, so a synthetic `MouseEvent` sequence is a faithful gesture.

## Non-obvious quirks
- `sandboxels.html` contains a guard that **hides the game canvas** when the page
  is embedded in an iframe whose ancestor origin contains `game`, `browser`,
  `yizhif` or `.io`, and instead shows a "WARNING / use the main website" menu.
  If the preview ever shows only that warning (or a blank black page) instead of
  the game, this origin check is why — it is upstream behaviour, not a serving
  problem.
- The game writes to `localStorage` (`SandboxelsTempSave`, enabled mods,
  settings); that is per-browser and needs no setup.
- The upstream PWA registration does not work with this file's CDN `<base>`:
  `navigator.serviceWorker.register('/service-worker.js')` resolves to the CDN
  origin and rejects with a `SecurityError` (and no `service-worker.js` sits
  beside `sandboxels.html`). That call is wrapped in a `try`/`.catch` so a load
  no longer raises an uncaught error; don't remove the guard.
- External requests (Google Analytics/AdSense, `news.r74n.com`, Playlight SDK)
  are the upstream page's own; they are not needed for the game to run and
  there is no API key to supply.
