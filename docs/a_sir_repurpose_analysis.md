# Repurposing TradingAgents for an A Sir-Style Metric Framework

## Executive Summary

Yes, TradingAgents can be repurposed for an A Sir-style framework with minimal code changes, but only if we are precise about what "minimal" means.

The existing repo already has the correct analytical shape:

```text
Analyst Team
  -> Bull/Bear Research Debate
  -> Research Manager
  -> Trader
  -> Risk Debate
  -> Portfolio Manager
```

That maps naturally to:

```text
Direction -> Value -> Timing -> Risk
```

The strongest near-term approach is not to rebuild the graph. The best approach is to add an "A Sir mode" that changes:

1. Analyst prompts.
2. The required report sections.
3. The final scoring rubric.
4. A small number of deterministic metric snapshot tools.

The current code can already cover a large part of the framework:

- Technical timing from OHLCV, moving averages, RSI, MACD, ATR, Bollinger Bands, VWMA, and MFI.
- Company quality from yfinance or Alpha Vantage fundamentals, balance sheet, income statement, and cash flow.
- Basic valuation from yfinance `Ticker.info`.
- News and macro narrative from yfinance Search or Alpha Vantage news sentiment.
- Risk synthesis through the existing risk debate and portfolio manager.
- Investment journal behavior through the existing persistent memory log.

The current code does not yet provide metric-grade coverage for:

- Policy rates.
- Real yields.
- Yield curve.
- M2 growth.
- Credit spreads.
- central bank balance sheets.
- CPI, PPI, PMI, unemployment, retail sales, industrial production.
- Market breadth.
- historical valuation percentiles.
- peer-relative valuation.
- sector cycle metrics.
- portfolio-level exposure and correlation.

So the honest conclusion is:

```text
Prompt-only repurpose: feasible, very low code change, but weak on metric truth.
Small-tool repurpose: still low code change, much better analytical quality.
Full A Sir quant platform: not minimal; requires new data providers and stored time series.
```

Recommended path:

```text
Phase 1: Add A Sir prompt overlay and scorecard output.
Phase 2: Add 3 deterministic snapshot tools:
  - market regime snapshot
  - sector/benchmark relative strength snapshot
  - company metric snapshot
Phase 3: Add optional macro provider adapters for FRED/official data/paid data.
```

This keeps the current architecture intact and avoids turning the LLM into a fake spreadsheet.

---

## Current Architecture Fit

TradingAgents currently has four selectable analyst buckets:

| Existing component | Current role | A Sir framework mapping | Fit |
| --- | --- | --- | --- |
| Market Analyst | Price data and technical indicators | Timing, market regime, price trend, volatility | Strong |
| News Analyst | Global/company news and macro narrative | Macro direction, policy, sentiment/news | Medium |
| Social Analyst | Company-specific news and public sentiment | Sentiment, narrative, market reaction | Medium |
| Fundamentals Analyst | Company profile, statements, ratios | Quality, growth, cash flow, balance sheet, valuation | Strong |
| Bull Researcher | Builds upside case from analyst reports | Positive scenario, upside catalysts | Strong |
| Bear Researcher | Builds downside case from analyst reports | Red flags, downside, value trap detection | Strong |
| Research Manager | Structured investment plan | Directional synthesis and staged entry logic | Strong |
| Trader | Buy/Hold/Sell, entry, stop, sizing | Timing, execution, trade plan | Strong |
| Risk Analysts | Aggressive/Neutral/Conservative debate | Risk/reward, position sizing, invalidation | Strong |
| Portfolio Manager | Final structured decision | Final score, allocation, journal-quality decision | Strong |
| Memory log | Stores prior decisions and outcome reflections | Investment journal and learning loop | Strong |

The important point: the graph already expresses a multi-layer judgment process. We do not need a new graph to support the A Sir framework.

The biggest weakness is data. The agents can reason about many metrics, but some metrics are not available as tools.

---

## Relevant Code Surface

The main graph is in:

```text
tradingagents/graph/trading_graph.py
tradingagents/graph/setup.py
```

The selected analyst types are currently fixed to:

```text
cli/models.py
  AnalystType.MARKET
  AnalystType.SOCIAL
  AnalystType.NEWS
  AnalystType.FUNDAMENTALS
```

The current tool routing is in:

```text
tradingagents/dataflows/interface.py
```

The current data categories are:

```text
core_stock_apis
technical_indicators
fundamental_data
news_data
```

The tool wrappers are in:

```text
tradingagents/agents/utils/core_stock_tools.py
tradingagents/agents/utils/technical_indicators_tools.py
tradingagents/agents/utils/fundamental_data_tools.py
tradingagents/agents/utils/news_data_tools.py
```

The analyst prompts are in:

```text
tradingagents/agents/analysts/market_analyst.py
tradingagents/agents/analysts/fundamentals_analyst.py
tradingagents/agents/analysts/news_analyst.py
tradingagents/agents/analysts/social_media_analyst.py
```

