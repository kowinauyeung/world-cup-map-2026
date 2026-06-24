# ADR-003: Treat supplied GMT offsets as schedule facts and expose TBC explicitly

- **Status:** Accepted
- **Date:** 2026-06-24

## Context

The schedule supplies a `date`, optional `time`, and `gmt_offset` such as `GMT-4`. It does not contain IANA time-zone identifiers or sunset data. Sixty matches have no kick-off time, so treating every match as a JavaScript timestamp would invent a time.

## Decision

Interpret each known kick-off as local wall-clock time at the supplied fixed GMT offset. Compute UTC using the formula in the data contract, and use `Intl.DateTimeFormat` only after that UTC instant exists to display the visitor's browser-local equivalent.

Show both labels in the details UI:

- source schedule offset: `UTC-04:00 (GMT-4)`;
- converted UTC date and time, only when kick-off is known.

Use a deterministic presentation classification: `Day` for local `06:00`–`17:59`, `Night` for `18:00`–`05:59`. A null time produces `Kick-off time: TBC` and `Day/Night: TBC`; it has no UTC or browser-local conversion.

## Consequences

- The app faithfully displays the dataset even if political time-zone rules change or coordinates would suggest another zone.
- Day/night is a clear UI classification, not a claim about actual daylight at a venue.
- No extra time-zone or astronomy dependency is required.
- TBC states are first-class test cases rather than error paths.

## Alternatives considered

- **Derive an IANA zone from coordinates:** rejected because geocoding is imperfect and would override the source schedule.
- **Use the browser's zone as the event zone:** rejected because it gives different schedule semantics to each visitor.
- **Estimate sun position:** rejected because it exceeds the product need and requires date/time data that is absent for many matches.
