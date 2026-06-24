# Product requirements: World Cup Map 2026

## Purpose

Provide a fast, accessible way to explore the FIFA World Cup 2026 schedule by date and venue. A visitor can choose a calendar date, see the relevant venues on a map, select a match, and understand its scheduled local, UTC, and browser-local time.

The source of truth is the repository's `data.json`. This product is a schedule explorer, not a live-score service.

## Scope

In scope:

- Calendar-based filtering across all 104 scheduled matches.
- An unfiltered map of all 16 scheduled venues and a date-filtered map view.
- A complete, map-independent schedule of all 104 matches.
- Match selection from a list or a map venue representation.
- Match, venue, fixed UTC-offset, UTC-time, browser-local-time, and day/night details.
- Responsive and keyboard-accessible interaction.

Out of scope:

- Fetching live scores, changing the schedule, user accounts, favourites, routing, or ticket purchasing.
- Fetching real-time weather, stadium conditions, or live solar data from an external service.
- Translating team and venue names beyond the strings supplied in `data.json`.

## Definitions

- **Schedule date**: the `date` in `data.json`, interpreted at the venue's supplied `gmt_offset`.
- **Selected date**: one schedule date chosen in the calendar, or no date.
- **Selected match**: one match identified by its `match_id`, or no match.
- **Known kick-off**: a match whose `time` is not `null`.
- **Venue representation**: one selectable representation per venue coordinate. It can represent more than one match on a selected date.
- **Day**: a known kick-off whose calculated solar altitude at the venue coordinates is at or above the apparent horizon (`>= 0`).
- **Night**: a known kick-off whose calculated solar altitude at the venue coordinates is below the apparent horizon (`< 0`).
- **TBC**: information not supplied by the source data. It must never be fabricated.

## Functional requirements (EARS)

### Initial view and calendar

- **FR-01 — Ubiquitous:** The system shall load all valid matches from `data.json` and render a calendar/schedule experience, map, match-list area, and details area.
- **FR-02 — State-driven:** While no date is selected, the system shall show all 16 venue representations at their supplied geographic coordinates, make each venue's name and geographic context available, show the complete schedule, and show no selected-match details.
- **FR-03 — Event-driven:** When a user selects a calendar date that contains matches, the system shall filter venue representations and the out-of-map match list to matches on exactly that schedule date.
- **FR-04 — Event-driven:** When a user selects a calendar date that contains no matches, the system shall show an explicit empty state, show no venue representations, and retain no selected match.
- **FR-05 — Ubiquitous:** The system shall visually distinguish calendar dates that contain one or more matches and expose the count in accessible text.
- **FR-06 — Event-driven:** When a user clears the selected date, the system shall restore the all-venue map, complete schedule, and unfiltered venue-match lists, and clear the selected match.

### Selecting a match

- **FR-07 — Event-driven:** When a user selects a match in the match list, the system shall mark it selected, pan or zoom the map to its venue, and display its details without a page reload.
- **FR-08 — Event-driven:** When a user activates a venue representation that represents one match in the current filter, the system shall select that match.
- **FR-09 — Event-driven:** When a user activates a venue representation that represents multiple matches in the current filter, the system shall present those matches for selection and shall not select one arbitrarily.
- **FR-10 — Ubiquitous:** The system shall use `match_id` as the match-selection identity; it shall not use array position, title, or venue name as an identifier.
- **FR-11 — Ubiquitous:** The selected match details shall include stage, round, match name, venue, schedule date, and result when the result is present.
- **FR-12 — State-driven:** While no match is selected, the details area shall contain an instructional empty state rather than stale match details.

### Complete schedule and venue exploration

- **FR-13 — Ubiquitous:** The system shall make every loaded match discoverable in a complete schedule view independent of map interaction. Each entry shall include match name, schedule date, local kick-off time or `TBC`, and venue.
- **FR-14 — Event-driven:** When a user activates a venue representation, the system shall display contextual venue name, coordinates, and all matches at that venue within the active date filter. Each listed match shall be selectable; the venue context's placement is a UI-design decision.
- **FR-15 — State-driven:** While a date is selected, each out-of-map filtered match entry shall display match name, local kick-off time or `TBC`, source UTC offset, and venue.

### Time and day/night presentation

- **FR-16 — Event-driven:** When a selected match has a known kick-off, the system shall show the local schedule date and time, the source offset as `UTC±HH:00 (GMT±H)`, and the corresponding UTC date and time.
- **FR-17 — Event-driven:** When a selected match has a known kick-off, the system shall also show the equivalent time in the visitor's browser time zone using `Intl.DateTimeFormat`.
- **FR-18 — Event-driven:** When a selected match has a known kick-off, the system shall calculate the solar altitude at the match UTC instant and venue coordinates. When the altitude is at or above the apparent horizon, the system shall label it **Day**.
- **FR-19 — Event-driven:** When the solar altitude calculated under FR-18 is below the apparent horizon, the system shall label the match **Night**.
- **FR-20 — Unwanted behaviour:** If a match `time` is `null`, the system shall display `Kick-off time: TBC`, retain the supplied venue UTC offset, show `Day/Night: TBC`, and shall not calculate or display a UTC or browser-local timestamp.
- **FR-21 — Ubiquitous:** The map venue context or accessible label shall expose the venue name and its source UTC offset. It shall expose a Day/Night label only for a specifically selected match with a known kick-off.

