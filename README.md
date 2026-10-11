# S&P 500 Concentration Analysis

How much of the S&P 500's return since 2010 came from its 10 largest companies, and what would you have earned with or without them?

This project is a single Jupyter notebook that pulls public market data from Yahoo Finance and Wikipedia, rebuilds the S&P 500 from its constituents, and answers four questions:

1. **What did the S&P 500 return since 2010?** Annualized (CAGR) and calendar-year returns for the price index (`^GSPC`) and the total-return index (`^SP500TR`).
2. **What does the index return without its top 10 companies?** Nobody publishes an "S&P 500 ex-top-10" series, so the notebook rebuilds a cap-weighted index from the ~500 constituents and removes the top 10, either *today's* top 10 for the whole period or *each year's* top 10 (annual rebalance).
3. **What if you held only the top 10, re-picked every January?** Cap-weighted, equal-weighted and buy-once-in-2010 versions, compared on CAGR, volatility, drawdown, beta and years beating the index.
4. **Does the RSP/SPY ratio do what people claim?** Four common claims about the equal-weight vs cap-weight ratio (breadth, concentration, correction warning, leadership breakout), each tested against the data.

A PDF export of the full notebook, charts included, is in [`reports/sp500_concentration_report.pdf`](reports/sp500_concentration_report.pdf).

## Key findings

Data from 2010-01-04 to 2026-10-05 (16.75 years), dividends reinvested unless noted.

**The S&P 500 itself**

| Series | Total return | CAGR |
|---|---:|---:|
| S&P 500 price (`^GSPC`) | 586% | 12.18% |
| S&P 500 total return (`^SP500TR`) | 834% | 14.27% |

**Without the top 10.** Today the top 10 companies make up **40.7%** of the index (Nvidia, Apple, Alphabet, Microsoft, Amazon, Meta, Broadcom, Tesla, Micron, Berkshire Hathaway).

| Rebuilt index | CAGR | vs rebuilt full |
|---|---:|---:|
| Rebuilt full index | 15.46% | — |
| Ex-top-10, annual rebalance | 14.61% | −0.85 pts |
| Ex-top-10, today's top 10 removed | 13.00% | −2.46 pts |
| Today's top 10 only | 23.82% | +8.35 pts |

Removing each year's top 10 costs under a point a year. Removing *today's* top 10 costs about 2.5 points, but that version uses hindsight: those companies are the largest because they rose the most.

**Holding only the top 10, re-picked each January**

| Strategy | CAGR | Volatility | Max drawdown | $10,000 becomes |
|---|---:|---:|---:|---:|
| S&P 500 (`^SP500TR`) | 14.27% | 17.2% | −33.8% | $93,394 |
| Top 10, cap-weighted | 17.27% | 20.2% | −39.2% | $144,138 |
| Top 10, equal-weighted | 15.94% | 19.1% | −36.5% | $119,122 |
| 2010 top 10, never re-picked | 16.85% | 17.5% | −28.6% | $135,777 |

On average only 1.4 of the 10 names change each January.

**The RSP/SPY ratio.** It is a good gauge of concentration (−0.86 correlation with the top-10 weight) and is at its lowest level since RSP launched. It has *not* been a reliable timing signal: outside 2007–09, a weak ratio did not predict corrections, and ratio breakouts barely persisted. Owning RSP instead of SPY since 2010 cost about 1.9 points a year.

> Numbers above are from the committed run. Re-running the notebook fetches fresh data, so they will move.

## Getting started

Requires [uv](https://docs.astral.sh/uv/) and Python 3.14 (uv installs it if needed).

```bash
git clone https://github.com/DavidArmendariz/sp500-analysis.git
cd sp500-analysis
uv sync
uv run jupyter lab sp500_annualized_returns.ipynb
```

A full run downloads about 17 years of daily prices and share-count history for ~500 tickers, so it takes a few minutes and needs an internet connection.

To regenerate the PDF report (needs a LaTeX install for nbconvert's PDF exporter):

```bash
uv run jupyter nbconvert --to pdf --execute sp500_annualized_returns.ipynb --output-dir reports
```

## Method

- **Constituents:** today's S&P 500 list from Wikipedia. Share classes are grouped by company (GOOGL and GOOG count as one Alphabet).
- **Prices:** dividend-adjusted daily closes from Yahoo Finance via `yfinance`.
- **Market caps:** split-adjusted price × split-adjusted shares outstanding, using Yahoo's share-count history smoothed with a rolling median.
- **Index:** daily cap-weighted return using the prior day's market caps. Excluded stocks get zero weight on that day.
- **Validation:** the rebuilt full index is checked against the official `^SP500TR` (tracking error ≈ 0.76% a year).

## Caveats

- **Survivorship bias.** Only today's constituents are used, so companies that left the index or went bankrupt since 2010 are missing. This raises the rebuilt returns (rebuilt full: 15.46% vs official 14.27%).
- **Rough early weights.** Yahoo's share-count history starts around 2015, so earlier counts are carried back. The official index also uses float-adjusted caps, which this method does not.
- **Hindsight in the static ex-top-10 version.** The annual-rebalance version avoids it.
- Because these biases affect the rebuilt full index and the ex-top-10 indices alike, **the gap between them** is more reliable than any single level.
- No taxes or trading costs. This is an analysis, not investment advice.

## Project layout

```
.
├── sp500_annualized_returns.ipynb      # the full analysis
├── reports/
│   └── sp500_concentration_report.pdf  # PDF export of the notebook
├── pyproject.toml                      # dependencies (managed with uv)
└── uv.lock
```
