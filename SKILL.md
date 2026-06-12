---
name: stock-screener
description: "Natural-language A-share stock screening skill that translates user filters such as 连续分红, 低估值, 质押率, 北向加仓, 行业/概念/指数成分, 财务增长, 股东变化, and risk exclusions into verified Pandadata API calls and returns an evidence-backed stock list. Use when the user asks for 选股, 筛选股票, 找出满足条件的A股, 财务/分红/股东/资金面过滤, or natural-language stock screening."
license: GPL-3.0-only
metadata:
  organization: QuantSkills
  organization_url: https://github.com/quantskills
  repository: skill-stock-screener
  repository_url: https://github.com/quantskills/skill-stock-screener
  project_type: skill
  collection: stock-screener
  creator: abgyjaguo
  creator_url: https://github.com/abgyjaguo
  maintainer: abgyjaguo
  maintainer_url: https://github.com/abgyjaguo
---

# Stock Screener

Use this skill to turn natural-language A-share screening requirements into a reproducible Pandadata query plan, execute the filters layer by layer, and return a sourced stock list with the actual values behind every matched condition.

## Creator, Maintainer, And Scope

- Creator: `abgyjaguo` (`https://github.com/abgyjaguo`).
- Maintainer: `abgyjaguo` for the QuantSkills community.
- Repository: `https://github.com/quantskills/skill-stock-screener`.
- License: GNU General Public License v3.0 only (`GPL-3.0-only`).
- Scope: research-oriented A-share screening from public Pandadata interfaces. The skill is not official investment advice, a certified data product, or a guarantee of screening performance.

## Core Rules

- Use the local `pandadata-api` skill for every real data call. Before calling an API, inspect its `references/method-index.md` or run `scripts/search_api_docs.py --method <method>` to confirm exact parameters, fields, date conventions, and return shape.
- Never invent Pandadata methods, fields, credentials, factor definitions, or unsupported screening conditions. If a condition cannot be mapped to documented data, say so and offer the nearest auditable proxy.
- Treat screening date as a first-class input. Default to the latest available trading day, but ask for clarification when the user needs a historical point-in-time screen.
- Avoid look-ahead bias. For financial statements, dividends, forecasts, pledge data, and shareholder events, use only rows whose disclosure or announcement date is not later than the screening date.
- Keep every output reproducible: preserve original user criteria, normalized atomic filters, method names, parameters, data cutoff dates, row counts, and missing-data notes.
- End formal screening reports with: `本筛选结果基于公开数据与规则化条件生成，仅供研究参考，不构成任何投资建议。`

## Workflow

1. Parse the request into atomic filters: metric, operator, threshold, time window, universe, date basis, and ranking or limit rule. Restate ambiguous business meaning before execution, such as whether `连续3年分红` means three fiscal years with cash dividends or three calendar years with ex-dividend events.
2. Build the starting universe with `get_trade_list`, then apply explicit universe filters such as exchange, board, industry, concept, index membership, ST exclusion, listing-age, suspension, or user-provided symbols.
3. Read `references/screener-guide.md` when planning any non-trivial screen. Use it for the condition map, execution order, output schema, and QA checklist.
4. Sort filters by selectivity and API cost. Prefer bulk universe/status/industry filters first, then bulk market and financial filters, then per-symbol or sparse event calls such as pledge, unlock, shareholder change, and top-holder checks.
5. Smoke-test each unfamiliar method on one date range or a small symbol set before full-market execution. Record field names and row counts before expanding.
6. Execute filters layer by layer. After each layer, record the remaining stock count, eliminated count, API method, parameters, data date or report period, and any skipped rows.
7. Return a compact Chinese Markdown report by default: criteria interpretation, screening funnel, final table, evidence columns, missing-data caveats, saved result path when a file is created, and the fixed disclaimer.

## Common Method Map

| Need | Primary methods |
|---|---|
| Initial stock pool and trading status | `get_trade_list`, `get_stock_status_change`, `get_stock_detail` |
| Daily market and technical filters | `get_stock_daily`, `get_stock_daily_pre` |
| Valuation and financial statements | `get_fina_reports`, `get_fina_performance`, `get_fina_forecast` |
| Dividends and dividend yield proxies | `get_stock_cash_dividend`, `get_stock_dividend_amount` |
| Holder count, pledge, holder changes | `get_holder_count`, `get_stock_pledge_stat`, `get_stock_shareholder_change`, `get_top_holders` |
| Northbound, margin, abnormal trading | `get_hsgt_hold`, `get_margin`, `get_lhb_list` |
| Unlock and event-risk exclusions | `get_restricted_list` |
| Industry, concept, index membership | `get_industry_constituents`, `get_concept_constituents`, `get_index_weights` |

## Output Standards

- Include only stocks that can be traced to evidence. If a final name remains because data is missing rather than passing a condition, mark it as `数据缺失` and keep it out of strict-pass counts unless the user explicitly allows missing-data inclusion.
- Show actual values, not only pass/fail flags. Use columns such as `代码`, `名称`, `条件命中值`, `报告期/数据日`, `方法`, and `备注`.
- Save machine-readable results only when useful for reruns or when the user asks. Use `screens/<YYYY-MM-DD>-<slug>.json` in the user's working project, not inside the installed skill folder unless the skill folder is also the project workspace.
- For ranked screens, separate hard filters from ranking metrics. State tie-breakers, sort direction, and whether the result is top N, percentile, or threshold based.

## Resource Guide

- `references/screener-guide.md`: detailed condition-to-method map, normalized filter schema, default execution order, JSON result schema, and QA checklist.

## Cross-Agent Use

- Codex and Claude Code can load this folder directly as a skill named `$stock-screener` through `SKILL.md`.
- Cursor should use `agents/cursor-rule.mdc` as the project rule adapter and keep the full skill folder under `.cursor/skills/stock-screener`.
- Hermes and OpenClaw should use `agents/portable-loader.md` when they do not natively discover `SKILL.md` folders.
