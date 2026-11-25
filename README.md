# Roof Measure (static MVP, $0)

Type an address, get roof squares. No server, no build step, no keys, no Google. Everything runs in the browser.

Live site (GitHub Pages): https://jorellano.github.io/roof-measure/

## Files
- `index.html` (UI + CSS), `app.js` (logic)
- `vendor/pmtiles.js` (PMTiles reader, BSD-3, protomaps), `vendor/laz-perf.js` + `laz-perf.wasm` (LAZ decoder, Apache-2.0, hobu)
- `data/ept_index.json`: bounds of the ~2,300 USGS 3DEP lidar projects, so the right one is chosen with no extra request

## Run locally
    python3 -m http.server 8765
    open http://localhost:8765/?a=21050%20Kimberly%20Ct,%20Lake%20Elsinore,%20CA%2092532
(`?a=` auto-runs an address; opening index.html as file:// will not work because of the wasm/fetch rules.)

## Pipeline (all free, CORS-enabled, called straight from the browser)
1. Geocode: Nominatim (OSM), fallback Photon (komoot). The Census geocoder sends no CORS header, so a browser cannot call it.
2. Parcel: San Diego County GIS `cosd_warehouse/parcels_all_for_public_use` and Riverside County RCIT `parcels/Parcels` (house number + street, ZIP to break ties).
3. Outline: Overture Maps buildings PMTiles (`tiles.overturemaps.org`, z14 vector tile, HTTP range requests, about 1-2 MB) plus a live OSM Overpass query (8 s timeout; often busy).
   The building that sits inside the parcel wins. Outlines sourced from Google Open Buildings are skipped. With no parcel it falls back to the nearest building and says "check this outline".
4. Pitch: defaults to 5/12, labeled "assumed". One tap on "Measure with lidar" reads USGS 3DEP lidar (public S3, about 3-4 MB, 2-5 s) and fits pitch for each roof face.
5. Squares = plan area x pitch factor x (1 + waste) / 100. Waste is 0, 10 or 15%.

## Limits
- Parcel matching only covers San Diego and Riverside counties. Add others to `PARCELS` in app.js (any ArcGIS parcel layer that sends CORS headers).
- Overture tiles are quantized to about 0.5 m, so plan area runs 0.3-1.3% above the Python numbers. An outline cut at a tile edge gets flagged.
- Lidar is 2014 for Chula Vista and 2018 for Lake Elsinore, so newer roofs or additions will not show.
- Nominatim's usage policy asks for light use (about 1 request per second, no bulk). That is fine for an MVP; switch to Photon or a self-hosted geocoder if traffic grows.
