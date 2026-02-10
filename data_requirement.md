# Data Requirements Log

| Metric | Raw Fields | Source System | Source | Frequency | History | Calculation Logic | Missing / Quality Rule |
|--------|------------|---------------|--------|-----------|---------|------------------|-----------------------|
| ROE | Net Income, Shareholder Equity | PostgreSQL | financials | Semi-annual | 5y | ROE = Net Income / Shareholder Equity | If Shareholder Equity ≤ 0, drop observation. Forward-fill Net Income or Equity with max staleness 9 months. If staleness > 9 months, drop and log quality flag. |
| P/E | Price, EPS | PostgreSQL | market_data | Monthly Hybrid | 5y | P/E = Price / EPS | Price must be available within 3 trading days; EPS forward-fill ≤ 12 months; EPS=0 or missing >12 months → drop. |
| Debt/Equity | Total Debt, Shareholder Equity | PostgreSQL | financials | Semi-annual | 5y | Debt/Equity = Total Debt / Equity | If Total Debt missing, sum ST+LT; Equity ≤0 → drop. Forward-fill debt components ≤9 months. |
| Revenue Growth YoY | Revenue | PostgreSQL | trading updates | Quarterly | 5y | (Rev_t - Rev_t-4)/Rev_t-4 | If quarterly update missing, forward-fill revenue max 6 months; if base period t-4 missing → drop. |
| Operating Margin | Operating Income, Revenue | PostgreSQL | financials | Semi-annual | 5y | Op Margin = Operating Income / Revenue | Revenue ≤ 0 → drop. Forward-fill Operating Income & Revenue independently ≤9 months. No interpolation. |
| News Sentiment | Sentiment score | Alpha Vantage | NEWS_SENTIMENT | Monthly | 5y | Avg sentiment in 30-day window | If no news in window → return 0. Do not drop observation. |
