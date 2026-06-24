# Implementation architecture

This is the implementation contract for Claude Code. Follow the product requirements and accepted ADRs; do not substitute libraries or change time semantics without updating the relevant ADR.

## Dependencies to add

```bash
yarn add leaflet react-leaflet react-day-picker
yarn add --dev @types/leaflet vitest jsdom @testing-library/react @testing-library/user-event
```

Add the smallest compatible Vitest configuration for this Vite project and a `test` script. Do not add a state-management library, date-time library, backend client, or CSS framework for this scope.

## Proposed module boundaries

```text
src/
  domain/
    tournament.ts          # Raw and normalised types
    normalizeTournament.ts # parsing, validation, UTC and Day/Night helpers
    selectors.ts           # date counts, filters, venue grouping
  data/
    tournament.ts          # imports data.json and exports normalised matches
  features/
    calendar/
      MatchCalendar.tsx
    matches/
      MatchList.tsx
      MatchDetails.tsx
    map/
      VenueMap.tsx
      MapController.tsx
  App.tsx                  # selectedDate and selectedMatchId only
  App.css
```

Exact filenames may differ, but preserve these boundaries:

- **Domain** is pure TypeScript and must not import React or Leaflet.
- **Data** imports, validates, and normalises the static JSON once.
- **Feature components** receive typed props and emit intent through callbacks; they do not access raw JSON.
- **`App`** is the sole owner of selection state and derives all filtered values.
- **`VenueMap`** receives already-grouped markers and is the only Leaflet-aware feature boundary.

## State and event flow

```text
calendar select/clear ─┐
match list select ─────┼─> App: selectedDate + selectedMatchId
map marker activate ───┘              │
                                      ├─> derived matches by date
                                      ├─> derived venue marker groups
                                      └─> selected MatchDetails + map focus
```

Required transitions:

1. On calendar select: set `selectedDate`; set `selectedMatchId` to `null`.
2. On calendar clear: set both values to `null`.
3. On match selection: keep the current date and set that match ID.
4. On a single-match marker: select that match.
5. On a multi-match marker: expose an in-context list; set an ID only after the user chooses one.
6. If filtering means the selected ID is no longer present, clear it.

## Rendering rules

- Calendar days outside the dataset's range must not be selectable.
- With no selected date, map all venue groups from all matches; list may be a concise prompt rather than a 104-row list.
- With a selected date, render only that day's matches and their venue groups.
- One venue marker may represent multiple matches. Its popup/list must show only matches in the current filtered collection.
- Render result only if `result !== null`.
- Use an accessible HTML match list and details panel; do not make the map the sole interaction path.
- Import Leaflet CSS once, and confirm default marker assets render in the production build.

## Time helper contract

Expose pure helpers with behaviour equivalent to:

```ts
getUtcInstant(match: Match): Date | null
getDayNight(localTime: string | null): 'day' | 'night' | 'tbc'
formatUtcOffset(hours: number, original: string): string
```

`getUtcInstant` must return `null` for `localTime: null`. Do not construct a local-time `Date` from a bare `YYYY-MM-DDTHH:mm` string; browser parsing would apply the visitor's zone and corrupt the schedule conversion.

## Styling and responsive layout

- Use the existing CSS approach; avoid introducing Tailwind for this project.
- Keep the map in a fixed, responsive-height container so Leaflet has a non-zero layout size.
- Use a two-column desktop layout for controls/details and map, with a single-column layout below 768px.
- Do not rely on colour alone for selected state or Day/Night distinction.
- Preserve useful focus outlines and ensure text contrast meets WCAG AA for normal text.

## Delivery sequence

1. Replace the starter UI and establish global layout/styles.
2. Add domain types, JSON loader/validation, and pure unit tests.
3. Add calendar and filtered match list with component tests.
4. Add details panel and time/day-night formatting tests.
5. Add Leaflet map, marker grouping, and map-controller behaviour.
6. Complete accessibility/responsive checks and run all quality gates.