The structured decision schemas are in:

```text
tradingagents/agents/schemas.py
```

The final reporting path is primarily in:

```text
cli/main.py
```

Because the framework is prompt-driven and report-driven, a lot can be done by adding a small helper that appends A Sir instructions to existing prompts.

---

## Current Tool Coverage

### Existing Price and Technical Coverage

The Market Analyst already has:

```text
get_stock_data(symbol, start_date, end_date)
get_indicators(symbol, indicator, curr_date, look_back_days)
```

Current yfinance/stockstats indicators include:

| Indicator | A Sir use |
| --- | --- |
| `close_10_ema` | short-term timing |
| `close_50_sma` | medium-term trend |
| `close_200_sma` | long-term regime |
| `macd` | momentum turn |
| `macds` | MACD signal |
| `macdh` | momentum acceleration |
| `rsi` | overbought/oversold |
| `boll`, `boll_ub`, `boll_lb` | volatility band / extremes |
| `atr` | stop distance and volatility sizing |
| `vwma` | price-volume trend confirmation |
| `mfi` | volume-adjusted momentum |

This already covers much of:

```text
Market regime
Technical timing
Volatility and stop placement
Risk/reward
```

Missing but easy to add:

| Missing technical metric | Implementation difficulty | Notes |
| --- | ---: | --- |
| 6M / 12M return | Low | Compute from OHLCV. |
| Distance from 52W high/low | Low | Available from yfinance info or OHLCV. |
| ADX | Low/Medium | Stockstats may support it, otherwise compute. |
| Rate of change | Low | Compute from OHLCV. |
| 12M momentum ex-last-month | Low | Compute from OHLCV. |
| Prior support/resistance | Medium | Needs heuristic over swing highs/lows. |
| ZigZag pivots | Medium | Needs deterministic pivot function. |
| Fibonacci retracements | Medium | Derive from swing high/low. |

### Existing Fundamental Coverage

The Fundamentals Analyst already has:

```text
get_fundamentals(ticker, curr_date)
get_balance_sheet(ticker, freq, curr_date)
get_cashflow(ticker, freq, curr_date)
get_income_statement(ticker, freq, curr_date)
```

yfinance fundamentals currently include:

| Existing field | A Sir use |
| --- | --- |
| Sector | sector mapping |
| Industry | peer/sector context |
| Market Cap | size/liquidity |
| PE Ratio (TTM) | valuation |
| Forward PE | valuation |
| PEG Ratio | valuation/growth |
| Price to Book | valuation |
| EPS / Forward EPS | earnings |
| Dividend Yield | income/value |
| Beta | risk |
| 52 Week High/Low | cycle position |
| 50/200 Day Average | trend |
| Revenue | growth base |
| Gross Profit | quality/growth |
| EBITDA | operating profitability |
| Net Income | profitability |
| Profit Margin | profitability |
| Operating Margin | profitability |
| ROE | quality |
| ROA | quality |
| Debt to Equity | balance sheet risk |
| Current Ratio | liquidity |
| Book Value | valuation/accounting |
| Free Cash Flow | cash generation |

This covers a large part of:

```text
Company quality
Growth
Cash flow quality
Balance sheet strength
Basic valuation
Risk profile
```

Missing but easy to compute from statements:

| Missing company metric | Implementation difficulty | Notes |
| --- | ---: | --- |
| Gross margin trend | Low | Gross profit / revenue from income statement. |
| Operating margin trend | Low | Operating income / revenue. |
| Net margin trend | Low | Net income / revenue. |
| Revenue YoY / 3Y CAGR | Low/Medium | Need period alignment. |
| EPS growth | Medium | yfinance statement support varies. |
| FCF margin | Low | FCF / revenue. |
| CFO / net income | Low | Cash flow + income statement. |
| Capex intensity | Low | Capex / revenue. |
| Interest coverage | Medium | EBIT / interest expense if available. |
| Net debt / EBITDA | Medium | Cash + debt + EBITDA. |
| ROIC | Medium | Need invested capital approximation. |
| Altman Z-score | Medium | Needs several fields and sector caveats. |

Missing and not minimal:

| Missing company metric | Why not minimal |
| --- | --- |
| historical valuation percentiles | Requires multi-period market cap/price plus fundamentals history. |
| peer-relative valuation | Requires peer universe definition and peer data fetch. |
| management guidance accuracy | Requires historical guidance data. |
| restatement / auditor change checks | Requires filings dataset. |
| related-party transactions | Requires filings or exchange-specific data. |

### Existing News and Sentiment Coverage

News tools:

```text
get_news(ticker, start_date, end_date)
get_global_news(curr_date, look_back_days, limit)
get_insider_transactions(ticker)
```

Current coverage:

| Current tool | A Sir use | Limit |
| --- | --- | --- |
| yfinance company news | company narrative, earnings headlines, market reaction | No formal sentiment score. |
| yfinance global news search | macro narrative | News articles, not macro data. |
| Alpha Vantage news sentiment | sentiment score if configured | API key and coverage required. |
| insider transactions | governance/alignment | Not all markets reliable. |

