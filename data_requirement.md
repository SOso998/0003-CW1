# Data Requirements Log  
Strategy: Quality, Dividend & Sentiment Composite Strategy  
History Requirement: Minimum 5 years backfill  
Query Requirement: Must support retrieval by company and by year  

---

## 1. Dividend Yield

| Field | Specification |
|------|--------------|
| Metric Name | Dividend Yield |
| Raw Fields | Dividend Per Share (DPS), Market Price |
| Source System | PostgreSQL |
| Source Tables | financials (DPS), market_data (Price) |
| Frequency | Monthly (computed using latest available dividend) |
| History | ≥ 5 years |
| Calculation Logic | Dividend Yield = Annualized DPS / Price |

### Missing / Quality Rule
- Price must be available within the previous 3 trading days (Strictly backward-looking).
- If DPS missing:
  - Use latest declared dividend within last 12 months.
  - If no dividend in last 12 months → set Dividend Yield = 0.
- If Price ≤ 0 → drop observation.
- No forward-fill allowed for Price beyond 3 trading days.
- Log quality flag if stale dividend used.

---

## 2. Return on Equity (ROE)

| Field | Specification |
|------|--------------|
| Metric Name | ROE |
| Raw Fields | Net Income, Shareholder Equity |
| Source System | PostgreSQL |
| Source Tables | financials |
| Frequency | Quarterly |
| History | ≥ 5 years |
| Calculation Logic | ROE = Net Income / Shareholder Equity |

### Missing / Quality Rule
- If Shareholder Equity ≤ 0 → drop observation.
- Net Income or Equity may be forward-filled up to 9 months (max staleness).
- If staleness > 9 months → drop observation and log quality flag.
- No interpolation allowed.

---

## 3. Debt/Equity Ratio

| Field | Specification |
|------|--------------|
| Metric Name | Debt/Equity |
| Raw Fields | Total Debt, Short-term Debt, Long-term Debt, Shareholder Equity |
| Source System | PostgreSQL |
| Source Tables | financials |
| Frequency | Quarterly |
| History | ≥ 5 years |
| Calculation Logic | Debt/Equity = Total Debt / Shareholder Equity |

### Missing / Quality Rule
- If Total Debt missing:
  - Compute as Short-term Debt + Long-term Debt.
- If Shareholder Equity ≤ 0 → drop observation.
- Debt components may be forward-filled up to 9 months.
- If debt missing beyond 9 months → drop and log quality flag.

---

## 4. P/E Ratio

| Field | Specification |
|------|--------------|
| Metric Name | P/E |
| Raw Fields | Market Price, EPS (Trailing Twelve Months preferred) |
| Source System | PostgreSQL |
| Source Tables | market_data (Price), financials (EPS) |
| Frequency | Monthly |
| History | ≥ 5 years |
| Calculation Logic | P/E = Price / EPS |

### Missing / Quality Rule
- Price must be available within ±3 trading days.
- EPS may be forward-filled up to 12 months.
- If EPS = 0 or negative → drop observation.
- If EPS missing beyond 12 months → drop.
- Log quality flag if EPS is stale.

---

## 5. News Sentiment Score

| Field | Specification |
|------|--------------|
| Metric Name | News Sentiment Score |
| Raw Fields | Article Sentiment Score |
| Source System | Alpha Vantage API |
| Source | NEWS_SENTIMENT |
| Frequency | Daily (aggregated to monthly for portfolio construction) |
| History | ≥ 5 years (where available) |
| Calculation Logic | Average sentiment score over rolling 30-day window |

### Missing / Quality Rule
- If no news available in 30-day window → assign score = 0 (neutral).
- Do not drop observation due to missing news.
- Cap extreme sentiment scores to [-1, +1].
- Log count of articles used per window for audit.

---

# Acceptance Criteria (System-Level)

The following criteria must be verifiable via automated tests:

### Data Coverage
- Each metric must contain ≥ 5 years of historical data for ≥ 3 companies.
- Missing observations must respect defined staleness limits.

### Query Capability
System must support:

1. Query all metrics for a single company over a 5-year period.
2. Query a single metric across all companies for a given calendar year.

### Uniqueness
- No duplicate keys allowed for (company_id, metric_name, as_of_date).

### Flexibility
- Adding a new company must not require schema change.
- Adding a new metric must not require altering factor_observations schema.

### Quality Flags
- Stale data usage must be logged.
- Dropped observations must be traceable.
- Sentiment window article counts must be stored for audit.



