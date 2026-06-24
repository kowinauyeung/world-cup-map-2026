# Testing strategy and definition of done

## Test layers

| Layer | Tool | Required coverage |
| --- | --- | --- |
| Pure domain | Vitest | parsing, validation, UTC conversion, offset formatting, Day/Night classification, date/venue selectors |
| Components | Vitest + React Testing Library + `user-event` | calendar filtering, list/detail selection, TBC states, accessible names, empty states |
| Map boundary | mocked `VenueMap` or mocked Leaflet adapter | marker-group inputs and select callbacks; no real tile request in unit tests |
| Manual browser check | Vite dev server | actual tiles, Leaflet icon assets, viewport layout, keyboard traversal |

## Required automated cases

- Load and validate the checked-in dataset according to [`002-data-contract.md`](../requirements/002-data-contract.md).
- Select `2026-06-11` and assert only match IDs 1 and 2 are passed to list/map selectors.
- Select match 1 and assert its known time, `UTC-06:00 (GMT-6)`, `2026-06-11 23:00 UTC`, and `Day` are rendered.
- Select match 45 and assert TBC time/day-night behaviour; assert calculated UTC and browser-local time are absent.
- Assert a `result: null` match has no score output.
- Assert a date with zero matches has the required empty state and zero map marker inputs.
- Assert clearing the date clears the selected match and restores all venue groups.
- Assert a multi-match marker invokes a chooser/list rather than selecting a match immediately.
- Assert all interactive calendar, list, and marker-proxy controls have accessible names.

Freeze or mock `Intl.DateTimeFormat` in tests that assert formatted browser-local time, so tests do not depend on the CI machine's locale or time zone.

## Manual acceptance checklist

- Check desktop and 375px-wide layouts: no horizontal page scrolling and all content remains usable.
- Navigate date selection, match selection, and marker-associated controls by keyboard only.
- Inspect the real map at initial load and after date/match selection: tiles load, attribution is visible, markers are positioned, and selected-match focus works.
- Verify a marker representing multiple matches offers a choice limited to the active date.
- Verify that a no-time match never displays a fabricated UTC/local time or Day/Night label.

## Completion gate

Implementation is complete only when:

1. Every acceptance scenario in [`001-product-requirements.md`](../requirements/001-product-requirements.md) is covered by automated or documented manual verification.
2. `yarn test`, `yarn lint`, and `yarn build` pass.
3. No TypeScript `any`, unchecked raw JSON access, console errors, or required runtime network call beyond map tiles remains.
4. The README and relevant docs reflect any deliberate requirement or ADR change made during implementation.