This is useful for:

```text
Sentiment/news checklist
Policy/regulatory narrative
Macro qualitative context
Short-term catalysts
```

It is not enough for:

```text
CPI, PMI, M2, rates, credit spreads, yield curve, unemployment
```

For those, we need either:

1. A real macro data provider.
2. Proxy tickers from yfinance.
3. A manual CSV/local data input.

---

## Metric Coverage Matrix

### 1. Macro / Direction

| Metric group | Current support | Minimal repurpose | Honest quality |
| --- | --- | --- | --- |
| Policy rates | News only | Add `get_macro_snapshot()` with provider/proxy fields | Weak unless external data provider added |
| 10Y yields | Possible via yfinance proxy tickers if prompted | Add explicit benchmark/yield tickers | Medium |
| Yield curve | Not directly | Add computed 10Y minus 2Y from provider/proxy | Medium with proper data |
| Real yield | Not directly | Need inflation plus yield | Low without macro provider |
| M2 growth | Not available | Requires FRED/official data | Low unless new provider |
| Credit spreads | Not available | Use ETF/proxy or FRED | Medium with provider |
| Central bank balance sheet | News only | Requires FRED/central bank data | Low unless new provider |
| CPI/PPI/PMI/unemployment/GDP/retail sales | News only | Requires macro provider | Low unless new provider |
| USD/DXY, USD/CNH, oil, gold, copper | yfinance proxy possible | Add macro proxy tickers | Medium/Strong |

Conclusion:

```text
Macro direction can be framed today, but cannot be metric-grade without new data.
```

Minimal change:

```text
Add prompt instructions that force News Analyst to separate:
1. measured macro data available
2. news/proxy inference
3. missing data
```

Better small change:

```text
Add get_macro_proxy_snapshot(region, curr_date) that fetches proxy tickers:
US: ^TNX, ^IRX or 2Y proxy if available, DX-Y.NYB, GLD, USO, CPER/SPY/QQQ/IWM
HK/China: ^HSI, 2800.HK, 2822.HK, CNH/USD proxy, FXI, KWEB, MCHI
```

### 2. Market Regime

| Metric | Current support | Minimal repurpose |
| --- | --- | --- |
| Index price vs 200D MA | Tool supports any symbol if yfinance recognizes it | Prompt Market Analyst to call benchmark index. |
| 50D vs 200D MA | Existing indicators | Prompt or computed snapshot. |
| 6M/12M return | Not explicit | Compute from OHLCV or ask LLM to infer from CSV. |
| Distance from 52W high | Company fundamentals has 52W high/low for ticker | Add index/benchmark fundamentals or compute. |
| Market drawdown | Not explicit | Compute from OHLCV. |
| Breadth | Not available | Requires breadth dataset. |
| Index valuation | Not available | Requires external data or ETF facts. |
| Equity risk premium | Not available | Requires earnings yield and bond yield. |

Conclusion:

```text
Trend regime is easy. Breadth and market valuation are not minimal.
```

Recommended minimal tool:

```text
get_market_regime_snapshot(symbol, curr_date, benchmark=None)
```

Output should include:

```text
close
50D MA
200D MA
6M return
12M return
drawdown from 52W high
volatility
beta if available
regime label: bull / bear / recovery / range
```

### 3. Sector / Theme

| Metric | Current support | Minimal repurpose |
| --- | --- | --- |
| Sector classification | yfinance `sector` and `industry` | Already available. |
| Sector relative strength | Not direct | Need benchmark/sector ETF mapping. |
| Sector above 200D MA | Tool can fetch ETF if known | Add sector ETF map. |
| Sector breadth | Not available | Requires breadth data. |
| Sector valuation vs history | Not available | Requires sector valuation dataset. |
| Sector cycle metrics | Not available | Requires sector-specific data. |

Conclusion:

```text
The repo can discuss sector qualitatively today. It needs a sector ETF map for useful relative strength.
```

Minimal change:

```text
Map yfinance sector -> ETF benchmark:
Technology -> XLK / QQQ
Consumer Cyclical -> XLY
Financial Services -> XLF
Energy -> XLE
Industrials -> XLI
Healthcare -> XLV
Utilities -> XLU
Real Estate -> XLRE
Materials -> XLB
Communication Services -> XLC
Consumer Defensive -> XLP
```

For Hong Kong/China, sector ETF mapping is harder and should be configurable.

### 4. Company Quality

