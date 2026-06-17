# 📈 SMA Crossover Trading Engine

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![License](https://img.shields.io/badge/License-MIT-green)
![Mode](https://img.shields.io/badge/Trading-Paper%20Only-orange)

A continuous, event-driven **paper-trading bot** that trades the S&P 500 ETF (`SPY`) using a dual **Simple Moving Average (SMA) crossover** strategy, executed live against the Alpaca paper-trading API.

Built in pure Python with a minimal dependency footprint, the engine bootstraps from historical market data, maintains a rolling price window in memory, and autonomously submits orders when momentum shifts, all while keeping API credentials isolated and trading risk contained to a paper account.

> ⚠️ **Educational project.** This runs exclusively against Alpaca's **paper-trading** environment. It is not financial advice and is not intended for live capital.

---

## 🎯 What this project demonstrates

- Integrating a third-party financial REST API (Alpaca **Market Data v2** + **Trading API**) end to end
- Designing a stateful, long-running execution loop: **bootstrap → poll → compute → act**
- Implementing a moving-average crossover strategy with **position-state tracking** to prevent duplicate orders
- Handling UTC time zones, historical data windows, and rate-limit-friendly polling
- Managing secrets with **environment variables** instead of hardcoded keys
- Clean startup and graceful shutdown of a continuous process

---

## ⚙️ How it works

The engine runs in two phases:

**1. Historical bootstrap.** On startup it pulls recent 1-minute bars for `SPY` from Alpaca's Market Data v2 API (UTC, IEX feed) and seeds a rolling window with the most recent **170 closing prices**. This gives the moving averages enough history to be meaningful from the very first tick.

**2. Live polling loop.** Every 60 seconds the engine requests the latest quote, appends the new price to the rolling window, drops the oldest entry, and recomputes both moving averages:

| Average | Window | Role |
|---|---|---|
| Fast | 25-period SMA | Reacts quickly to recent price action |
| Slow | 170-period SMA | Represents the broader trend |

**Trading logic (long-only):**

- 🟢 **Buy** when the 25-SMA crosses **above** the 170-SMA (upward momentum) and no position is open.
- 🔴 **Liquidate** when the 25-SMA crosses **below** the 170-SMA (downward momentum) and a position is open.

A position-state flag ensures the engine never stacks duplicate buys and never accidentally goes net short. Orders are submitted as **market orders** to Alpaca's paper-trading endpoint.

---

## 🛠️ Tech stack

- **Language:** Python 3.9+
- **APIs:** Alpaca Trading API, Alpaca Market Data API v2
- **Libraries:** `requests`, `python-dotenv` (plus standard library: `datetime`, `json`, `time`, `os`)

---

## 🚀 Setup & installation

### 1. Prerequisites
- Python 3.9+
- A free Alpaca account with **paper-trading** API keys → [alpaca.markets](https://alpaca.markets)

### 2. Clone and install
```bash
git clone https://github.com/EhsanDevv/alpaca-sma-trader.git
cd alpaca-sma-trader

python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

pip install -r requirements.txt
```

**`requirements.txt`**
```
requests
python-dotenv
```

### 3. Configure credentials
Create a `.env` file in the project root:
```
ALPACA_API_KEY=your_paper_api_key
ALPACA_SECRET_KEY=your_paper_secret_key
```
> Add `.env` to your `.gitignore` so credentials are never committed.

### 4. Run
```bash
python3 main.py
```

**Sample output**
```
SMA_170: 748.79
SMA_25:  749.65
Buy order placed
...
^C
Terminating process: Shutting down
```
Stop the engine at any time with `Ctrl+C` for a clean shutdown.

---

## 📁 Project structure
```
alpaca-sma-trader/
├── main.py            # Engine: bootstrap, polling loop, strategy, order execution
├── .env               # API credentials (git-ignored, not committed)
└── README.md
```

---

## 📊 Strategy notes

The 25/170 SMA crossover is a classic **momentum** approach: the short average reacts faster than the long one, so a cross signals a shift in trend direction. The engine is deliberately **long-only** — it opens and closes long positions but never goes net short — to avoid whipsaw losses during rapid reversals.

---

## 🧭 Limitations & roadmap

This is a focused proof of concept. The following are known limitations and the planned direction — included deliberately, because knowing the edges of a system is part of building one:

- **In-memory state.** Position state lives in the running process and resets on restart.
  *Planned: reconcile against Alpaca's positions endpoint on startup.*
- **Polling, not streaming.** Prices are pulled on a 60-second REST cycle.
  *Planned: migrate to Alpaca's WebSocket market-data stream for lower latency.*
- **IEX data feed.** The free IEX feed reflects only IEX volume, not full-market NBBO.
  *Planned: optional SIP feed support.*
- **No backtesting harness.** The strategy is currently validated live on paper only.
  *Planned: a historical backtest mode to tune parameters before deployment.*
- **Minimal error handling.** A single network hiccup can interrupt the loop.
  *Planned: request retries with exponential backoff, market-hours awareness via Alpaca's clock endpoint, and structured logging.*

---

## 📄 License

Released under the MIT License. See `LICENSE` for details.
