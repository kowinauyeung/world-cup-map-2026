# ADR-003: Treat supplied GMT offsets as schedule facts and calculate solar Day/Night

- **Status:** Accepted
- **Date:** 2026-06-24

## Context

The schedule supplies a `date`, optional `time`, `gmt_offset` such as `GMT-4`, and venue coordinates. It does not contain IANA time-zone identifiers or sunrise/sunset data. Sixty matches have no kick-off time, so treating every match as a JavaScript timestamp would invent a time.

## Decision

Interpret each known kick-off as local wall-clock time at the supplied fixed GMT offset. Compute UTC using the formula in the data contract, and use `Intl.DateTimeFormat` only after that UTC instant exists to display the visitor's browser-local equivalent.

Show both labels in the details UI:

- source schedule offset: `UTC-04:00 (GMT-4)`;
- converted UTC date and time, only when kick-off is known.

Use `suncalc@^2.0.0` to calculate the solar altitude at the known UTC instant and venue coordinates. Classify a match as `Day` when `SunCalc.getPosition(utcInstant, latitude, longitude).altitude >= 0`; classify it as `Night` when the altitude is below zero. This uses the apparent horizon at the venue, rather than a local clock-time heuristic.

A null time produces `Kick-off time: TBC` and `Day/Night: TBC`; it has no UTC or browser-local conversion and no solar calculation.

## Consequences

- The app faithfully displays the dataset even if political time-zone rules change or coordinates would suggest another zone.
- Day/night reflects the calculated solar position at the venue and kick-off instant. It does not account for stadium roofs, local terrain, weather, or artificial lighting.
- `suncalc` is a small deterministic client-side dependency; no sunrise/sunset API call is required.
- TBC states are first-class test cases rather than error paths.

## Alternatives considered

- **Derive an IANA zone from coordinates:** rejected because geocoding is imperfect and would override the source schedule.
- **Use the browser's zone as the event zone:** rejected because it gives different schedule semantics to each visitor.
- **Fixed local-time window (06:00–17:59):** rejected because it does not represent actual daylight across venues and dates.
- **External sunrise/sunset API:** rejected because coordinates and known UTC instants are already available, and a client-side calculation avoids a runtime dependency and inconsistent API availability.
