# ADR-001: Use a static client-side React application with local schedule data

- **Status:** Accepted
- **Date:** 2026-06-24

## Context

The project already uses Vite, React, and TypeScript. The schedule is versioned in `data.json`; localized stadium profiles are also static, curated content. The requested experience is read-only. There is no requirement for accounts, server-side schedule updates, or live scores.

## Decision

Build a static single-page React application. Add typed loaders that import `data.json` and the venue-profile content dataset, validate and normalise each once, then expose derived selectors to the UI. Enable TypeScript `resolveJsonModule` if required by the import path.

Keep UI state in the root feature composition as two values:

```ts
selectedDate: string | null
selectedMatchId: number | null
```

Derive filtered matches, venue groups, calendar counts, and selected-match details from the normalised match collection. Do not store these derivations separately.

Resolve the browser locale at application bootstrap and provide it as immutable application configuration, not as interaction state. The resolved locale selects static UI messages and venue-profile content.

## Consequences

- The application can deploy as static files with no application backend.
- Schedule or venue-profile content changes require a repository change and redeployment.
- The bundle includes small, versioned schedule, locale, and venue-profile data; this is acceptable for 104 records and 16 venues.
- Typed loaders are responsible for validation, which prevents inconsistent schedule, coordinate, locale, and profile parsing across components.

## Alternatives considered

- **Backend/API:** rejected because the data is static and it adds operational work without a product need.
- **Parsing data inside components:** rejected because it duplicates validation and creates divergent behaviour.
- **Persisting derived filters in state:** rejected because it risks stale UI after selection changes.
