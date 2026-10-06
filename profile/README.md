# EX-CT — open, AI-friendly EVE Online fitting toolkit

**中文** · EX-CT 正在构建 EXFA（精密装配助理 / Exactitude Fitting Assistant）：一个现代化、对 AI 友好的
EVE Online 配船工具。目标是完整复现 Pyfa 的计算结果与功能，拆分为 *无状态 Rust 计算引擎* + *独立的 SDE 数据管线*
+ *CLI / WASM 接口* + *MCP 服务器* + *网页 UI*。同一个 JSON 请求（`FitRequest`）总是得到逐字节相同的结果
（`FitStats`），既能嵌入网页，也能被 AI 代理直接调用。

**English** · EX-CT builds EXFA (Exactitude Fitting Assistant), a modern, AI-friendly fitting toolkit for
EVE Online. It reproduces Pyfa's numbers and feature set, split into a *stateless Rust engine*, a separate
*SDE data pipeline*, *CLI / WASM* interfaces, an *MCP server* and a *web UI*. The same JSON request
(`FitRequest`) always gives a byte-identical result (`FitStats`), so the engine can run in a browser tab or
be called directly by AI agents.

**▶ Try it / 在线试用: <https://ex-ct.github.io/EXFA-App/>** (runs fully in the browser · 完全在浏览器中计算)

## Repositories / 仓库

| Repo | Role (English) | 作用（中文） | License |
|---|---|---|---|
| [EXFA-Engine](https://github.com/EX-CT/EXFA-Engine) | The fitting engine in Rust: stateless CLI, JSONL RPC (calc, batch, graphs, search), browser WASM build | 配船引擎（Rust）：无状态 CLI、JSONL RPC、浏览器 WASM | LGPL-3.0-or-later |
| [EXFA-Data](https://github.com/EX-CT/EXFA-Data) | CCP SDE → compact versioned engine dataset; daily jita4 market prices, published as [releases](https://github.com/EX-CT/EXFA-Data/releases) | CCP 官方 SDE 转引擎数据包；每日吉他市场价格 | MIT (+ CCP data licence) |
| [EXFA-Bench](https://github.com/EX-CT/EXFA-Bench) | Correctness/performance suites: 326+ cases, Pyfa black-box oracle, scorers, baselines | 统一评测：数百用例、Pyfa 对照、基线门禁 | MIT (oracle: GPL-3.0) |
| [EXFA-Docs](https://github.com/EX-CT/EXFA-Docs) | Design docs: Pyfa analysis, feature-parity inventory, API schema, migration plan, [licensing](https://github.com/EX-CT/EXFA-Docs/blob/main/LICENSING.md) | 设计文档、分析、接口规范、迁移方案 | CC-BY-4.0 (docs), MIT (schemas) |
| [EXFA-App](https://github.com/EX-CT/EXFA-App) | Monorepo: `apps/web` fitting UI (React, engine runs as WASM in-page, zh/en) + `packages/mcp` MCP server for AI agents | 应用仓：网页配船界面 + MCP 服务器 | MIT |
| [history](https://github.com/EX-CT/history) | Consolidated archive of the pre-migration `eve-*` repositories (full git history + release assets) | 迁移前旧仓库的归拢存档（完整 git 历史 + release 资产） | mixed |

## How the engine is judged / 引擎评测方法

- **Contract / 契约:** the engine implements a stateless JSON contract
  ([CONTRACT.md](https://github.com/EX-CT/EXFA-Bench/blob/main/CONTRACT.md),
  [docs/05-api-schema.md](https://github.com/EX-CT/EXFA-Docs/blob/main/docs/05-api-schema.md))
  and loads the same SDE dataset. 引擎实现同一份无状态 JSON 契约，加载同一个 SDE 数据包。
- **Oracle / 对照:** expected values come from Pyfa's eos engine run headless as a **black box**
  ([oracle/](https://github.com/EX-CT/EXFA-Bench/tree/main/oracle)); outputs are compared, no Pyfa code is copied.
  Known Pyfa-vs-SDE disagreements are listed in `expected/known_divergences.json`.
  期望值由无界面运行的 Pyfa 作为“黑盒”产生，只比较数值，不复制代码。
- **Corpus / 用例:** hundreds of fits (community regression fits, hand-written edge cases, and variations:
  fleet boosts, projected effects, wormhole/abyssal/incursion environments, implants, boosters, mutated
  modules, RAH…). Tolerance `|got − want| ≤ max(1e-3, 1e-4·|want|)`.
- **Pyfa parity / 对齐清单:** full feature coverage is tracked in
  [docs/19-pyfa-feature-inventory](https://github.com/EX-CT/EXFA-Docs/blob/main/docs/19-pyfa-feature-inventory.md)
  and audited in [docs/25-pyfa-parity-audit](https://github.com/EX-CT/EXFA-Docs/blob/main/docs/25-pyfa-parity-audit.md).
- **History / 沿革:** the engine won a 10-variant architecture bake-off; the lab and all pre-migration
  repos are preserved in [EX-CT/history](https://github.com/EX-CT/history)
  (evaluation: [docs/09](https://github.com/EX-CT/EXFA-Docs/blob/main/docs/09-engine-round-1-evaluation.md)).

## Licensing / 许可

Policy: **[EXFA-Docs/LICENSING.md](https://github.com/EX-CT/EXFA-Docs/blob/main/LICENSING.md)**.

- Engine (EXFA-Engine): **LGPL-3.0-or-later**, so MIT or proprietary front-ends can link it.
- Pipeline, MCP server, web UI, bench runner, schemas: **MIT**. Docs: **CC-BY-4.0**.
- Pyfa's GPL code is never translated into the LGPL/MIT repos; Pyfa is used only as a black-box test oracle.
  Pyfa-derived preset tables are shipped only as separately labelled LGPL/GPL files.
- EVE Online data © CCP hf., used under CCP's third-party developer licence; generated datasets ship with
  `LICENSE.EVE` as release assets.

中文：引擎采用 LGPL-3.0+，其余仓库 MIT，文档 CC-BY-4.0；不翻译 Pyfa 的 GPL 代码，只把 Pyfa 当黑盒对照；
EVE 数据版权属于 CCP，仅作为 Release 资源分发。

## Other EX-CT projects / 其他项目

[Vexor](https://github.com/EX-CT/Vexor) (static bilingual fit viewer / 配置展示网站) ·
[Shuttle](https://github.com/EX-CT/Shuttle) (system accessibility analysis / 星系通达度分析)

<sub>EVE Online and all related trademarks are the property of CCP hf. EX-CT is not affiliated with CCP.</sub>
