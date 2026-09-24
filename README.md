# Stock Market SAS (Single-Agent System)

A single-agent stock analysis assistant. You ask a question in plain English, and an LLM decides which tools to call to fetch real market data, compute indicators, and produce a simple BUY/HOLD/SELL suggestion.

Built with Groq's `openai/gpt-oss-20b` (tool calling), the Alpha Vantage API, SQLite, and pandas.

> **Disclaimer:** This is a learning project. Outputs are not financial advice and are based on limited historical data and simple heuristics.

## How it works

The agent runs a loop: the LLM reads your question, picks one or more tools, the code executes them, the results go back to the LLM, and it writes the final answer. If a tool returns an error, the agent tells you instead of making up numbers.

### Tools

| Tool | What it does |
|------|--------------|
| `get_stock_data` | Historical daily OHLCV data (last ~100 days available, cached) |
| `get_live_price` | Latest quote and % change |
| `get_indicators` | RSI(14) and SMA(20) computed from cached history |
| `predict_next` | Next-day close estimate using linear regression on the last 20 closes |
| `suggest_action` | Rule-based BUY/HOLD/SELL score combining RSI, SMA, and the prediction |

### Caching

Alpha Vantage's free tier has strict rate limits, so responses are cached in SQLite:

- **Live quotes:** 60-second TTL (prices change often)
- **Daily history:** 6-hour TTL (daily data rarely changes)

### Scoring logic

`suggest_action` adds or subtracts one point for each signal:

- RSI < 30 (oversold): +1, RSI > 70 (overbought): -1
- Close above SMA(20): +1, below: -1
- Regression predicts up: +1, down: -1

A score of 2 or more gives **BUY**, -2 or less gives **SELL**, anything else gives **HOLD**.

## Setup

```bash
git clone https://github.com/tanishq2905/Stock-Market-SAS.git
cd Stock-Market-SAS
pip install -r requirements.txt
```

Get free API keys from [Alpha Vantage](https://www.alphavantage.co/support/#api-key) and [Groq](https://console.groq.com/keys), then set them as environment variables:

```bash
export ALPHA_VANTAGE_API_KEY="your_key"
export GROQ_API_KEY="your_key"
```

On Windows (PowerShell):

```powershell
$env:ALPHA_VANTAGE_API_KEY="your_key"
$env:GROQ_API_KEY="your_key"
```

If you're using Google Colab, store the keys in Colab Secrets instead of pasting them into the notebook.

## Usage

```bash
python stock_agent.py
```

Then ask questions like:

- `What is the current price of AAPL?`
- `What's the RSI and 20-day average for MSFT?`
- `Should I buy TSLA?`

### Example

```
You: Should I buy AAPL?

Agent: <paste a real output from your own run here>
```

## Limitations

- The linear-regression "prediction" is a simple heuristic, not a real forecasting model. Stock prices are noisy and this should not be relied on.
- The BUY/HOLD/SELL rules are basic and don't account for fundamentals, news, or risk.
- The Alpha Vantage free tier limits how many requests you can make per minute and per day.
- Only daily data is used; there is no intraday analysis.

## Ideas for improvement

- Add more indicators (MACD, Bollinger Bands)
- Add a news-sentiment tool
- Backtest the scoring rules against historical data
- Build a simple web UI

## Project structure

```
Stock-Market-SAS/
├── stock_agent.py      # agent, tools, and CLI
├── requirements.txt
├── .gitignore          # includes stocks.db
└── README.md
```
