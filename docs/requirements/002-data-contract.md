# Data contract: `data.json`

## Authority and current snapshot

`data.json` is the authoritative schedule dataset for this application. The implementation must not call an external schedule API or enrich missing schedule fields.

The checked-in snapshot contains:

- 104 matches with unique numeric `match_id` values from 1 through 104.
- 16 distinct venues and valid latitude/longitude pairs.
- Schedule dates from `2026-06-11` through `2026-07-19`.
- Fixed supplied offsets: `GMT-4`, `GMT-5`, `GMT-6`, and `GMT-7`.
- 60 matches with `time: null` (matches 45–104).
- 61 matches with `result: null` (matches 44–104).

Those null values are valid source data, not parsing failures.

## Raw schema

```ts
type RawTournament = {
  tournament: string
  stages: RawStage[]
}

type RawStage = {
  stage_name: string
  rounds: RawRound[]
}

type RawRound = {
  round_name: string
  matches: RawMatch[]
}

type RawMatch = {
  match_id: number
  date: string // YYYY-MM-DD
  time: string | null // HH:mm, or TBC
  gmt_offset: string // GMT-4 through GMT-7 in the current dataset
  match_name: string
  result: string | null
  venue: string
  coordinates: string // "latitude, longitude"
}
```

## Normalised application model

Create one parsing/normalisation boundary. UI components must receive this model rather than re-parse strings independently.

```ts
type Match = {
  id: number
  stageName: string
  roundName: string
  date: string
  localTime: string | null
  utcOffsetHours: number
  utcOffsetLabel: string // e.g. "UTC-04:00 (GMT-4)"
  title: string
  result: string | null
  venue: string
  coordinates: {
    latitude: number
    longitude: number
  }
}
```

The normalisation layer shall:

1. Flatten stage and round names into each match.
2. Parse `coordinates` into numeric latitude and longitude.
3. Parse `GMT±H` into an integer hour offset, retaining the original value for display.
4. Preserve `null` for unknown kick-off time and result.
5. Reject malformed records with a descriptive error before rendering, instead of silently omitting them.

## Validity rules

| Field | Required | Rule |
| --- | --- | --- |
| `match_id` | yes | positive, unique integer |
| `date` | yes | ISO `YYYY-MM-DD` calendar date |
| `time` | no | `HH:mm` 24-hour time or `null` |
| `gmt_offset` | yes | `GMT` plus signed whole-hour offset |
| `match_name` | yes | non-empty string |
| `result` | no | non-empty string or `null` |
| `venue` | yes | non-empty string |
| `coordinates` | yes | two finite numbers: latitude −90…90, longitude −180…180 |

## Time conversion contract

The supplied offset is a fixed schedule fact. Do not infer or replace it with a guessed IANA zone based on coordinates.

For a match with date `YYYY-MM-DD`, known local time `HH:mm`, and numeric offset `offsetHours`:

```text
UTC instant = Date.UTC(year, month - 1, day, hour, minute) - offsetHours × 3,600,000
```

For example, `2026-06-11 17:00 GMT-6` is `2026-06-11 23:00 UTC`. A negative offset therefore adds hours to the local wall-clock timestamp.

If `time` is `null`, do not create a JavaScript `Date` for the match and do not convert it.

## Match display-status contract

`data.json` provides no explicit match-status or end-time field. Derive display status with a supplied reference clock, in this exact precedence order:

```ts
type MatchDisplayStatus =
  | 'finished'
  | 'not_started'
  | 'result_pending'
  | 'time_tbc'
```

1. Return `finished` when `result !== null`, regardless of the scheduled timestamp.
2. Return `time_tbc` when `result === null` and `localTime === null`.
3. Return `not_started` when `result === null`, the UTC kick-off instant exists, and it is later than the reference clock.
4. Return `result_pending` when `result === null` and the known UTC kick-off instant is at or before the reference clock.

Do not infer a live, postponed, cancelled, or finished state from date/time alone. The reference clock must be injected into status helpers in tests rather than read implicitly from the system clock.

## Required data tests

- The loader returns 104 matches for the current file.
- Every loaded match has a unique `id` and a valid coordinate range.
- Match 1 normalises to `2026-06-11`, `17:00`, `utcOffsetHours: -6`, and coordinates `19.3029`, `-99.1504`.
- Match 45 retains `localTime: null` and `result: null`.
- The computed UTC instant for match 1 formats to `2026-06-11 23:00 UTC`.
- Match 1 has `finished` status for any reference clock because its source result is non-null. With a reference clock of `2026-06-23T02:00:00Z`, match 44 has `not_started` status; with a clock later than `2026-06-23T03:00:00Z`, match 44 has `result_pending` status.
- A match with null result and a known past kick-off returns `result_pending`; a match with null time returns `time_tbc`.
- An invalid coordinate, offset, or duplicate ID makes the loader fail with an actionable error.