| Metric | Current support | Minimal repurpose |
| --- | --- | --- |
| Gross margin | Statement data available | Compute deterministic ratios. |
| Operating margin | yfinance info and statement data | Already partial. |
| Net margin | yfinance info and statement data | Already partial. |
| ROE | yfinance info | Already available. |
| ROA | yfinance info | Already available. |
| ROIC | Not direct | Compute approximation. |
| Revenue growth | Statement data | Compute deterministic trend. |
| EPS growth | Partial | Use yfinance info/statements when available. |
| FCF growth | Cash flow data | Compute deterministic trend. |
| CFO > net income | Cash flow + income statement | Compute deterministic ratio. |
| Capex intensity | Cash flow + revenue | Compute deterministic ratio. |
| Debt / equity | yfinance info | Already available. |
| Current ratio | yfinance info | Already available. |
| Interest coverage | Statement data if available | Compute if fields exist. |

Conclusion:

```text
This is the easiest high-value improvement. Do not rely on the LLM to compute all ratios from raw CSV. Add a deterministic company metric snapshot.
```

Recommended minimal tool:

```text
get_company_metric_snapshot(ticker, curr_date)
```

It should return a compact markdown table with:

```text
profitability
growth
cash flow quality
balance sheet
valuation
red flags
missing fields
```

### 5. Valuation

| Metric | Current support | Minimal repurpose |
| --- | --- | --- |
| PE | yfinance info | Already available. |
| Forward PE | yfinance info | Already available. |
| PB | yfinance info | Already available. |
| Dividend yield | yfinance info | Already available. |
| FCF yield | free cash flow + market cap | Compute. |
| EV/EBITDA | likely available in yfinance info but not currently surfaced | Add fields. |
| P/FCF | FCF + market cap | Compute. |
| 5Y valuation percentile | Not available | Requires historical fundamentals/market cap. |
| Peer valuation | Not available | Requires peer list and loop. |
| DCF / SOTP / NAV | Not available | Could be LLM-assisted, but assumptions must be explicit. |

Conclusion:

```text
Basic valuation is easy. Historical and peer-relative valuation are not minimal unless we accept approximations.
```

Minimal safe stance:

```text
Report current valuation and clearly label historical/peer valuation as unavailable unless a new tool provides it.
```

### 6. Poor Man's 8P

The existing architecture maps unusually well to the suggested 8P proxy.

| Pillar | Existing source | Minimal code change |
| --- | --- | --- |
| Policy | News Analyst | Add A Sir prompt and optional macro snapshot. |
| Phase | Market Analyst + News Analyst | Add market regime snapshot. |
| Price trend | Market Analyst | Already strong. |
| Profitability | Fundamentals Analyst | Add computed ratios. |
| Profit growth | Fundamentals Analyst | Add computed growth metrics. |
| Prudence | Fundamentals + Risk Analysts | Add balance sheet score. |
| Peer value | Fundamentals | Needs peer tool or mark as missing. |
| Protection | Trader + Risk Analysts + PM | Already strong, improve ATR sizing. |

Recommended output:

```text
| Pillar | Score (-2 to +2) | Evidence | Confidence | Missing data |
```

This table can be prompt-only at first. Later it can be structured.

### 7. Technical Timing

Existing support is good. Additions are incremental.

| Metric | Current support | Recommendation |
| --- | --- | --- |
| Close > 200D MA | Existing indicators | Keep. |
| 50D > 200D | Existing indicators | Keep. |
| Higher highs/lows | Price data available | Add deterministic support/resistance helper later. |
| ADX | Not current | Add if needed. |
| Relative strength vs index | Not direct | Add benchmark snapshot. |
| RSI | Existing | Keep. |
| MACD | Existing | Keep. |
| ROC | Not current | Add simple calculation. |
| Volume surge | OHLCV available | Add deterministic calculation. |
| ATR stop | Existing ATR | Prompt Trader to use it. |

### 8. Wave Theory / Pattern

Current support:

```text
None explicitly, but OHLCV is available.
```

Minimal approach:

```text
Do not add wave counting as a hard signal.
```

If included, implement it as scenario language:

```text
get_price_structure_snapshot(symbol, curr_date)
```

Possible fields:

```text
recent swing highs
recent swing lows
trend structure
major retracement levels
invalidating support
```

The LLM can then say:

```text
The structure resembles an impulsive move, but this is scenario planning, not a confirmed wave count.
```

### 9. Sentiment / News

Current support is adequate for narrative, weak for measurement.

| Metric | Current support | Recommendation |
| --- | --- | --- |
| News volume | yfinance and Alpha Vantage articles | Count articles in tool output. |
| Positive/negative ratio | Alpha Vantage if configured | Add extraction when provider is Alpha Vantage. |
| Policy news | Global news | Improve queries. |
| Earnings headlines | Company news | Already possible. |
| Management guidance | News only | Not reliable without transcripts. |
| Regulatory news | News only | Already possible. |
| Market reaction to news | Price data + news | Add prompt instruction to compare. |

Minimal prompt improvement:

```text
Ask Social/News Analyst to always include:
- narrative direction
- market reaction
- whether price confirms or rejects the news
- missing sentiment data
```

### 10. Risk Management

Current risk architecture is already strong.

