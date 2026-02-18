# Repository Guidelines

## Project Structure & Module Organization
- `code/`: Main BYOND/DM game logic, split by feature area (machines, mobs, reagents, etc.).
- `maps/`: `.dmm` map files and map-specific content.
- `icons/`, `sound/`, `strings/`: Art, audio, and text resources used by the game.
- `tgui/`: React/TypeScript UI workspace (`packages/*`), including tests and lint config.
- `browserassets/`: Browser-facing static assets.
- `config/`: Server config defaults and local config files.
- `tools/`: CI helpers, map/icon tooling, hooks, and scripts.

## Build, Test, and Development Commands
- Compile game locally: open `goonstation.dme` in DreamMaker and build.
- Run game server locally: launch `goonstation.dmb` in DreamDaemon after compiling.
- Install git hooks: `tools\hooks\Install.bat` (map merge helpers and tgui rebuild hooks).
- Build tgui: `tgui\bin\tgui.bat`
- Run tgui dev server: `tgui\bin\tgui.bat --dev`
- Run tgui CI checks locally: `tgui\bin\tgui.bat --ci`
- Run map/icon python checks used in CI:
  - `tools/bootstrap/python -m dmi.test`
  - `tools/bootstrap/python -m mapmerge2.dmm_test`

## Coding Style & Naming Conventions
- Follow `.editorconfig`: tabs by default, spaces for YAML, LF for `*.dm`, `*.dme`, `*.dmm`.
- Keep DM code consistent with `.github/CODE_GUIDELINES.md`:
  - prefer descriptive names (`fed_food`, not `F`);
  - use explicit `src.var`/`src.proc()` in object procs;
  - avoid `usr` outside verb contexts.
- Keep PRs atomic: one behavior change per PR.
- TGUI uses TypeScript/React with ESLint + Prettier; test files use `*.test.ts(x)` or `*.spec.ts(x)`.

## Testing Guidelines
- DM unit tests live in `code/modules/unit_tests/`; add new test files there and include them in `_unit_tests.dm`.
- Local DM unit-test flow: enable `UNIT_TESTS` in `code/_build.dm`, then run the server once to execute tests.
- For UI work, run `tgui\bin\tgui.bat --test` and `tgui\bin\tgui.bat --lint` before opening a PR.

## Commit & Pull Request Guidelines
- Use concise, imperative commit subjects; issue references are encouraged (example: `Fixes #25931`).
- Recent history also uses PR-number suffixes on merge commits (example: `Adds X (#25932)`).
- PRs must follow `.github/PULL_REQUEST_TEMPLATE.md` sections:
  - `About the PR`
  - `Why's this needed?`
  - `Testing` (screenshots/GIFs for visual changes when applicable)
- Changelog is optional for small changes; when included, use a `changelog` code block and `(*)`/`(+)` markers.
