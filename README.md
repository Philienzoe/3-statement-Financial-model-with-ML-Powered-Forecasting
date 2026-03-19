# Three-Statement Financial Model with ML-Powered Forecasting

A Python application that automatically generates three-statement financial models (Income Statement, Balance Sheet, Cash Flow) for any US-listed company, with forecast assumptions powered by machine learning trained on 7,000+ stocks.

## What This Project Does

Traditional financial models require analysts to manually estimate future assumptions — revenue growth, profit margins, working capital ratios — often based on just a few years of one company's history. This project automates that process in two ways:

1. **Financial Model Generator** — Enter any US stock ticker and get a complete three-statement model with 5-year forecasts, exported as a formatted Excel workbook with linked formulas
2. **ML-Powered Assumptions** — Instead of relying on one company's historical average, the ML pipeline learns patterns from thousands of companies across sectors, market conditions, and economic cycles to make smarter predictions

## How It Works

```
┌─────────────────┐     ┌─────────────────┐     ┌──────────────────┐
                        │  (.xlsx output) │     │                  │
│  Yahoo Finance  │───▶  Data Fetcher     ───▶    315+ financial  
│  (any ticker)   │     │  (dynamic)      │     │  fields captured │
└─────────────────┘     └─────────────────┘     └────────┬─────────┘
                                                         │
                        ┌─────────────────┐              ▼
                        │  ML Predictor   │     ┌──────────────────┐
                        │  (.xlsx output) │     │                  │
                        │  (trained on    │───▶  Assumptions Engine      
                        │   7,000+ stocks)│     │                  │
                        └─────────────────┘     └────────┬─────────┘
                                                         ▼
                        ┌─────────────────┐     ┌──────────────────┐        
                        │  (.xlsx output) │     │                  │
                        │  Excel Report   │◀────  Forecast Engine 
                        │  (.xlsx output) │     │                  │
                        └─────────────────┘     └──────────────────┘
```

### The ML Advantage

| Approach | How Assumptions Are Set | Data Used |
|----------|------------------------|-----------|
| Traditional | Analyst manually reviews 3-4 years of one company | ~4 data points |
| This Project | ML models learn patterns across 7,000+ companies | ~25,000 data points |

The ML pipeline predicts 14 financial assumptions including revenue growth, COGS %, SGA %, tax rate, AR/AP/inventory days, and CapEx. It only uses ML predictions when they demonstrably outperform simple historical averages — otherwise it falls back to the traditional approach.

**Key results** (ML vs historical average baseline):
- Tax Rate: **+24%** more accurate
- Accounts Payable Days: **+38%** more accurate
- Accounts Receivable Days: **+16%** more accurate
- Inventory Days: **+6%** more accurate
- Revenue Growth: **+4%** more accurate

## Project Structure

```
code/
├── main.py                          # Entry point — run the full model
├── data_fetcher.py                  # Pulls 315+ fields from Yahoo Finance
├── assumptions.py                   # Computes historical ratios & forecast drivers
├── forecast_engine.py               # Builds 5-year forecasts for all 3 statements
├── report_generator/                # Excel workbook generation
│   ├── __init__.py                  # Orchestrates all sections
│   ├── base.py                      # Shared styling (Arial Narrow, CFI format)
│   └── sections/
│       ├── assumptions_section.py   # Assumptions tab with historical backsolve
│       ├── income_statement.py      # IS with forecast formulas
│       ├── balance_sheet.py         # BS with summary row formulas
│       ├── cash_flow.py             # CF with cross-statement linking
│       └── supporting_schedules.py  # PP&E schedule, Debt schedule
├── ml/
│   ├── build_dataset.py             # Scrapes 7,000+ US stocks, computes ratios
│   ├── train_models.py              # Trains Ridge, Random Forest, XGBoost
│   ├── predictor.py                 # Predicts assumptions for any ticker
│   ├── macro_fetcher.py             # FRED macroeconomic indicators
│   ├── tune_rf.py                   # Random Forest hyperparameter tuning
│   ├── scrape_russell3000.py        # Russell 3000 ticker list scraper
│   ├── models/                      # Saved .pkl model files
│   └── data/                        # Training CSVs
├── russell3000_tickers.xlsx         # ~7,000 US stock tickers
└── yf_fields_analysis.xlsx          # Field mapping reference
```

