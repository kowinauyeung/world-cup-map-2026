# Tech Stack

## Core
- **Vite 8** + **React 19** (with React Compiler via `babel-plugin-react-compiler` / `@rolldown/plugin-babel`)
- **TypeScript ~6.0** (strict mode required; no `any` in production code per NFR-01)
- **Package manager: Yarn Classic** (yarn.lock present). Node.js 22+.
- ESLint 10 (flat config) with `typescript-eslint`, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`.

## To be added during implementation (per architecture.md)
```bash
yarn add maplibre-gl suncalc@^2.0.0
yarn add --dev vitest jsdom @testing-library/react @testing-library/user-event
```
Add a minimal Vitest config + a `test` script.

## Technical constraints and UI freedom
- Do not add a state-management library, date-time library, backend client, or MapLibre renderer wrapper without a technical decision.
- Map: **MapLibre GL JS** with a branded **MapTiler** vector-map style (ADR-002). Configure a browser token through `VITE_MAP_STYLE_URL`; a `VITE_*` token is public and must be restricted by allowed origin. Do not commit a real style URL/token.
- Calendar control, general UI styling system, layout, and venue-marker visual form are deliberately left to UI design. Preserve the documented functional and accessibility requirements.

## TS config notes (tsconfig.app.json)
- target/lib ES2023 + DOM, `moduleResolution: bundler`, `verbatimModuleSyntax: true`, `noEmit`, `jsx: react-jsx`.
- `noUnusedLocals`, `noUnusedParameters`, `erasableSyntaxOnly`, `noFallthroughCasesInSwitch` enabled.
- Will likely need `resolveJsonModule` for importing `data.json` (per ADR-001).