| Risk item | Current support | Gap |
| --- | --- | --- |
| Stop / invalidation | Trader schema has stop_loss | Need stronger prompt and ATR support. |
| Position sizing | Trader schema has position_sizing | Need portfolio inputs. |
| Volatility target | ATR and beta available | Need explicit formula. |
| Correlation exposure | Not available | Requires portfolio holdings. |
| Max single position | Not available | Requires portfolio policy input. |
| Portfolio beta | Not available | Requires holdings. |
| Expected value | Prompt-only possible | Needs explicit assumptions. |

Minimal change:

```text
Extend Trader and PM prompts to require:
- entry
- stop/invalidation
- ATR-based stop if appropriate
- upside/downside estimate
- position size under a default risk budget
- what would prove the thesis wrong
```

Better but still small:

```text
Add optional CLI inputs:
- portfolio value
- risk per trade
- max position percentage
```

That would let the Trader compute a real suggested position size.

---

## Recommended "A Sir Mode" Design

### Principle

Do not fork the entire agent graph.

Add a framework instruction layer:

```text
tradingagents/agents/utils/frameworks.py
```

Example API:

```python
def get_framework_instruction(agent: str) -> str:
    ...
```

The default returns an empty string. If config says `analysis_framework = "a_sir"`, it returns a targeted prompt appendix for each agent.

### Config

Add one setting:

```python
DEFAULT_CONFIG = {
    ...
    "analysis_framework": "default",
}
```

Optional CLI prompt:

```text
Select analysis framework:
- Default
- A Sir: Direction -> Value -> Timing -> Risk
```

To keep code changes minimal, this can be an environment variable first:

```text
TRADINGAGENTS_ANALYSIS_FRAMEWORK=a_sir
```

Then CLI support can come later.

### Prompt Overlay

For A Sir mode, every analyst should be told:

```text
Use the Direction -> Value -> Timing -> Risk framework.
Do not treat any one metric as decisive.
Separate hard data from inference.
Mark missing metrics as "Not available from current tools" rather than inventing them.
End with a score table.
```

Agent-specific overlays:

| Agent | Overlay |
| --- | --- |
| Market Analyst | Market regime, trend, momentum, support/resistance, ATR stop, benchmark relative strength if available. |
| News Analyst | Macro direction, policy, inflation/growth cycle, FX/commodity, policy risks, macro missing data. |
| Social Analyst | Sentiment/news flow, market reaction to news, narrative improvement/deterioration. |
| Fundamentals Analyst | Quality, growth, cash flow, balance sheet, valuation, governance red flags, missing metrics. |
| Bull Researcher | Build the best aligned multi-layer setup case. |
| Bear Researcher | Find breaks in macro, sector, quality, valuation, timing, and risk. |
| Research Manager | Produce the 8P proxy score and final watchlist/buy/avoid stance. |
| Trader | Convert score into entry/stop/sizing/staged action. |
| Risk Analysts | Debate downside, invalidation, sizing, concentration, and value trap risk. |
| Portfolio Manager | Produce final 11-category score and journal-ready thesis. |

---

## Proposed Report Contract

The final report should have this shape.

```markdown
# A Sir Framework Report: {ticker}

## 1. Final View

| Decision | Rating | Confidence | Time Horizon |
| --- | --- | --- | --- |
| ... | Buy/Hold/Sell/etc. | High/Medium/Low | ... |

## 2. Direction

### Macro Regime
...

### Market Regime
...

### Sector Phase
...

## 3. Value

### Company Quality
...

### Growth
...

### Cash Flow and Balance Sheet
...

### Valuation
...

## 4. Timing

### Technical Trend
...

### Momentum
...

### Support / Resistance
...

## 5. Risk

### Invalidation
...

### Downside / Upside
...

### Position Sizing
...

## 6. 8P Proxy Score

| Pillar | Score | Evidence | Confidence | Missing Data |
| --- | ---: | --- | --- | --- |

## 7. Full 11-Category Score

| Category | Score | Evidence | What Would Change the Score |
| --- | ---: | --- | --- |

## 8. Journal Entry

Date:
Ticker:
Price:
Macro regime:
Market regime:
Sector view:
Company thesis:
Valuation thesis:
Technical setup:
Catalyst:
Risk:
Stop / invalidation:
Target:
Position size:
Expected holding period:
Confidence:
What would prove me wrong?
```

This can be implemented as markdown first, without changing `AgentState`.

---

## Minimal Code Change Plan

### Phase 1: Prompt-Only A Sir Mode

Estimated change size:

```text
Small: 1 new helper file + edits to 8-10 prompt call sites.
```

Files:

```text
tradingagents/default_config.py
tradingagents/agents/utils/frameworks.py
tradingagents/agents/analysts/market_analyst.py
tradingagents/agents/analysts/news_analyst.py
tradingagents/agents/analysts/social_media_analyst.py
tradingagents/agents/analysts/fundamentals_analyst.py
tradingagents/agents/researchers/bull_researcher.py
tradingagents/agents/researchers/bear_researcher.py
tradingagents/agents/managers/research_manager.py
tradingagents/agents/trader/trader.py
tradingagents/agents/risk_mgmt/*.py
tradingagents/agents/managers/portfolio_manager.py
```

