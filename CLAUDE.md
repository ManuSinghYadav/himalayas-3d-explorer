# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A single-file (`index.html`) 3D Himalayan map explorer. No build step, no dependencies to install — open the file directly in a browser.

## Running

```bash
open index.html
# or serve locally if needed:
python3 -m http.server 8000
```

## Architecture

Everything lives in `index.html` — HTML, CSS, and JS are all inline. The file is structured in this order:

1. **Head** — Mapbox GL JS v3.3.0 + Mapbox Geocoder v5.0.0 loaded from CDN
2. **CSS** — All styles inline in `<style>`. Key sections: token modal, trek panel (collapsible left sidebar), search bar (top-center), stats bar (bottom-center), popup overrides
3. **HTML** — Token modal → map div → header → trek panel → search container → stats bar
4. **JS** — One large `<script>` block with these logical sections (marked by `// ═══` comments):
   - `TREK DATA` — `TREKS` object: 8 famous routes, each with `trail` (array of `[lng, lat]` coords), `waypoints` array, color, metadata
   - `MAP DATA` — GeoJSON `FeatureCollection` constants: `PEAKS`, `VILLAGES`, `GLACIERS`, `RIVERS`, `BORDERS`
   - `STATE` — `activeTreks` Set, `lastActiveTrek`, `map` instance
   - `buildTrekList()` — populates trek sidebar from `TREKS` object on page load
   - `initMap()` — called on token submit; creates Mapbox map, attaches geocoder, registers `load` handler that adds all sources/layers/popups
   - `addTrekLayers(id, t)` / `removeTrekLayers(id)` — dynamically add/remove 6 layers per trek (casing, line, dash, wp-ring, wp-dot, wp-label)
   - `toggleTrek(id)` — activates/deactivates a trek, updates info card
   - `toggleTrekPanel()` — CSS class toggle (`open`) on `#trek-panel` drives expand/collapse via `max-height` transition
   - `updateInfoCard()` / `flyToActive()` — update the info card and fly camera to last active trek

## Key Conventions

- **Mapbox token** is entered at runtime via the modal; never hardcoded. Token is stored only in `mapboxgl.accessToken`.
- **Trek layers** follow the naming pattern `trek-{type}-{id}` (e.g. `trek-line-ebc`, `trek-wp-dot-ebc`). Sources follow `trek-trail-{id}` and `trek-wp-{id}`.
- **All map layers** (peaks, rivers, borders, villages, glaciers) are always visible — there are no runtime toggles for them.
- **Terrain exaggeration** default is `1.5`. Mapbox DEM source id is `dem`.
- **Scroll zoom** is tuned: `setWheelZoomRate(1/200)` and `setZoomRate(1/80)` for faster navigation than Mapbox defaults.
- **Style**: `mapbox://styles/mapbox/satellite-streets-v12` (includes OSM roads, labels, waterways on top of satellite imagery).
