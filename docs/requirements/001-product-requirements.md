# Product requirements: World Cup Map 2026

## Purpose

Provide a fast, accessible way to explore the FIFA World Cup 2026 schedule by date and venue. A visitor can choose a calendar date, see the relevant venues on a map, select a match, and understand its scheduled local, UTC, and browser-local time.

The source of truth is the repository's `data.json`. This product is a schedule explorer, not a live-score service.

## Scope

In scope:

- Calendar-based filtering across all 104 scheduled matches.
- A map of the 16 scheduled venues, filtered by the selected date.
- Match selection from a list or a map marker.
- Match, venue, fixed UTC-offset, UTC-time, browser-local-time, and day/night details.
- Responsive and keyboard-accessible interaction.

Out of scope:

- Fetching live scores, changing the schedule, user accounts, favourites, routing, or ticket purchasing.
- Inferring an IANA time zone or a real astronomical sunrise/sunset from coordinates.
- Translating team and venue names beyond the strings supplied in `data.json`.

## Definitions

- **Schedule date**: the `date` in `data.json`, interpreted at the venue's supplied `gmt_offset`.
- **Selected date**: one schedule date chosen in the calendar, or no date.
- **Selected match**: one match identified by its `match_id`, or no match.
- **Known kick-off**: a match whose `time` is not `null`.
- **Venue marker**: one marker per venue coordinate. A marker can represent more than one match on a selected date.
- **Day**: a known kick-off whose calculated solar altitude at the venue coordinates is at or above the apparent horizon (`>= 0`).
- **Night**: a known kick-off whose calculated solar altitude at the venue coordinates is below the apparent horizon (`< 0`).
- **TBC**: information not supplied by the source data. It must never be fabricated.

## Functional requirements (EARS)

### Initial view and calendar

- **FR-01 — Ubiquitous:** The system shall load all valid matches from `data.json` and render a calendar, map, match-list area, and details area.
- **FR-02 — State-driven:** While no date is selected, the system shall show all 16 venue markers, no selected-match details, and a clear prompt to select a date or venue.
- **FR-03 — Event-driven:** When a user selects a calendar date that contains matches, the system shall filter the match list and venue markers to matches on exactly that schedule date.
- **FR-04 — Event-driven:** When a user selects a calendar date that contains no matches, the system shall show an explicit empty state, show no match markers, and retain no selected match.
- **FR-05 — Ubiquitous:** The system shall visually distinguish calendar dates that contain one or more matches and expose the count in accessible text.
- **FR-06 — Event-driven:** When a user clears the selected date, the system shall restore the all-venue map, clear the selected match, and remove date filtering from the match list.

### Selecting a match

- **FR-07 — Event-driven:** When a user selects a match in the match list, the system shall mark it selected, pan or zoom the map to its venue, and display its details without a page reload.
- **FR-08 — Event-driven:** When a user activates a venue marker that represents one match in the current filter, the system shall select that match.
- **FR-09 — Event-driven:** When a user activates a venue marker that represents multiple matches in the current filter, the system shall present those matches for selection and shall not select one arbitrarily.
- **FR-10 — Ubiquitous:** The system shall use `match_id` as the match-selection identity; it shall not use array position, title, or venue name as an identifier.
- **FR-11 — Ubiquitous:** The selected match details shall include stage, round, match name, venue, schedule date, and result when the result is present.
- **FR-12 — State-driven:** While no match is selected, the details area shall contain an instructional empty state rather than stale match details.

### Time and day/night presentation

- **FR-13 — Event-driven:** When a selected match has a known kick-off, the system shall show the local schedule date and time, the source offset as `UTC±HH:00 (GMT±H)`, and the corresponding UTC date and time.
- **FR-14 — Event-driven:** When a selected match has a known kick-off, the system shall also show the equivalent time in the visitor's browser time zone using `Intl.DateTimeFormat`.
- **FR-15 — Event-driven:** When a selected match has a known kick-off, the system shall calculate the solar altitude at the match UTC instant and venue coordinates. When the altitude is at or above the apparent horizon, the system shall label it **Day**.
- **FR-16 — Event-driven:** When the solar altitude calculated under FR-15 is below the apparent horizon, the system shall label the match **Night**.
- **FR-17 — Unwanted behaviour:** If a match `time` is `null`, the system shall display `Kick-off time: TBC`, retain the supplied venue UTC offset, show `Day/Night: TBC`, and shall not calculate or display a UTC or browser-local timestamp.
- **FR-18 — Ubiquitous:** The map marker popup or accessible label shall expose the venue name and its source UTC offset. It shall expose a Day/Night label only for a specifically selected match with a known kick-off.

### Usability and accessibility

- **FR-19 — Ubiquitous:** All functionality available with a pointer shall be available with a keyboard.
- **FR-20 — Ubiquitous:** Every interactive control shall have an accessible name; map markers shall have a name containing the venue and match count for the active filter.
- **FR-21 — Event-driven:** When a selection changes the details panel, the system shall announce the selected match name through a polite live region or move focus to a clearly labelled details heading without unexpected focus trapping.
- **FR-22 — State-driven:** While the viewport is narrower than 768 CSS pixels, the system shall provide access to calendar, match selection, match details, and map without horizontal page scrolling. Their visual arrangement is a UI-design decision.

## Acceptance scenarios

| ID | Given | When | Then |
| --- | --- | --- | --- |
| AT-01 | no date is selected | the app loads | 16 venue markers are available and the details area is an empty state. |
| AT-02 | the calendar is visible | the user selects `2026-06-11` | exactly matches 1 and 2 are listed, and the two corresponding venue markers are shown. |
| AT-03 | `2026-06-11` is selected | the user selects match 1 | the details show Mexico vs South Africa, Mexico City Stadium, `17:00`, `UTC-06:00 (GMT-6)`, a UTC timestamp, and `Day`. |
| AT-04 | `2026-06-23` is selected | the user selects match 45 | the details show England vs Ghana, `Kick-off time: TBC`, the supplied `UTC-04:00 (GMT-4)`, and `Day/Night: TBC`, with no calculated UTC or browser-local timestamp. |
| AT-05 | `2026-06-24` is selected | the user activates a marker with multiple matches | the UI offers only the matches at that venue on `2026-06-24`; it does not silently choose one. |
| AT-06 | a selected match has `result: null` | its details are displayed | no result value or placeholder score is presented. |
| AT-07 | a date with no matches is selected | filtering completes | the UI states that no matches are scheduled and shows no match markers. |
| AT-08 | a keyboard-only user is on the app | they navigate date, match, and marker controls with keyboard | they can reach the same selected-match details as a pointer user. |

## Non-functional requirements

- **NFR-01:** TypeScript shall remain in strict mode; no production code may introduce `any`.
- **NFR-02:** The application shall have no runtime backend dependency. Requests for the configured vector-map style, map tiles, glyphs, sprites, and required map-provider telemetry are the only permitted runtime network requests.
- **NFR-03:** `yarn lint` and `yarn build` shall pass before merge.
- **NFR-04:** Automated tests shall cover all acceptance scenarios that do not require a real map-tile network request.
- **NFR-05:** The map implementation shall provide all required MapTiler and map-data attribution.
