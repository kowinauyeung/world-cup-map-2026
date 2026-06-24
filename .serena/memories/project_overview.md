# Project Overview: World Cup Map 2026

## Purpose
An interactive, static client-side web app to explore the **FIFA World Cup 2026** schedule by date and venue. A visitor picks a calendar date, sees relevant venues on a map, selects a match, and views its local / UTC / browser-local time plus a Day/Night indicator. It is a **schedule explorer, not a live-score service**.

## Source of truth
- `data.json` (repo root) is the **authoritative** schedule dataset. The app must NOT call any external schedule API or enrich missing fields.
- Current snapshot: 104 matches (`match_id` 1–104), 16 venues, dates `2026-06-11` → `2026-07-19`, offsets GMT-4..GMT-7. 60 matches have `time: null`; 61 have `result: null`. Those nulls are valid data, not parse errors.

## Current state (IMPORTANT)
- The app is **not yet implemented**. `src/` still contains the default Vite + React starter (`App.tsx` is the boilerplate counter/landing page).
- All behaviour is specced in `docs/` (requirements, ADRs, architecture, testing strategy). Implementation must satisfy requirements and follow accepted ADRs.
- Recent commits: "docs: define project requirements and architecture", "initial commit".

## Key docs (read before implementing)
- `docs/requirements/001-product-requirements.md` — EARS functional requirements (FR-01..FR-22), acceptance scenarios (AT-01..AT-08), NFRs.
- `docs/requirements/002-data-contract.md` — raw schema, normalised `Match` model, validity rules, time-conversion formula.
- `docs/decisions/ADR-001..003` — static SPA, MapLibre GL JS + branded MapTiler vector-map style, schedule time semantics (GMT facts + solar Day/Night + TBC).
- `docs/implementation/architecture.md` — module boundaries, state/event flow, delivery sequence.
- `docs/implementation/testing-strategy.md` — test layers, required cases, completion gate.
