# Testing strategy and definition of done

## Test layers

| Layer | Tool | Required coverage |
| --- | --- | --- |
| Pure domain | Vitest | parsing, validation, UTC conversion, offset formatting, solar Day/Night classification, date/venue selectors |
| Components | Vitest + React Testing Library + `user-event` | calendar filtering, list/detail selection, TBC states, accessible names, empty states |
| Map boundary | mocked `VenueMap` or mocked MapLibre adapter | venue-group inputs and select callbacks; no real map-provider request in unit tests |
| Manual browser check | Vite dev server | branded vector style, tiles, glyphs, venue representations, attribution, WebGL, viewport layout, keyboard traversal |

## Required automated cases

- Load and validate the checked-in dataset according to [`002-data-contract.md`](../requirements/002-data-contract.md).
- Select `2026-06-11` and assert only match IDs 1 and 2 are passed to list/map selectors.
- Assert that the unfiltered schedule view makes all 104 loaded matches discoverable without map interaction.
- Activate an unfiltered venue representation and assert the venue name, coordinates, and every match at that venue are offered without an arbitrary match selection, regardless of the venue context's visual placement.
- Assert each date-filtered out-of-map match entry displays match name, local time or `TBC`, source UTC offset, and venue.
- Select match 1 and assert its known time, `UTC-06:00 (GMT-6)`, `2026-06-11 23:00 UTC`, and `Day` are rendered.
- With a fixed reference clock of `2026-06-23T02:00:00Z`, assert match 44 is `Not started` and displays every required local/UTC/browser-local start-time value.
- Assert a match with source result `2-0` is labelled `Finished`; assert a known past kick-off with null result is labelled `Result pending`, not `Finished` or `Not started`.
- Select match 45 and assert TBC time/day-night behaviour; assert calculated UTC and browser-local time are absent.
- Assert solar classification with deterministic fixtures: Mexico City (`19.3029`, `-99.1504`) at `2026-06-11T18:00:00Z` is `Day`, and at `2026-06-12T06:00:00Z` is `Night`.
- Assert a `result: null` match has no score output.
- Assert a date with zero matches has the required empty state and zero map venue-representation inputs.
- Assert clearing the date clears the selected match and restores all venue groups.
- Assert a multi-match venue representation invokes a chooser/list rather than selecting a match immediately.
- Assert all interactive calendar, list, and venue-proxy controls have accessible names.

Freeze or mock `Intl.DateTimeFormat` in tests that assert formatted browser-local time, so tests do not depend on the CI machine's locale or time zone.

## Manual acceptance checklist

- Check desktop and 375px-wide layouts: no horizontal page scrolling and all content remains usable.
- Navigate date selection, match selection, and venue-associated controls by keyboard only.
- Inspect the real map at initial load and after date/match selection: the branded style, tiles, glyphs, and venue representations load; attribution is visible; venue coordinates are positioned; and selected-match focus works.
- Verify a venue representation representing multiple matches offers a choice limited to the active date.
- Verify that a no-time match never displays a fabricated UTC/local time or Day/Night label.

## Completion gate

Implementation is complete only when:

1. Every acceptance scenario in [`001-product-requirements.md`](../requirements/001-product-requirements.md) is covered by automated or documented manual verification.
2. `yarn test`, `yarn lint`, and `yarn build` pass.
3. No TypeScript `any`, unchecked raw JSON access, console errors, or required runtime network call beyond configured map-provider resources remains.
4. The README and relevant docs reflect any deliberate requirement or ADR change made during implementation.
