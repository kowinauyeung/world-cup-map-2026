# Task Completion Checklist

When a coding task is complete, before considering it done / before merge:

1. **Run quality gates** — all three must pass:
   - `yarn test` (Vitest; add the script if not yet present)
   - `yarn lint` (`eslint .`)
   - `yarn build` (`tsc -b && vite build`)
2. **No forbidden patterns remain**: no TypeScript `any`, no unchecked raw JSON access, no console errors, and no runtime network calls other than configured map-provider resources (NFR-02).
3. **Acceptance coverage**: every acceptance scenario in `docs/requirements/001-product-requirements.md` (AT-01..AT-18) is covered by an automated test or documented manual verification (`docs/implementation/testing-strategy.md`).
4. **Docs in sync**: if a requirement or ADR was deliberately changed during implementation, update the relevant doc in the same change and explain why in the PR (per `docs/README.md`).

## Required automated test cases (testing-strategy.md)
- Loader returns 104 matches; unique ids; valid coordinate ranges; fails with actionable error on invalid coord/offset/duplicate id.
- Match 1 normalises: `2026-06-11`, `17:00`, `utcOffsetHours: -6`, coords `19.3029, -99.1504`; UTC formats to `2026-06-11 23:00 UTC`, with Day/Night based on solar altitude.
- Select `2026-06-11` → only match ids 1 & 2 passed to list/map selectors.
- Unfiltered schedule view makes all 104 matches discoverable without map interaction; venue activation exposes venue name, coordinates, and all active-filter matches without arbitrary selection, with UI placement left to design.
- Each date-filtered out-of-map entry shows match name, local time or TBC, source UTC offset, and venue.
- A non-null source result renders `Finished` and the source score. A known future kick-off with null result renders `Not started`; a known past kick-off with null result renders `Result pending`; null time renders `Kick-off time: TBC`.
- Browser locales resolve only to `en`, `ja`, `ko`, `es`, or `zh-Hant`, with English fallback. Every venue profile is schedule-matched, has all five locales and factual source links, and selected venues render localized detail/background/history.
- Match 45: `localTime: null`, `result: null`; TBC behaviour; no calculated UTC / browser-local time shown.
- `result: null` match → no score output.
- Date with zero matches → required empty state + zero map venue-representation inputs.
- Clearing date → clears selected match + restores all venue groups.
- Multi-match venue representation → invokes chooser/list, not immediate selection.
- All interactive controls have accessible names.
- Freeze/mock `Intl.DateTimeFormat` in tests asserting browser-local time (don't depend on CI locale/TZ).

## Manual browser checklist (Vite dev server)
- At 320px, 768px, and 1440px, no horizontal scroll; keyboard-only navigation of date/match/venue controls; branded MapLibre style, WebGL, attribution, venue representation, and localized venue content load; multi-match venue selection is limited to active date; no-time match never shows fabricated time/Day-Night.
