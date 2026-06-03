# Outbound data and third-party exposure

Shadowbroker is **self-hosted**: each install uses its own backend egress IP (and optional `OPERATOR_HANDLE` in `User-Agent`). This documents intentional third-party contact for audit issues #348–#366.

## Architecture

| Path | Who calls third parties |
|------|-------------------------|
| UI → `/api/*` → fetchers | **Backend** |
| Map basemap tiles/fonts | **Browser** (CARTO, demotiles.maplibre.org) |
| CCTV proxy | **Backend** (with upstream-required `Referer` / `Origin`) |

## Ukraine frontline mirror (#362)

- **Layer:** `ukraine_frontline` → `frontlines` on the map (DeepStateMap polygons). **Not** UAP (`uap_sightings` / NUFORC).
- **Code:** `backend/services/geopolitics.py`
- **Default:** `cyterat/deepstate-map-data` @ `main`, latest `data/deepstatemap_data_*.geojson`
- **Pin:** `DEEPSTATE_MIRROR_COMMIT=<sha>` — immutable Git snapshot; bump SHA when you want newer lines
- **Optional:** `DEEPSTATE_MIRROR_REPO=owner/repo`

## Madrid CCTV (#363)

- **Ingest:** HTTPS-first KML on `datos.madrid.es` (catalog only); HTTP fallback if needed
- **Feeds:** Still images from URLs inside the KML (`informo.madrid.es`, etc.), proxied with `Referer: https://informo.madrid.es/` — unchanged by KML transport

## KiwiSDR (#364)

- HTTPS first, then HTTP; shape validation + bundled `backend/data/kiwisdr_directory.json`

## Other documented exposures

- **#354 Basemap:** browser → `*.basemaps.cartocdn.com`, `demotiles.maplibre.org`
- **#349 CCTV Referer:** required for many DOT/city streams; backend proxy only
- **#361 Operator UA:** `OPERATOR_HANDLE` / `outbound_user_agent()` per install
- **#366 Broadcastify:** backend scrape with honest UA
- **#348 LiveUAMap:** `SHADOWBROKER_ENABLE_LIVEUAMAP_SCRAPER` (default on Linux, off Windows)

## Operator checklist

1. Set `OPERATOR_HANDLE` if you want a recognizable contact on upstream logs.
2. Pin `DEEPSTATE_MIRROR_COMMIT` after reviewing a mirror commit (see `backend/.env.example`).
3. Set `SHADOWBROKER_ENABLE_LIVEUAMAP_SCRAPER=false` to disable LiveUAMap contact.
4. Self-host map tiles if basemap CDN exposure matters.