Implementation:

```python
# tradingagents/agents/utils/frameworks.py
from tradingagents.dataflows.config import get_config

def get_framework_instruction(agent_name: str) -> str:
    if get_config().get("analysis_framework") != "a_sir":
        return ""
    return ASIR_AGENT_INSTRUCTIONS[agent_name]
```

Then append it:

```python
system_message = existing_prompt + get_framework_instruction("market_analyst")
```

Benefits:

- Very low risk.
- Preserves graph and report plumbing.
- Immediately changes outputs toward the requested checklist.
- No new dependencies.

Limitations:

- The LLM may infer unavailable macro/breadth/peer metrics.
- The model may compute ratios inconsistently from raw tables.
- Scoring may be inconsistent across runs.

Guardrail:

Add this to every A Sir prompt:

```text
If a metric is not available from the provided tools, say "not available from current tools"; do not invent values.
```

### Phase 2: Add Three Deterministic Snapshot Tools

Estimated change size:

```text
Medium-small: 3 tools + 1 dataflow module + tests.
```

Add:

```text
tradingagents/agents/utils/framework_metric_tools.py
tradingagents/dataflows/framework_metrics.py
tests/test_framework_metrics.py
```

Tool 1:

```python
get_market_regime_snapshot(symbol, curr_date, benchmark=None)
```

Returns:

```text
close
50D MA
200D MA
6M return
12M return
52W drawdown
20D/60D realized vol
regime label
```

Tool 2:

```python
get_sector_relative_snapshot(ticker, curr_date, benchmark=None, sector_proxy=None)
```

Returns:

```text
sector
industry
chosen benchmark
chosen sector proxy
stock 3M/6M/12M return
benchmark 3M/6M/12M return
sector proxy 3M/6M/12M return
relative strength score
```

Tool 3:

```python
get_company_metric_snapshot(ticker, curr_date)
```

Returns:

```text
profitability ratios
growth rates
cash flow quality
balance sheet safety
valuation ratios
red flags
missing data
```

Benefits:

- Still uses existing yfinance/Alpha Vantage dependency surface.
- Avoids hallucinated math.
- Gives LLM compact, purpose-built evidence.
- Makes scorecards much more stable.

Limitations:

- Still does not solve policy rates, CPI, M2, PMI, credit spreads, breadth, or peer valuation.

### Phase 3: Optional Macro Data Provider

Estimated change size:

```text
Medium/large depending on source.
```

Add a new data category:

```python
"macro_data": {
    "description": "Macro economic indicators",
    "tools": [
        "get_macro_snapshot",
        "get_yield_curve",
        "get_inflation_snapshot",
        "get_liquidity_snapshot",
    ],
}
```

Possible providers:

| Provider | Pros | Cons |
| --- | --- | --- |
| FRED | Strong US macro, rates, spreads | US-centric, needs API key for scale |
| OECD / World Bank | broad economic data | slower, less market-focused |
| Trading Economics | broad and convenient | paid for serious use |
| Nasdaq Data Link | many datasets | mixed licensing |
| Local CSV | easiest for custom A Sir dashboard | user must maintain data |

This phase is not required for a first useful version.

---

## Minimal Prompt Changes by Agent

### Market Analyst

Current strength:

- Already selects indicators.
- Already calls OHLCV and technical tools.

Add A Sir requirements:

```text
Classify timing:
- bullish / neutral / bearish trend
- price vs 50D and 200D
- momentum condition
- volatility condition
- support/invalidation level
- ATR-based stop estimate
- whether price confirms or rejects the fundamental thesis
```

Also ask for:

```text
If benchmark/index data is available, compare the stock against the benchmark.
```

### Fundamentals Analyst

Current strength:

- Already pulls profile, ratios, statements.

Add A Sir requirements:

```text
Report quality under:
- profitability
- growth
- cash flow quality
- balance sheet
- valuation
- governance/red flags

Use a table:
| Metric | Current value | Direction | Interpretation | Missing data |
```

Important guardrail:

```text
When raw statement data is insufficient to compute a metric reliably, mark it missing.
```

### News Analyst

Current strength:

- Already uses global news and company news.

Add A Sir requirements:

```text
Separate:
- hard macro data
- policy/regulatory news
- market narrative
- inference
- missing macro metrics
```

The report should not pretend that news articles equal CPI/M2/PMI data.

### Social Analyst

Current strength:

- Company-specific news and sentiment narrative.

Add A Sir requirements:

```text
Focus on:
- narrative improving/deteriorating
- reaction to good news
- reaction to bad news
- attention spike
- sentiment/news mismatch
```

### Bull Researcher

Add:

```text
Build the bullish case only where multiple layers align:
macro tailwind + sector upcycle + company quality + valuation + timing + controlled downside.
```

### Bear Researcher

Add:

