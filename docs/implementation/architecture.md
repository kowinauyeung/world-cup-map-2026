# Implementation architecture

This is the implementation contract for Claude Code. Follow the product requirements and accepted ADRs; do not substitute libraries or change time semantics without updating the relevant ADR.

## Dependencies to add

```bash
yarn add maplibre-gl suncalc@^2.0.0
yarn add --dev vitest jsdom @testing-library/react @testing-library/user-event
```

Add the smallest compatible Vitest configuration for this Vite project and a `test` script. Do not add a state-management library, date-time library, backend client, or map-renderer wrapper without a technical decision. The calendar and general UI styling systems are deliberately not prescribed: Claude may choose them during UI design while preserving the requirements.

Create a tracked `.env.example` containing an empty `VITE_MAP_STYLE_URL=` value. Store the real MapTiler style URL only in `.env.local` and deployment environment configuration. It is a public browser credential, not a secret: restrict its MapTiler token to approved origins and never grant administration privileges.

## Proposed module boundaries

```text
src/
  domain/
    tournament.ts          # Raw and normalised types
    normalizeTournament.ts # parsing, validation, UTC and solar Day/Night helpers
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
      mapConfig.ts          # reads and validates VITE_MAP_STYLE_URL
  App.tsx                  # selectedDate and selectedMatchId only
  App.css
```

Exact filenames may differ, but preserve these boundaries:

- **Domain** is pure TypeScript and must not import React or MapLibre.
- **Data** imports, validates, and normalises the static JSON once.
- **Feature components** receive typed props and emit intent through callbacks; they do not access raw JSON.
- **`App`** is the sole owner of selection state and derives all filtered values.
- **`VenueMap`** receives already-grouped venue representations and is the only MapLibre-aware feature boundary. It owns the map instance, teardown, and selected-venue `flyTo` effect.

## State and event flow

```text
calendar select/clear ─┐
match list select ─────┼─> App: selectedDate + selectedMatchId
map venue activate ────┘              │
                                      ├─> derived matches by date
                                      ├─> derived venue groups
                                      └─> selected MatchDetails + map focus
```

Required transitions:

1. On calendar select: set `selectedDate`; set `selectedMatchId` to `null`.
2. On calendar clear: set both values to `null`.
3. On match selection: keep the current date and set that match ID.
4. On a single-match venue representation: select that match.
5. On a multi-match venue representation: expose an in-context list; set an ID only after the user chooses one.
6. If filtering means the selected ID is no longer present, clear it.

## Rendering rules

- Calendar days outside the dataset's range must not be selectable.
- With no selected date, map all venue groups from all matches and make all 104 matches discoverable in a complete, map-independent schedule view. UI design may choose its visual arrangement, but it must not omit records.
- With a selected date, render only that day's matches and venue groups, and show each out-of-map match entry's name, local time or `TBC`, source UTC offset, and venue.
- One venue representation may represent multiple matches. Its context/list must show only matches in the current filtered collection.
- Render result only if `result !== null`.
- Use an accessible HTML match list and details panel; do not make the map the sole interaction path.
- Import MapLibre CSS once. Use a UI-design-defined venue representation rather than provider default pins, and verify the branded style, icons, glyphs, attribution, and an equivalent keyboard-accessible selection path in the production build.

## Time helper contract

Expose pure helpers with behaviour equivalent to:

```ts
getUtcInstant(match: Match): Date | null
getDayNight(match: Match): 'day' | 'night' | 'tbc'
getMatchDisplayStatus(match: Match, referenceTime: Date): MatchDisplayStatus
formatUtcOffset(hours: number, original: string): string
```

`getUtcInstant` must return `null` for `localTime: null`. Do not construct a local-time `Date` from a bare `YYYY-MM-DDTHH:mm` string; browser parsing would apply the visitor's zone and corrupt the schedule conversion.

`getDayNight` shall return `tbc` when `getUtcInstant` returns `null`. Otherwise it shall call `SunCalc.getPosition(utcInstant, latitude, longitude)` and return `day` when `altitude >= 0`, or `night` when it is below zero. The calculation is based on the apparent horizon; do not substitute a local clock-time range or make a network request.

`getMatchDisplayStatus` shall implement the precedence rules in the data contract. Pass the current clock into the helper; do not let domain logic access `new Date()` implicitly. UI code may provide `new Date()` at render time, while tests pass a fixed reference time.

## Styling and responsive layout

- The UI styling approach and layout are intentionally left to the design implementation.
- Keep the map in a fixed, responsive-height container so MapLibre has a non-zero layout size.
- Ensure narrow viewports have no horizontal page scrolling and retain access to every required interaction.
- Do not rely on colour alone for selected state or Day/Night distinction.
- Preserve useful focus outlines and ensure text contrast meets WCAG AA for normal text.

## Delivery sequence

1. Replace the starter UI and establish global layout/styles.
2. Add domain types, JSON loader/validation, and pure unit tests.
3. Add calendar and filtered match list with component tests.
4. Add details panel and time/day-night formatting tests.
5. Add MapLibre map, branded style configuration, venue grouping, and selected-venue `flyTo` behaviour.
6. Complete accessibility/responsive checks and run all quality gates.