### Usability and accessibility

- **FR-22 — Ubiquitous:** All functionality available with a pointer shall be available with a keyboard.
- **FR-23 — Ubiquitous:** Every interactive control shall have an accessible name; map venue controls shall have a name containing the venue and match count for the active filter.
- **FR-24 — Event-driven:** When a selection changes the details panel, the system shall announce the selected match name through a polite live region or move focus to a clearly labelled details heading without unexpected focus trapping.
- **FR-25 — State-driven:** While the viewport is narrower than 768 CSS pixels, the system shall provide access to calendar, match selection, match details, and map without horizontal page scrolling. Their visual arrangement is a UI-design decision.

### Match results and start status

- **FR-26 — Event-driven:** When a selected match has a non-null source `result`, the system shall label the match **Finished** and display that result. The result shall take precedence over time-based status calculation.
- **FR-27 — State-driven:** While a match has a null `result`, a known UTC kick-off instant, and that instant is later than the reference clock, the system shall label it **Not started** and display its scheduled local start date/time, source UTC offset, UTC start time, and browser-local start time.
- **FR-28 — Unwanted behaviour:** If a match has a null `result` and either a null kick-off time or a known kick-off instant no later than the reference clock, the system shall not label it **Finished** or **Not started**. It shall display `Kick-off time: TBC` for the former, and **Result pending** for the latter.

## Acceptance scenarios

| ID | Given | When | Then |
| --- | --- | --- | --- |
| AT-01 | no date is selected | the app loads | 16 venue representations are available and the details area is an empty state. |
| AT-02 | the calendar is visible | the user selects `2026-06-11` | exactly matches 1 and 2 are listed, and the two corresponding venue representations are shown. |
| AT-03 | `2026-06-11` is selected | the user selects match 1 | the details show Mexico vs South Africa, Mexico City Stadium, `17:00`, `UTC-06:00 (GMT-6)`, a UTC timestamp, and `Day`. |
| AT-04 | `2026-06-23` is selected | the user selects match 45 | the details show England vs Ghana, `Kick-off time: TBC`, the supplied `UTC-04:00 (GMT-4)`, and `Day/Night: TBC`, with no calculated UTC or browser-local timestamp. |
| AT-05 | `2026-06-24` is selected | the user activates a venue representation with multiple matches | the UI offers only the matches at that venue on `2026-06-24`; it does not silently choose one. |
| AT-06 | a selected match has `result: null` | its details are displayed | no result value or placeholder score is presented. |
| AT-07 | a date with no matches is selected | filtering completes | the UI states that no matches are scheduled and shows no venue representations. |
| AT-08 | a keyboard-only user is on the app | they navigate date, match, and venue controls with keyboard | they can reach the same selected-match details as a pointer user. |
| AT-09 | no date is selected | the app loads | all 16 venues and all 104 schedule entries are discoverable, with no map interaction required to browse the schedule. |
| AT-10 | no date is selected | the user activates a venue | the UI shows that venue's name, coordinates, and every match scheduled there, without automatically choosing one. |
| AT-11 | a date with matches is selected | the filtered list is shown outside the map | every entry includes its match name, local time or `TBC`, source UTC offset, and venue. |
| AT-12 | match 1 is selected | its result is present in the source data | the UI labels the match `Finished` and displays `2-0`. |
| AT-13 | the reference clock is `2026-06-23T02:00:00Z` | match 44 is displayed | the UI labels it `Not started` and displays its `2026-06-22 20:00 GMT-7` start time and converted UTC/browser-local start times. |
| AT-14 | the reference clock is after `2026-06-23T03:00:00Z` | match 44 has no source result | the UI labels it `Result pending`, not `Finished` or `Not started`. |

## Non-functional requirements

- **NFR-01:** TypeScript shall remain in strict mode; no production code may introduce `any`.
- **NFR-02:** The application shall have no runtime backend dependency. Requests for the configured vector-map style, map tiles, glyphs, sprites, and required map-provider telemetry are the only permitted runtime network requests.
- **NFR-03:** `yarn lint` and `yarn build` shall pass before merge.
- **NFR-04:** Automated tests shall cover all acceptance scenarios that do not require a real map-tile network request.
- **NFR-05:** The map implementation shall provide all required MapTiler and map-data attribution.
