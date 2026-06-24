# ADR-004: Use a calendar component with explicit schedule-date state

- **Status:** Accepted
- **Date:** 2026-06-24

## Context

The primary discovery interaction is selecting a schedule date. A generic native date input does not visibly communicate match density and makes keyboard and empty-state behaviour harder to verify consistently.

## Decision

Use `react-day-picker` in single-selection mode. It shall be constrained to the inclusive date range present in the dataset and use modifiers to mark dates with matches. Each marked day must expose its match count through accessible text.

The calendar owns no hidden match selection. Selecting or clearing a date updates the root `selectedDate` state and clears `selectedMatchId`.

## Consequences

- Users can see which dates are meaningful before selecting one.
- The chosen date and clear action have deterministic testable state transitions.
- A new dependency is added, but it avoids maintaining a bespoke calendar and supports keyboard navigation.

## Alternatives considered

- **Native `input[type=date]`:** rejected because match counts and day modifiers are not portable or visible before interaction.
- **Custom calendar grid:** rejected because calendar focus management and locale edge cases are already solved by a maintained component.