## Quick Start

### 1. Install Dependencies

```bash
pip install yfinance openpyxl pandas numpy scikit-learn xgboost joblib fredapi
```

### 2. Generate a Financial Model (No ML)

```bash
cd code
python main.py
```

This generates a complete Excel model for the default ticker (AAPL). To change the ticker, edit `TICKER` in `main.py` or pass it as needed.

### 3. Train ML Models (Optional)

```bash
# Step 1: Build the training dataset (~3-4 hours for 7,000 stocks)
python ml/build_dataset.py

# Step 2: Train models (~5 minutes)
python ml/train_models.py

# Step 3: Test predictions for any ticker
python ml/predictor.py MSFT
```

### 4. Generate a Model with ML Predictions

After training, the ML predictor integrates with `main.py` automatically. It replaces manual assumptions with data-driven predictions where ML outperforms the baseline.

## ML Pipeline Details

### Feature Engineering (124 Features)

The ML models use four categories of features to make predictions:

- **Company Ratios (~28)** — Margins, leverage, efficiency metrics (e.g., gross margin, debt-to-equity, AR days)
- **Lag/Momentum (~56)** — Prior-period values and changes (e.g., was revenue growth accelerating or decelerating?)
- **Sector Peer Benchmarks (~22)** — How the company compares to its sector median (e.g., "this company's margin is 5% above its sector")
- **Macroeconomic Indicators (~18)** — GDP growth, consumer confidence, unemployment rate, M2 money supply from FRED

### Model Training

Three models are trained per target and the best is selected:

- **Ridge Regression** — Linear baseline, works well for stable ratios like tax rate
- **Random Forest** — Captures non-linear patterns, tuned via OOB error following the R `tuneRF()` methodology (p/3 rule for `mtry`)
- **XGBoost** — Gradient boosting for complex feature interactions

Models are trained both globally (all sectors) and per-sector. The sector model is saved only when it beats the global model.

### Data Sources

| Source | What It Provides | Update Frequency |
|--------|-----------------|------------------|
| Yahoo Finance | Income Statement, Balance Sheet, Cash Flow (315+ fields) | Quarterly |
| FRED | 8 macroeconomic indicators + derived features | Monthly/Quarterly |
| SEC EDGAR + iShares | Russell 3000 ticker universe | Annual rebalance |

## Excel Output

The generated Excel workbook includes:

- **Assumptions section** — Historical ratios back-solved from reported data, forecast assumptions (blue font = inputs per CFI convention)
- **Income Statement** — Revenue through Net Income with forecast formulas
- **Balance Sheet** — All assets, liabilities, and equity with summary row formulas and a balance check row
- **Cash Flow Statement** — Operating, investing, and financing activities with cross-statement links
- **Supporting Schedules** — PP&E schedule (CapEx, D&A, closing balance) and Debt schedule (issuance, repayment, interest)

All forecast cells contain Excel formulas, not hardcoded values — so you can change any assumption and the entire model recalculates.

## Technologies

- **Python 3.10+** — Core language
- **pandas / numpy** — Data processing
- **yfinance** — Financial data from Yahoo Finance
- **scikit-learn** — Ridge, Random Forest, preprocessing
- **XGBoost** — Gradient boosting models
- **openpyxl** — Excel workbook generation with formatting
- **fredapi** — FRED macroeconomic data
- **joblib** — Model serialization

## Acknowledgments

- Financial modeling methodology follows [CFI (Corporate Finance Institute)](https://corporatefinanceinstitute.com/) best practices
- Training data sourced from Yahoo Finance and FRED (Federal Reserve Economic Data)
- Russell 3000 universe sourced from iShares IWV ETF holdings and SEC EDGAR

## License

This project is for educational and research purposes.
