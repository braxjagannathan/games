# Working in this repository

## What this app is

The whole app is one static file: `sandboxels.html` (a copy of the Sandboxels
falling-sand game). There is no build step, no package manager, no backend and
no tests. `README.md` is a placeholder.

Note the `<base href="https://cdn.jsdelivr.net/gh/R74nCom/sandboxels@main/">`
tag in the page head: every relative URL in the file (icons, manifest, mods,
language files) resolves against that CDN, not against this repository. So the
page renders fully even though only the HTML file is tracked here.

## Running it (Base44 sandbox)

```sh
docker compose -f docker-compose.base44.yml up -d --build
```

`web` is a plain `nginx:alpine` container that serves the bind-mounted working
tree on host port 3000 (`.base44/nginx.conf` sets `index sandboxels.html`, so
`/` serves the game, and denies dot paths so `.git` is never served). Because
the tree is bind-mounted, an edit to `sandboxels.html` is live after a page
reload — no rebuild, no restart.

## Verifying it works

```sh
curl -s http://localhost:3000/ | grep -i '<title'
docker compose -f docker-compose.base44.yml ps
```

A non-200, or a page without the `<title>` tag, means nginx is not serving the
game. There is nothing else to check: the app is client-side only.

## Deployment

No deployment configuration exists in this repository (no CI, no Dockerfile, no
hosting manifest). Publishing is a decision for the repository owner; see the
conversation for the options discussed.
