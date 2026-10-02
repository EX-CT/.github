# EX-CT — open, AI-friendly EVE Online fitting toolkit

**中文** · EX-CT 正在构建一个现代化、对 AI 友好的 EVE Online 配船工具：目标是完整复现 Pyfa 的计算结果，
但拆分为 *无状态计算引擎* + *独立的 SDE 数据管线* + *CLI / HTTP / WASM 接口* + *MCP 服务器* + *网页 UI*。
同一个 JSON 请求（`FitRequest`）总是得到逐字节相同的结果（`FitStats`），既能嵌入网页，也能被 AI 代理直接调用。

**English** · EX-CT builds a modern, AI-friendly fitting toolkit for EVE Online. It aims to reproduce Pyfa's numbers,
split into a *stateless engine*, a separate *SDE data pipeline*, *CLI / HTTP / WASM* interfaces, an *MCP server* and
a *web UI*. The same JSON request (`FitRequest`) always gives a byte-identical result (`FitStats`), so the engine can run
in a browser tab or be called directly by AI agents.

**▶ Try it / 在线试用: <https://ex-ct.github.io/eve-fit-web/>** (runs fully in the browser · 完全在浏览器中计算)

## Repositories / 仓库

| Repo | Role (English) | 作用（中文） | License |
|---|---|---|---|
| [eve-dogma-rs](https://github.com/EX-CT/eve-dogma-rs) | Reference dogma engine in Rust: stateless CLI, JSONL RPC (calc, EFT parse/export, search) | 参考引擎（Rust）：无状态 CLI、JSONL RPC | LGPL-3.0-or-later |
| [eve-dogma-lab](https://github.com/EX-CT/eve-dogma-lab) | Engine architecture bake-off: one branch per competing variant (B–K) and round-2 graph prototypes (`graphs-g1`…) | 引擎架构竞赛：每个方案一个分支 | LGPL-3.0-or-later (variant E: GPL-3.0-or-later) |
| [eve-dogma-bench](https://github.com/EX-CT/eve-dogma-bench) | Shared correctness/performance bench: 326 cases, Pyfa oracle, scorer, round evaluation | 统一评测：326 个用例、Pyfa 对照、计分 | MIT (oracle: GPL-3.0) |
| [eve-sde-pipeline](https://github.com/EX-CT/eve-sde-pipeline) | CCP SDE (JSONL) → compact versioned engine dataset + fitting presets; GitHub Actions every 6 h, published as [releases](https://github.com/EX-CT/eve-sde-pipeline/releases) | 把 CCP 官方 SDE 转为引擎数据包与预设，每 6 小时自动发布 | MIT (+ CCP data licence) |
| [eve-fit-web](https://github.com/EX-CT/eve-fit-web) | Web fitting UI (React); engines in the browser (TypeScript worker, Rust→WASM) or over HTTP; zh/en | 网页配船界面，中英双语，引擎在浏览器内运行 | MIT |
| [eve-fit-mcp](https://github.com/EX-CT/eve-fit-mcp) | MCP server for AI agents: search, validate, compute, compare, optimise fits on any contract engine | 给 AI 代理用的 MCP 服务器 | MIT |
| [eve-fit-docs](https://github.com/EX-CT/eve-fit-docs) | Design docs: Pyfa analysis, feature parity checklist, API schema, MCP design, evaluations, [licensing](https://github.com/EX-CT/eve-fit-docs/blob/main/LICENSING.md) | 设计文档、分析、接口规范、评测报告 | CC-BY-4.0 (docs), MIT (schemas) |

Engine variants in **eve-dogma-lab** / 竞赛方案 (details: [docs/09](https://github.com/EX-CT/eve-fit-docs/blob/main/docs/09-engine-round-1-evaluation.md)):

| Branch | Approach / 思路 |
|---|---|
| [variant-b](https://github.com/EX-CT/eve-dogma-lab/tree/variant-b) | Rust, data-oriented: flat CSR modifier graph / 数据导向 |
| [variant-c](https://github.com/EX-CT/eve-dogma-lab/tree/variant-c) | Go, pull-based modifier registry / Go 拉取式 |
| [variant-d](https://github.com/EX-CT/eve-dogma-lab/tree/variant-d) | TypeScript, zero-dependency, browser + Node / TypeScript（网页默认引擎之一） |
| [variant-e](https://github.com/EX-CT/eve-dogma-lab/tree/variant-e) | Rust, Pyfa eos handlers transpiled (GPL-3.0) / 忠实移植 Pyfa |
| [variant-f](https://github.com/EX-CT/eve-dogma-lab/tree/variant-f) | Rust codegen: SDE compiled into code, builds to WASM / 代码生成 + WASM |
| [variant-g](https://github.com/EX-CT/eve-dogma-lab/tree/variant-g) | Python + NumPy, vectorised batch / 批量向量化 |
| [variant-h](https://github.com/EX-CT/eve-dogma-lab/tree/variant-h) | Rust ECS (hecs) / 实体组件系统 |
| [variant-i](https://github.com/EX-CT/eve-dogma-lab/tree/variant-i) | Rust salsa, incremental queries / 增量计算 |
| [variant-j](https://github.com/EX-CT/eve-dogma-lab/tree/variant-j) | C++20, mmapped POD dataset image / C++ 高性能 |
| [variant-k](https://github.com/EX-CT/eve-dogma-lab/tree/variant-k) | C# / .NET 8 Native AOT, typed rule book / C# 规则书 |

Variant A is [eve-dogma-rs](https://github.com/EX-CT/eve-dogma-rs) itself. / 方案 A 即 eve-dogma-rs。

## How engines are judged / 引擎评测方法

- **Contract / 契约:** every engine implements the same stateless JSON contract
  ([CONTRACT.md](https://github.com/EX-CT/eve-dogma-bench/blob/main/CONTRACT.md),
  [schemas](https://github.com/EX-CT/eve-fit-docs/tree/main/schema)) and loads the same SDE dataset.
  所有引擎实现同一份无状态 JSON 契约，加载同一个 SDE 数据包。
- **Oracle / 对照:** expected values come from Pyfa's eos engine run headless as a **black box**
  ([oracle/](https://github.com/EX-CT/eve-dogma-bench/tree/main/oracle)); outputs are compared, no Pyfa code is copied.
  Known Pyfa-vs-SDE disagreements are listed in `expected/known_divergences.json`.
  期望值由无界面运行的 Pyfa 作为“黑盒”产生，只比较数值，不复制代码。
- **Corpus / 用例:** 326 fits (community regression fits, hand-written edge cases, and variations: fleet boosts,
  projected effects, wormhole/abyssal/incursion environments, implants, boosters, mutated modules, RAH…), 21 051 values.
  Tolerance `|got − want| ≤ max(1e-3, 1e-4·|want|)`.
- **Gate + score / 门槛与计分:** an engine is ranked only if it passes **all** cases at its branch head as of the cutoff
  (no fallback). Then **Total = 0.40·Speed + 0.35·Maintainability + 0.15·Features + 0.10·Portability**
  (speed = latency, batch throughput, cold start on a log scale; maintainability = tests, data-driven design, size,
  docs, dependencies, build time). Authoritative rules: [`tools/evaluate.py`](https://github.com/EX-CT/eve-dogma-bench/blob/main/tools/evaluate.py);
  write-up: [docs/09](https://github.com/EX-CT/eve-fit-docs/blob/main/docs/09-engine-round-1-evaluation.md).
  必须通过全部用例才能参与排名；之后按 速度 40%、可维护性 35%、功能 15%、可移植性 10% 计分。

## Licensing / 许可

Policy: **[eve-fit-docs/LICENSING.md](https://github.com/EX-CT/eve-fit-docs/blob/main/LICENSING.md)**.

- Engine (eve-dogma-rs and the lab variants): **LGPL-3.0-or-later**, so MIT or proprietary front-ends can link it.
  The Pyfa-transpiled variant E is GPL-3.0.
- Pipeline, MCP server, web UI, bench runner, schemas: **MIT**. Docs: **CC-BY-4.0**.
- Pyfa's GPL code is never translated into the LGPL/MIT repos; Pyfa is used only as a black-box test oracle.
  Pyfa-derived preset tables are shipped only as separately labelled LGPL/GPL files.
- EVE Online data © CCP hf., used under CCP's third-party developer licence; generated datasets ship with `LICENSE.EVE`
  as release assets.

中文：引擎采用 LGPL-3.0+，其余仓库 MIT，文档 CC-BY-4.0；不翻译 Pyfa 的 GPL 代码，只把 Pyfa 当黑盒对照；
EVE 数据版权属于 CCP，仅作为 Release 资源分发。

## Other EX-CT projects / 其他项目

[Vexor](https://github.com/EX-CT/Vexor) (static bilingual fit viewer / 配置展示网站) ·
[Shuttle](https://github.com/EX-CT/Shuttle) (system accessibility analysis / 星系通达度分析) ·
[eve-incursions](https://github.com/EX-CT/eve-incursions) (incursion tracker, retired / 已停止维护)

<sub>EVE Online and all related trademarks are the property of CCP hf. EX-CT is not affiliated with CCP.</sub>
