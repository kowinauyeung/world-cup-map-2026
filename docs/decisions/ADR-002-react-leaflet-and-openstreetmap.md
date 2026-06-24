# ADR-002: Use React Leaflet with OpenStreetMap tiles for the venue map

- **Status:** Accepted
- **Date:** 2026-06-24

## Context

The product needs an interactive world/continental map with custom venue markers, popups, and programmatic focus on a selected match. The implementation should remain compatible with a static Vite application and should not require a map-service API key.

## Decision

Use `leaflet` with `react-leaflet` and the standard OpenStreetMap tile layer. Add `@types/leaflet` as a development dependency. The map shall include visible OpenStreetMap attribution and preserve Leaflet attribution controls.

Render one marker per venue coordinate for the active filter. Group matches by coordinate and venue before rendering markers. The selected match must use a visually distinct marker or popup state, and selecting a match must call the map's `flyTo`/`setView` through a small map-controller component.

Keep Leaflet-specific code behind a `VenueMap` component and a narrow marker-group interface so unit/component tests can mock the map instead of requesting tiles.

## Consequences

- No map API key or backend is required.
- Production map rendering depends on the availability and usage policy of the selected tile provider; attribution is mandatory.
- Leaflet CSS must be imported once and marker icon asset handling must be verified in the Vite build.
- Map interaction needs targeted integration tests; business logic remains testable without Leaflet.

## Alternatives considered

- **Google Maps / Mapbox:** rejected because a public API key, billing, and vendor setup are unnecessary for this first release.
- **A static image:** rejected because it cannot meet marker selection and map-focus requirements.
- **Hand-written SVG map:** rejected because accurate geographic projection and interaction would create unnecessary implementation risk.
