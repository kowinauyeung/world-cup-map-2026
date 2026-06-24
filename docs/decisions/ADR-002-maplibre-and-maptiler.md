# ADR-002: Use MapLibre GL JS with a branded MapTiler vector-map style

- **Status:** Accepted
- **Date:** 2026-06-24

## Context

The map is a primary visual surface of this product, not merely a location picker. The design requires a fully branded basemap: colour palette, label density, points of interest, roads, land, and water must be controllable. The initial Leaflet and standard OpenStreetMap raster-tile approach can style markers and popups, but cannot precisely style those basemap layers.

The application remains a static Vite site and must not embed a private server-side credential.

## Decision

Use `maplibre-gl` as the WebGL vector-map renderer. Use a branded vector-map style created and hosted in MapTiler Cloud, supplied to the application through the public build-time variable `VITE_MAP_STYLE_URL`.

Create a MapTiler browser token limited to the production and local-development origins, with no administrative privileges. The token is intentionally public because Vite embeds `VITE_*` values in client code; it must be restricted by allowed origin and monitored for usage. Do not commit a real style URL or token. Commit `.env.example` with an empty placeholder only.

Map requirements:

- Render one selectable venue representation per venue coordinate in the active filter, grouping matches by coordinate and venue. Its visual form (DOM marker, SVG, or MapLibre layer) is a UI-design decision.
- Apply selected, Day, Night, and TBC states through the chosen presentation. If a canvas map layer is used, provide an equivalent keyboard-accessible venue-selection control.
- Keep the MapLibre instance and its `flyTo` behaviour inside `VenueMap`. It receives typed venue-marker groups and selection callbacks, so domain/component tests can replace it with a test double.
- Keep the map provider's required attributions visible.

## Consequences

- The product can have a distinct tournament visual system across the basemap and application overlays.
- No application backend is required, but a MapTiler account, browser token, and style configuration are required before production deployment.
- The style is an external design asset. Its style URL, allowed origins, usage limits, and attribution must be recorded in deployment configuration.
- Map rendering requires WebGL. The calendar and accessible match list remain the complete non-map interaction path if a browser cannot render the map.
- Map interaction requires targeted integration/manual tests; filter and selection logic remain testable without MapLibre or network requests.

## Alternatives considered

- **React Leaflet + standard OpenStreetMap raster tiles:** rejected because it cannot deliver a fully branded basemap; CSS filters cannot independently control roads, water, labels, and POIs.
- **Mapbox GL:** rejected because MapLibre provides the required vector rendering without locking the renderer to a commercial SDK. A tile/style provider is still selected separately.
- **A static image or hand-written SVG world map:** rejected because it cannot meet interactive marker, map-focus, projection, and responsive requirements.