```text
Search explicitly for:
- value trap
- liquidity/rate headwind
- sector underperformance
- weak cash conversion
- debt/refinancing risk
- valuation not cheap vs quality
- broken trend
- undefined invalidation
```

### Research Manager

Add:

```text
Produce:
- 8P proxy score
- one-line final stance
- staged action: buy now / wait / watchlist / avoid
- what evidence would change the view
```

### Trader

Add:

```text
Translate the plan into:
- entry
- stop/invalidation
- ATR or support-based stop
- position size using default risk budget
- staged entry plan
```

### Risk Analysts

Add:

```text
Aggressive: what upside justifies risk?
Neutral: what setup is good but not yet confirmed?
Conservative: what breaks the thesis, and how much can be lost?
```

### Portfolio Manager

Add:

```text
Final 11-category score:
- Macro
- Market
- Sector
- Quality
- Growth
- Balance sheet
- Cash flow
- Valuation
- Technical
- Sentiment
- Risk/reward
```

Final decision should include:

```text
rating
confidence
time horizon
position size
stop/invalidation
target or upside case
journal entry
missing critical data
```

---

## Suggested Structured Schema Extension

This is optional. Markdown-first is less invasive.

If we want stronger consistency, add structured outputs:

```python
class FrameworkScore(BaseModel):
    category: str
    score: int  # -2 to +2
    evidence: str
    confidence: str
    missing_data: Optional[str] = None

class ASirDecision(BaseModel):
    rating: PortfolioRating
    total_score: int
    confidence: str
    executive_summary: str
    direction: str
    value: str
    timing: str
    risk: str
    scorecard: list[FrameworkScore]
    entry_plan: Optional[str] = None
    stop_or_invalidation: Optional[str] = None
    position_sizing: Optional[str] = None
    journal_entry: str
```

Where to put it:

```text
tradingagents/agents/schemas.py
```

But this is not necessary for the first pass. The existing `PortfolioDecision` can render the framework in markdown inside `investment_thesis`.

---

## Data Integrity Rules

The biggest risk in this project is not code complexity. It is false precision.

The A Sir checklist is metric-heavy. If the model is asked to produce CPI, credit spread, M2, or peer valuation without data, it may invent or over-infer.

Add these rules to the A Sir prompt overlay:

```text
1. Separate measured data from inference.
2. Mark missing metrics explicitly.
3. Do not invent macro series, peer averages, or historical percentiles.
4. If using a proxy, label it as a proxy.
5. Use score confidence:
   - High: direct metric available
   - Medium: proxy available
   - Low: qualitative/news inference only
```

Recommended score table:

```markdown
| Category | Score | Confidence | Evidence | Missing data |
| --- | ---: | --- | --- | --- |
```

This makes the output auditable.

---

## Proposed Scoring Rubric

### 11-Category Score

Use the user's final checklist:

| Category | Score rule |
| --- | --- |
| Macro | rates/liquidity/inflation/growth support risk appetite |
| Market | index trend, breadth if available, market valuation if available |
| Sector | relative strength, cycle, policy |
| Quality | ROE, margins, ROIC/moat proxy |
| Growth | revenue, EPS, FCF, segment growth |
| Balance sheet | debt, cash, interest coverage |
| Cash flow | CFO, FCF, accruals, working capital |
| Valuation | PE/PB/FCF yield vs history/peers if available |
| Technical | MA, RSI, MACD, support/resistance |
| Sentiment | news, guidance, market reaction |
| Risk/reward | upside/downside, stop, sizing |

Each category:

```text
-2 = materially negative
-1 = mildly negative
 0 = neutral / mixed / unavailable
+1 = mildly positive
+2 = materially positive
```

Final:

```text
+15 or above = high-conviction candidate
+8 to +14 = watchlist / staged entry
+3 to +7 = neutral / wait
<3 = avoid
```

Adjustment:

```text
If one critical category is -2, cap final rating at Hold unless the PM explains why not.
```

Critical categories:

```text
Balance sheet
Cash flow
Risk/reward
Technical trend for trading entries
Macro for high-beta/cyclical names
```

---

## Recommended First PR Scope

If the goal is minimal code changes with meaningful output improvement, this is the best first PR:

### Include

1. Add `analysis_framework` config.
2. Add `get_framework_instruction(agent_name)`.
3. Add A Sir prompt overlays for:
   - Market Analyst
   - Fundamentals Analyst
   - News Analyst
   - Social Analyst
   - Research Manager
   - Trader
   - Risk Analysts
   - Portfolio Manager
4. Add tests that verify:
   - default framework leaves prompts unchanged.
   - A Sir framework injects the scorecard instructions.
   - prompts include missing-data guardrails.
5. Add README documentation for enabling A Sir mode.

### Exclude

1. New macro providers.
2. Peer valuation engine.
3. Market breadth.
4. Portfolio exposure engine.
5. Wave theory hard-signal engine.

Reason:

```text
The first PR should prove the framework shape without dragging in new data contracts.
```

### Good second PR

Add deterministic snapshot tools:

