# Localization and venue-profile contract

## Purpose

This contract supplies the factual stadium content and localized application text required by US-09 and US-10. It does not alter `data.json`, which remains the authoritative schedule source.

## Supported application locales

Only these locales are supported:

| App locale | Browser preference matches | Fallback |
| --- | --- | --- |
| `en` | `en`, `en-*` | default when no supported preference matches |
| `ja` | `ja`, `ja-*` | none |
| `ko` | `ko`, `ko-*` | none |
| `es` | `es`, `es-*` | none |
| `zh-Hant` | `zh-Hant`, `zh-TW`, `zh-HK`, `zh-MO` | none |

Resolve locale by iterating `navigator.languages` in order and selecting the first supported match. `navigator.language` is the fallback only when `navigator.languages` is unavailable. Generic `zh`, Simplified Chinese tags such as `zh-CN`/`zh-Hans`, and all unsupported languages resolve to `en`.

The app may add a user-controlled language switcher later, but browser-locale resolution is the required behaviour for this release.

## What must be localized

- Static application UI, accessibility labels, empty states, validation messages, and match-status labels.
- Dates and browser-local times through `Intl` with the resolved locale.
- Venue profile city/country labels, stadium details, background, and history.

Do not automatically translate the source-owned strings in `data.json`: tournament name, match/team names, venue name, stage, round, and result remain the supplied values unless a future source-data contract adds authoritative translations.

## Venue-profile source data

Add a versioned static dataset at `data/venue-profiles.json` (or an equivalent typed module). It is a separate curated-content source, not a replacement for `data.json`.

```ts
type AppLocale = 'en' | 'ja' | 'ko' | 'es' | 'zh-Hant'

type LocalizedText = Record<AppLocale, string>

type VenueProfile = {
  venue: string // exact `venue` value in data.json
  coordinates: {
    latitude: number
    longitude: number
  }
  city: LocalizedText
  country: LocalizedText
  capacity?: number
  openedYear?: number
  background: LocalizedText
  history: LocalizedText
  sources: Array<{
    title: string
    url: string
  }>
}
```

Validation requirements:

1. The dataset has exactly one profile for each of the 16 unique venues in `data.json`.
2. Every profile's venue and coordinates match its schedule venue exactly.
3. Every `LocalizedText` includes all five supported locales and contains non-empty strings.
4. Every profile contains at least one factual source URL. Do not write unsupported or generated historical claims.
5. A missing/duplicate/mismatched profile or missing locale fails validation before the UI renders.

## Downstream consumers

| Consumer | Required input |
| --- | --- |
| `VenueMap` | localized venue name/city plus coordinate-validated profile for venue context |
| venue details experience | background, history, facts, and sources in the resolved locale |
| calendar and full schedule | source `data.json` labels; optionally localized city/country context, never an invented translated match name |
| match details | source match data plus localized shared UI/status labels |
| accessibility labels | localized UI text plus source venue/match names |

No component may read raw profile JSON directly. A typed loader validates it once and returns the resolved profile for the active locale.

## Required tests

- `['ja-JP', 'en-US']` resolves to `ja`; `['ko']` to `ko`; `['es-MX']` to `es`; and `['zh-HK']` to `zh-Hant`.
- `['de-DE', 'en-US']`, `['zh-CN']`, and a missing browser-languages API resolve to `en` when no supported preference is available.
- Every UI message catalogue has the same keys for all five supported locales.
- Venue-profile validation succeeds only with 16 unique, schedule-matched profiles that contain all locales and at least one source.
- Selecting a venue renders the resolved background/history; changing the locale resolver fixture changes the localized profile content without changing source match names.
