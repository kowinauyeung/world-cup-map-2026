# User stories: venue and schedule exploration

These stories define user outcomes. They intentionally do not prescribe a calendar library, layout, visual style, or presentation pattern; those remain UI-design decisions.

## US-01 — See the tournament's venue geography

**As a visitor, I want to open the main page and see every tournament stadium on a geographic map, so that I can understand where the World Cup is being played.**

Acceptance notes:

- With no date filter, the map represents all 16 venues from the current dataset at their supplied coordinates.
- A venue representation exposes its name and geographic context; it is not an unlabeled dot.
- The venue map has a keyboard-accessible equivalent selection path.

## US-02 — Explore a stadium's matches from the map

**As a visitor, I want to select a stadium on the map and see the matches scheduled there, so that I can explore one venue's programme and match details.**

Acceptance notes:

- The result identifies the selected venue and exposes its coordinates.
- With no selected date, it lists every match at that venue; with a selected date, it lists only that date's matches at that venue.
- Every listed match is selectable and opens its complete details without a page reload. The venue-match list's placement is a UI-design decision.
- Selecting a venue with several matches must not silently select an arbitrary match.

## US-03 — Browse every match in the calendar schedule

**As a visitor, I want the calendar experience to expose the complete tournament schedule, so that I can discover all matches rather than only dates that I already know.**

Acceptance notes:

- Every one of the 104 loaded matches is discoverable through the complete schedule/calendar experience.
- Each schedule entry identifies the match, schedule date, local kick-off time or `TBC`, and venue.
- The visual calendar design and how it connects to the complete list are UI-design decisions.

## US-04 — Filter venues and match details by date

**As a visitor, I want to choose a date from a month calendar and see only that date's stadiums on the map, so that I can focus on one matchday.**

Acceptance notes:

- Selecting a schedule date filters map venues and the out-of-map match list to exactly that date's matches.
- Every filtered match entry outside the map displays match name, local time or `TBC`, source UTC offset, and venue.
- Selecting a date with no matches gives an explicit empty state and renders no match venues.
- Clearing the date restores the complete venue map and schedule.

## US-05 — See every venue on the map

**As a visitor, I want to see the full venue/stadium set on the unfiltered map, so that no host location is hidden before I choose a date.**

Acceptance notes:

- The unfiltered map represents all unique venues in `data.json`, not one venue representation per match.
- A venue with several scheduled matches remains one venue representation with a match count or equivalent clear affordance.
- No date filter is required to start exploring venues.

## US-06 — Browse the complete match schedule

**As a visitor, I want to see the full match schedule, so that I can browse the whole tournament independently of the map.**

Acceptance notes:

- The complete schedule includes all stages and rounds supplied by `data.json`.
- It does not omit matches with unknown kick-off times or results; those fields display `TBC` or remain absent according to the data contract.
- The schedule remains usable without interacting with the map.

## Traceability

| User story | Primary requirements |
| --- | --- |
| US-01, US-05 | FR-01, FR-02, FR-14, FR-21–FR-23 |
| US-02 | FR-07–FR-12, FR-14 |
| US-03, US-06 | FR-01, FR-05, FR-06, FR-13 |
| US-04 | FR-03–FR-06, FR-15 |
