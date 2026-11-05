---
name: testing-games-page
description: How to serve and test the bacongames static site locally, especially the /games/ library page (cover images, search, Secret card).
---

# Testing the bacongames static site

## Serving
Pure static site, no build step or dependencies:

```bash
cd <repo root> && python3 -m http.server 8123
# open http://localhost:8123/games/
```

Nav links are relative (`./games/`, `../info/`, ...), so they work on localhost, on any fork's
GitHub Pages project site, and on a custom domain.

## games/index.html behaviour worth knowing
- The card list paints immediately from the baked-in `GAMES` array (cover art comes from the
  optional `COVERS` map), then refreshes from the GitHub contents API of the repo serving the
  page (`<owner>.github.io/<repo>/`, default PeptidesDev/bacongames) and caches the result in
  `localStorage['bacongames_list_cache_v2']` for 30 min. Clear that key for a clean run.
  Keep `GAMES` in sync when adding or removing a game folder.
- The GitHub API call is often rate-limited (HTTP 403) from CI/agent IPs; the page logs
  `Failed to refresh games list:` and keeps the baked-in list. This console error is expected
  and not a regression.
- The `Secret` game is only rendered when the search box contains exactly `secret`
  (case-insensitive). It must NOT show with an empty search — verify visually between
  `Sandtris` and `Slope` in the alphabetical grid.
- Search is debounced ~120ms; wait a moment before asserting.
- Thumbnails are `<img loading="lazy">` layered over a text `.fallback` div; the fallback is
  hidden on `onload` and the img hidden on `onerror`. So a card showing the game name as text
  means its cover URL failed.

## Counting broken cover images
Scroll the whole page first (lazy loading), then evaluate in the page:

```js
[...document.querySelectorAll('.game-card img')]
  .filter(i => i.complete && i.naturalWidth === 0).map(i => i.alt)   // broken
[...document.querySelectorAll('.game-card img')].filter(i => !i.complete).map(i => i.alt) // not yet fetched
```

Third-party hotlinks (ignimgs.com, poki, crazygames, ...) can fail transiently when ~200
images load at once; re-check with a hard reload before reporting a URL as dead.

## Gotcha: DevTools/CDP access after restarting Chrome
If Chrome is relaunched manually, the managed `browser_console` tool may report
"Could not connect to Chrome via CDP" even though `http://localhost:9222/json/version` works.
Workaround — drive CDP directly:

```bash
pip install websocket-client
# connect to the page's webSocketDebuggerUrl with websocket.create_connection(url, suppress_origin=True)
# then send {"method":"Runtime.evaluate","params":{"expression":..., "returnByValue":true}}
```
`suppress_origin=True` is required or Chrome rejects the handshake with 403.

## Devin Secrets Needed
None — the site is fully static and public.