1. `get_market_regime_snapshot`.
2. `get_sector_relative_snapshot`.
3. `get_company_metric_snapshot`.

This is where the output becomes materially better.

---

## Example A Sir Prompt Overlay

```text
You are operating in A Sir framework mode:

Core flow:
Direction -> Value -> Timing -> Risk

Do not treat any one metric as decisive. Look for multi-layer alignment:
macro tailwind + sector upcycle + company quality + attractive valuation
+ improving technicals + controlled downside.

For every major conclusion:
- cite the evidence from available tools
- label whether evidence is direct data, proxy data, or inference
- mark missing metrics explicitly
- avoid inventing policy rates, CPI, PMI, M2, credit spreads, peer averages,
  or historical valuation percentiles if tools did not provide them

End your report with:
1. A 3-5 bullet thesis.
2. Key red flags.
3. A score table using -2 to +2.
4. A final Direction / Value / Timing / Risk summary.
```

---

## Example Final Portfolio Manager Output

```markdown
## Final Decision

**Rating**: Overweight
**Framework Score**: +11 / +22
**Confidence**: Medium
**Action**: Stage entry rather than chase.

## Direction

Macro is mixed-positive. Liquidity is inferred from lower yields and risk-on market behavior, but policy rate/M2/credit spread data were not available from current tools.

## Value

Company quality is strong, with positive margins and cash generation. Valuation is fair to slightly expensive versus current earnings, but peer and 5Y percentile data are unavailable.

## Timing

Trend is positive because price is above the 50D and 200D averages. RSI is elevated, so immediate upside may be less favorable than a pullback entry.

## Risk

Invalidation is a break below 200D MA or a deterioration in cash flow after earnings. Position size should be smaller if ATR is elevated.

## Scorecard

| Category | Score | Confidence | Evidence | Missing data |
| --- | ---: | --- | --- | --- |
| Macro | +1 | Low | News/proxy inference | policy rate, M2, credit spread |
| Market | +1 | Medium | index above 200D proxy | breadth, index valuation |
| Sector | +1 | Low | sector narrative | sector ETF/relative strength |
| Quality | +2 | High | margins, ROE, balance sheet | ROIC |
| Growth | +1 | Medium | revenue/earnings trend | segment CAGR |
| Balance sheet | +1 | Medium | debt/current ratio | maturity profile |
| Cash flow | +1 | Medium | FCF positive | working capital detail |
| Valuation | 0 | Medium | PE/PB available | 5Y percentile, peers |
| Technical | +1 | High | MA/MACD/RSI | support map |
| Sentiment | +1 | Medium | news flow positive | article sentiment score |
| Risk/reward | +1 | Medium | stop definable | precise target |
```

This style is much closer to the user's checklist than the current free-form decision.

---

## Engineering Risks

### 1. Prompt Bloat

A full A Sir checklist is long. Adding all metrics to every prompt will increase context and cost.

Mitigation:

```text
Use agent-specific overlays instead of pasting the full checklist everywhere.
```

### 2. Hallucinated Metrics

The model may invent missing macro data.

Mitigation:

```text
Require "missing from current tools" marking.
Add deterministic snapshot tools for frequently used metrics.
```

### 3. Long Runs and Stream Fragility

The deeper the framework, the longer the prompts and responses. Long Codex OAuth calls already exposed stream fragility, now patched.

Mitigation:

```text
Use checkpointing.
Keep scorecard tables compact.
Use computed metric snapshots instead of raw huge CSV dumps where possible.
```

### 4. Data Vendor Variability

yfinance fields vary by ticker and market. HK/China tickers may have weaker coverage.

Mitigation:

```text
Report missing fields explicitly.
Allow vendor override by tool.
Add local CSV/manual provider later.
```

### 5. False Comparability Across Sectors

ROE, PB, debt, margins, and FCF mean different things for banks, property, tech, commodities, and industrials.

Mitigation:

```text
Add sector-aware interpretation to the prompt.
Eventually add sector-specific score rules.
```

---

## Recommendation

Build this in two steps.

### Step 1: A Sir Prompt Mode

This gives immediate value with minimal code changes.

Target output:

```text
Every run produces Direction -> Value -> Timing -> Risk sections
and a scorecard with explicit missing-data labels.
```

This is enough to test whether the style is useful.

### Step 2: Deterministic Metric Snapshot Tools

This makes the framework reliable.

Target output:

```text
Market regime, sector relative strength, and company quality/valuation
are computed by code before the LLM interprets them.
```

This avoids fake precision and lets the agent focus on judgment.

### Do Not Start With

Do not start by adding a full macro database, market breadth engine, peer screener, DCF engine, and wave theory module. That turns the project into a new platform before proving that the framework improves decisions.

The minimal useful version is:

```text
same graph
+ framework prompt overlay
+ missing-data discipline
+ scorecard output
+ three computed metric tools
```

That is the cleanest path to repurpose TradingAgents into an A Sir-style investment decision agent.
