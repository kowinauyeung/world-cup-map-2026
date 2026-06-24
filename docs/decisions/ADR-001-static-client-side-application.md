# ADR-001: Use a static client-side React application with local schedule data

- **Status:** Accepted
- **Date:** 2026-06-24

## Context

The project already uses Vite, React, and TypeScript. The complete schedule is versioned in `data.json`, and the requested experience is read-only. There is no requirement for accounts, server-side schedule updates, or live scores.

## Decision

Build a static single-page React application. Add a typed loader that imports `data.json`, validates and normalises it once, then exposes derived selectors to the UI. Enable TypeScript `resolveJsonModule` if required by the import path.

Keep UI state in the root feature composition as two values:

```ts
selectedDate: string | null
selectedMatchId: number | null
```

Derive filtered matches, venue groups, calendar counts, and selected-match details from the normalised match collection. Do not store these derivations separately.

## Consequences

- The application can deploy as static files with no application backend.
- Schedule changes require a repository change and redeployment.
- The bundle includes a small, versioned data file; this is acceptable for 104 records.
- One loader is responsible for validation, which prevents inconsistent coordinate and time parsing across components.

## Alternatives considered

- **Backend/API:** rejected because the data is static and it adds operational work without a product need.
- **Parsing data inside components:** rejected because it duplicates validation and creates divergent behaviour.
- **Persisting derived filters in state:** rejected because it risks stale UI after selection changes.
