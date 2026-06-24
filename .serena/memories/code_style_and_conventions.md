# Code Style & Conventions

## TypeScript
- Strict mode; **no `any` in production code** (NFR-01). Prefer `const`.
- No semicolons in existing code (Vite starter style: single quotes, no semicolons, 2-space indent). Match surrounding style.
- Use `import type` where appropriate (`verbatimModuleSyntax: true`).

## Architecture / module boundaries (docs/implementation/architecture.md)
Target layout (filenames may vary, but preserve boundaries):
```
src/
  domain/      # pure TS: types, normalizeTournament, venueProfiles, selectors. MUST NOT import React or MapLibre.
  data/        # imports + validates + normalises data.json and venue profiles ONCE.
  i18n/        # browser-locale resolver and static UI message catalogues.
  features/
    calendar/  MatchCalendar.tsx
    matches/   MatchList.tsx, MatchDetails.tsx
    venues/    VenueDetails.tsx
    map/       VenueMap.tsx, mapConfig.ts         # only MapLibre-aware boundary
  App.tsx      # owns ONLY selectedDate + selectedMatchId; derives everything else
```
- Feature components receive typed props, emit intent via callbacks; never touch raw JSON.
- Venue details receive a validated, locale-resolved profile; source match/team/venue strings are never machine-translated.
- `App` is sole owner of selection state; do NOT persist derived filters in state (ADR-001).

## State (root)
```ts
selectedDate: string | null
selectedMatchId: number | null
```
Required transitions: calendar select → set date, clear matchId; calendar clear → both null; match select → keep date, set id; single-match venue representation → select it; multi-match venue representation → show chooser, don't auto-select; if filter drops selected id → clear it.

## Time semantics (ADR-003 / data-contract) — CRITICAL
- Supplied `gmt_offset` is a fixed schedule fact; never infer IANA zone from coordinates.
- `UTC instant = Date.UTC(y, m-1, d, h, min) - offsetHours*3_600_000`. E.g. `2026-06-11 17:00 GMT-6` = `2026-06-11 23:00 UTC`.
- If `time` is null: do NOT create a Date, show `Kick-off time: TBC`, `Day/Night: TBC`, no UTC/browser-local timestamp. Never fabricate TBC values.
- Use `suncalc@^2.0.0` with the UTC instant and venue coordinates: solar altitude `>= 0` is Day; below zero is Night. Use `Intl.DateTimeFormat` only after a UTC instant exists.
- Pure helpers contract: `getUtcInstant(match): Date | null`, `getDayNight(match): 'day'|'night'|'tbc'`, `formatUtcOffset(hours, original): string`.
- Match display status is derived in this order: non-null result → `finished`; null result + null time → `time_tbc`; null result + future UTC instant → `not_started`; otherwise → `result_pending`. Pass a reference clock into the helper; domain logic must not read `new Date()` directly.
- Resolve `navigator.languages` once at bootstrap to only `en`, `ja`, `ko`, `es`, or `zh-Hant`; no supported match (including `zh-CN`) falls back to English. Locale is immutable configuration, not root interaction state.

## Styling / a11y
- Calendar control, general UI styling system, layout, and venue-marker visual form are UI-design decisions. Narrow viewports must have no horizontal scroll and retain every required interaction.
- Don't rely on colour alone for selected/Day-Night state. Preserve focus outlines, WCAG AA contrast.
- All pointer functionality must be keyboard-accessible; every control needs an accessible name. Import MapLibre CSS once; verify the branded style, provider attribution, and an equivalent keyboard-accessible venue-selection path in production build.
- Verify calendar/schedule, map, venue details, match details, and localized content at 320px, 768px, and 1440px without horizontal overflow or hover-only required actions.
- Render `result` only when `result !== null`. Use `match_id` as selection identity (never array index/title/venue).
