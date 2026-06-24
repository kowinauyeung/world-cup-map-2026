# Suggested Commands

## Project (package.json scripts, run with yarn)
- `yarn install` — install dependencies.
- `yarn dev` — start Vite dev server (manual browser checks: branded MapLibre style, WebGL, venue selection, layout, keyboard).
- `yarn build` — `tsc -b && vite build` (type-check + production build; must pass before merge, NFR-03).
- `yarn lint` — `eslint .` (must pass before merge).
- `yarn preview` — preview the production build.
- `yarn test` — NOT yet defined; add a Vitest `test` script during implementation. Completion gate requires `yarn test` to pass.

## Quality gate before considering work done
`yarn test`, `yarn lint`, and `yarn build` must all pass. No `any`, no unchecked raw JSON access, no console errors, and no runtime network calls other than the configured map style/provider resources.

## Git / GitHub (user preference)
- Use `gh` CLI for GitHub ops; fall back to `git` only when gh can't. On `gh` 404/failure, fall back to `gh api`.
- Commit messages: Conventional Commits, imperative mood. Propose message and wait for explicit approval before committing.

## System (Darwin / macOS)
- Standard BSD-flavoured `ls`, `grep`, `find`, `cd`. Shell is zsh.
- Prefer Serena symbolic tools and dedicated file tools over `cat`/`sed`/`grep` shell calls.
