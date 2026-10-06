# Contributing to EX-CT

EX-CT is a small project series around EVE Online tooling. The flagship product is **EXFA · 精密装配助理**
(Exactitude Fitting Assistant) — a Rust fitting engine plus a browser app that replaces Pyfa.

## Repositories

| Repo | What it is | Language |
|---|---|---|
| [EXFA-Engine](https://github.com/EX-CT/EXFA-Engine) | Stateless fitting engine (native CLI + WASM modules) | Rust |
| [EXFA-Data](https://github.com/EX-CT/EXFA-Data) | SDE dataset, fitting presets, search aliases, Jita price snapshots | Python |
| [EXFA-Bench](https://github.com/EX-CT/EXFA-Bench) | Contract test suites + Pyfa oracle | Python, JSON |
| [EXFA-Docs](https://github.com/EX-CT/EXFA-Docs) | Architecture, design and planning docs (docs/00 = the plan) | Markdown |
| [EXFA-App](https://github.com/EX-CT/EXFA-App) | Web app + MCP monorepo ([live site](https://ex-ct.github.io/EXFA-App/)) | TypeScript |
| [history](https://github.com/EX-CT/history) | Archive of the retired eve-* repos | — |

## House rules

- **Commits**: short imperative subject in the repo's own style; the canonical co-author trailer for
  Devin-produced commits is `Co-Authored-By: Devin AI <158243242+devin-ai-integration[bot]@users.noreply.github.com>`
  (GitHub resolves `users.noreply.github.com` by numeric ID only — anything else attributes to strangers).
- **Licence**: code repos are LGPL-3.0-or-later, dual `LICENSE` + `LICENSE.GPL-3.0` files — keep both in sync and
  keep upstream copyright notices. Docs content is CC-BY-4.0. EVE data itself is CCP's and ships only via
  EXFA-Data releases, never in git.
- **Engine contract**: EXFA-Engine owns `contract/` (CONTRACT.md + CONTRACT-GRAPHS.md). Any change that alters
  request/response behaviour bumps the contract revision and updates Bench expecteds in the same sweep. CI pins
  Bench via `bench.lock` (`BENCH_SHA`) and gates output bytes via `ci/round1*.sha256` — intended output changes
  must update those hashes (reproduce locally, don't guess).
- **Warnings policy (contract 1.4.6)**: anything the engine can auto-correct is normalised silently; `warnings[]`
  is reserved for genuine gaps (missing skills, unsupported projected kinds, unknown buff ids).
- **EXFA-App i18n**: every user-facing string goes through `t()`/`tr()` with a zh entry in `i18n-zh.ts` —
  `i18n.test.ts` fails on raw English JSX literals.
- **Formatting**: EXFA-Engine is not rustfmt-clean — never run repo-wide `cargo fmt`; format only new code.
- **No secrets in public**: the ESI client uses PKCE public-client flow (client_id only) — the client secret must
  never appear in any frontend bundle or repo. App secrets belong on a server, if ever.
- **Deploy**: prefer GitHub (Pages, releases, Actions) over custom infra. The web app deploys from `main` via
  `pages.yml`; engine artifacts ship through release tags (`v*`), never built in consumers' CI.

## Testing

- Engine: `cargo test --workspace --locked` (needs `EXFA_DATASET` — see the repo README for where to get the dataset).
- Bench: `python3 run.py` in EXFA-Bench; informational checks live in `tools/check_*.py`.
- Web: `npm run test --workspaces --if-present` in EXFA-App, plus `npx tsc -b` before every push.
