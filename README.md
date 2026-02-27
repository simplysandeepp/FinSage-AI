# 🦅 FinSage AI — **Multi-Agent Quantitative & Sentiment Financial Intelligence**

> A professional-grade, full-stack AI system that synthesizes parallel agent intelligence into actionable financial signals.

---

## 📋 Table of Contents
1. [What Is This Project?](#3-what-is-this-project)
2. [Who Is This For?](#4-who-is-this-for)
3. [Live Demo](#5-live-demo)
4. [Full System Architecture](#6-full-system-architecture)
5. [Phase 0: Offline Data & Training Pipeline](#7-phase-0-offline-data--training-pipeline)
6. [Phase 1: User Request Entry](#8-phase-1-user-request-entry)
7. [Phase 2: Orchestrator & Data Routing](#9-phase-2-orchestrator--data-routing)
8. [Phase 3: The 4 Parallel AI Agents](#10-phase-3-the-4-parallel-ai-agents)
9. [Phase 4: Ensembler (CIO Agent)](#11-phase-4-ensembler-cio-agent)
10. [Phase 5: Response Assembly & Audit](#12-phase-5-response-assembly--audit)
11. [Frontend Pages](#13-frontend-pages)
12. [LLM Client](#14-llm-client)
13. [Data Sources Explained](#15-data-sources-explained)
14. [Project File Structure](#16-project-file-structure)
15. [Environment Variables & Configuration](#17-environment-variables--configuration)
16. [How to Run Locally](#18-how-to-run-locally)
17. [API Reference](#19-api-reference)
18. [Database Schema](#20-database-schema)
19. [Tech Stack Summary](#21-tech-stack-summary)
20. [Known Limitations & Future Work](#22-known-limitations--future-work)
21. [Quick Reference Card](#23-quick-reference-card)

---

## 🔷 3. What Is This Project?
FinSage AI is a multi-modal financial analysis engine designed to bridge the gap between quantitative technical data and qualitative market sentiment. Unlike traditional trading bots that rely solely on price action, FinSage AI employs a **Committee of Agents** architecture to simulate a professional investment committee.

**Core Problems Solved:**
- **Information Overload:** Automatically synthesizes news, macro trends, and financial statements into a single report.
- **Siloed Analysis:** Breaks the barrier between "Numbers" (Quant) and "Narrative" (Sentiment).
- **Transparency:** Every decision is backed by an audit trail of agent reasoning, preventing "black box" AI behavior.
- **Model Drift:** Uses a specialized training pipeline with synthetic financial data to ensure robustness in volatile markets.

---

## 🧑‍💻 4. Who Is This For?
- **Quantitative Analysts:** Looking for a framework to orchestrate ML models with LLM reasoning.
- **Portfolio Managers:** Needing a high-level "second opinion" on tickers with summarized peer-benchmarking.
- **FinTech Developers:** Seeking a reference architecture for building production-grade multi-agent systems.

---

## 🖥️ 5. Live Demo
```ascii
+-----------------------------------------------------------+
| [Search: NVDA      ] [ Analyze ]                          |
+-----------------------------------------------------------+
|  QUICK SIGNAL: [ BUY ]  CONFIDENCE: 89%                   |
+-----------------------------------------------------------+
| AGENT INSIGHTS                                            |
| > Financial: "Strong FCF, Debt/Equity < 0.5"      [OK]    |
| > Sentiment: "Heavy bullish options flow detected" [HIGH]  |
| > Macro: "Semiconductor cyclical peak approaching" [WARN]  |
| > Peer: "Outperforming AMD/INTC in gross margins"  [OK]    |
+-----------------------------------------------------------+
| REVENUE FORECAST (XGBOOST)                                |
|  $ |          /--[Forecast Range]                         |
|    |     *---/                                            |
|    | *--/                                                 |
| ---+-----------------------------------> Time            |
+-----------------------------------------------------------+
```

---

## ⚙️ 6. Full System Architecture
```ascii
[ USER ] 
   │
   ▼
[ FRONTEND (Vite/React) ] ───▶ [ FASTAPI GATEWAY ]
                                     │
                                     ▼
                      [ ORCHESTRATOR (Parallel Exec) ]
                      ╱        │           │        ╲
          [ FIN AGENT ]  [ SENTI AGENT ] [ MACRO ] [ PEER ]
               │               │           │         │
               └───────────────┼───────────┼─────────┘
                               ▼
                        [ ENSEMBLER (CIO) ]
                               │
                               ▼
                    [ RESPONSE + AUDIT LOG ] ──▶ [ SQLITE ]
```

---

## 🧪 7. Phase 0: Offline Data & Training Pipeline
Before the first request, the system runs a training pipeline to calibrate the **XGBoost Quantile Regressor**.
1. **Synthetic Generation:** `generator.py` uses Geometric Brownian Motion (GBM) and AR(1) models to create 10,000+ financial scenarios.
2. **Feature Engineering:** `feature_store.py` calculates rolling volatility, RSI, and fundamental ratios.
3. **Model Training:** `xgboost_model.py` trains on synthetic + historical `yfinance` data to predict the next quarter's revenue and risk variance.

---

## 📥 8. Phase 1: User Request Entry
The user provides a ticker (e.g., `AAPL`) and a timeframe.
- **Validation:** Frontend hooks validate ticker existence via `yfinance`.
- **Packet Construction:** A request payload containing the ticker, scope, and optional flags is sent to `/forecast`.

---

## 🛣️ 9. Phase 2: Orchestrator & Data Routing
The `orchestrator.py` module receives the request and:
1. Fetches "Ground Truth" data via `yfinance_fetcher.py`.
2. Spins up 4 parallel threads/tasks, each hosting one specialized AI Agent.
3. Injects relevant data slices into each agent's context.

---

## 🧠 10. Phase 3: The 4 Parallel AI Agents

### 📊 Financial Agent
- **Input:** Balance Sheet, Income Statement, Cash Flow (3 years).
- **Output:** Financial health score (0-100) + textual summary of solvency.
- **LLM:** Groq Llama-3-70b (Primary).
- **Fallback:** Rule-based ratio check if LLM times out.

### 🎭 Sentiment Agent
- **Input:** Recent news headlines + social media snippets.
- **Output:** Sentiment polarity (-1 to 1) + key narrative drivers.
- **LLM:** Gemini 1.5 Flash.
- **Fallback:** NLTK/VADER local sentiment analysis.

### 🌎 Macro Agent
- **Input:** Sector-specific macro indicators (CPI, Rates).
- **Output:** Sector tailwind/headwind assessment.
- **LLM:** Groq Llama-3-8b.
- **Fallback:** Static sector-lookup table.

### 🏁 Peer Agent
- **Input:** Ticker + list of competitors.
- **Output:** Comparative ranking on P/E, PEG, and Growth.
- **LLM:** Gemini 1.5 Flash.
- **Fallback:** Top-3 peer average comparison.

---

## 🗳️ 11. Phase 4: Ensembler (CIO Agent)
The **Chief Investment Officer (CIO) Agent** acts as the final judge.
- **Logic:** It receives the 4 JSON objects from the agents.
- **Synthesis:** It weights the insights based on historical accuracy of each agent type for that specific sector.
- **Signal:** Generates the final `{signal: 'BUY', confidence: 0.85, reasoning: '...'}`.

---

## 📂 12. Phase 5: Response Assembly & Audit
1. **JSON Assembly:** All agent outputs are merged into a comprehensive state object.
2. **Persistence:** The `audit/db.py` saves the prompt/response/signal into `audit_trail.sqlite` for later review.
3. **Response:** The frontend receives the full payload to render the dashboard.

---

## 🖼️ 13. Frontend Pages
- **🏠 Home:** Clean search interface with "Market Pulse" ticker.
- **📈 Results:** Interactive Recharts dashboard showing history + predicted forecast.
- **⚖️ Audit:** Table view of previous analyses with "View Reasoning" modals.
- **👥 Peers:** Side-by-side comparison cards for the current ticker vs its industry.

---

## 🔌 14. LLM Client
The system uses a **Rotation & Fallback** strategy:
- **Groq (Llama 3)** is preferred for speed and quantitative reasoning.
- **Gemini 1.5** is used for large context window news analysis.
- If an API key hits a rate limit, the system transparently switches to the other provider or a local `tiny-llama` instance (optional).

---

## 📉 15. Data Sources Explained
- **yfinance:** Primary source for technical and fundamental data. (Latency: ~1s).
- **Alpha Vantage:** Fallback for macro data/FX rates.
- **Synthetic Generator:** Used for testing "Edge Case" market crashes that haven't happened yet.

---

## 📁 16. Project File Structure
```ascii
finsage-ai/
├── backend/
│   ├── agents/               # 🧠 Individual AI specialized logic
│   ├── orchestrator/         # 🚦 Parallel execution & Signal Ensembling
│   ├── models/               # 📉 XGBoost forecasting logic
│   ├── data/                 # 📡 Live API fetchers
│   ├── llm/                  # 🔌 Multi-provider client (Groq/Gemini)
│   ├── audit/                # ⚖️ SQLite logging for accountability
│   └── out/                  # 🧊 ML Artifacts (pkl/json)
├── frontend/
│   ├── src/
│   │   ├── components/       # 🧩 Reusable UI units (Cards, Charts)
│   │   ├── pages/            # 📄 Main route views
│   │   └── hooks/            # 🎣 API & State management
└── .env                      # 🔑 API Keys (Not committed)
```

---

## 🔑 17. Environment Variables & Configuration
| Key | Required | Purpose |
| :--- | :--- | :--- |
| `GROQ_API_KEY` | Yes | High-speed LLM inference |
| `GEMINI_API_KEY` | Yes | News/Text analysis |
| `DATABASE_URL` | No | SQLite path (defaults to local) |
| `ML_THRESHOLD` | No | Confidence cutoff for signals |

---

## 🚀 18. How to Run Locally
1. **Clone the Repo:** `git clone https://github.com/your-repo/finsage-ai.git`
2. **Backend Setup:**
   ```bash
   cd backend
   python -m venv venv
   source venv/bin/activate  # venv\Scripts\activate on Windows
   pip install -r requirements.txt
   ```
3. **Frontend Setup:**
   ```bash
   cd ../frontend
   npm install
   ```
4. **Environment:** Copy `.env.example` to `.env` and add your keys.
5. **Start:** Run `python train_pipeline.py` then `npm run dev`.

---

## 📡 19. API Reference
| Method | Path | Description | Response |
| :--- | :--- | :--- | :--- |
| POST | `/forecast` | Run multi-agent analysis | `JSON (Signal + Agent Data)` |
| GET | `/audit` | Get historical reports | `Array[AuditRecord]` |
| GET | `/health` | Server status check | `{status: 'ok'}` |

---

## 🗄️ 20. Database Schema (Audit Trail)
| Column | Type | Description |
| :--- | :--- | :--- |
| `id` | INTEGER | Primary Key |
| `ticker` | TEXT | Stock ticker |
| `signal` | TEXT | BUY/HOLD/SELL |
| `agent_logs` | JSON | Full trace of agent responses |
| `timestamp` | DATETIME | Time of generation |

---

## 🛠️ 21. Tech Stack Summary
**Backend**
| Tool | Purpose | Version |
| :--- | :--- | :--- |
| FastAPI | Web Framework | 0.95+ |
| XGBoost | Forecasting | 1.7+ |
| SQLAlchemy | ORM / Audit | 2.0+ |

**Frontend**
| Tool | Purpose | Version |
| :--- | :--- | :--- |
| React | UI Library | 18.2 |
| Tailwind | Styling | 3.3 |
| Recharts | Visualization | 2.5 |

---

## ⚠️ 22. Known Limitations & Future Work
**Limitations**
- `yfinance` can sometimes be throttled by Yahoo Finance servers.
- Sentiment analysis is limited by the quality of free news scrapers.

**Future Roadmap**
- [ ] Integration with real-time WebSocket price feeds.
- [ ] Discord/Slack bot for signal alerts.
- [ ] Portfolio optimization (Mean-Variance) agent.

---

## ⚡ 23. Quick Reference Card
```ascii
+-----------------------------------------------------------+
| FIN-SAGE AI DEVELOPER CHEATSHEET                         |
+-----------------------------------------------------------+
| INSTALL: npm install && pip install -r requirements.txt   |
| TRAIN:   python backend/train_pipeline.py                 |
| SERVE:   uvicorn backend.orchestrator.api:app --reload    |
| FRONT:   npm run dev                                      |
| TEST:    pytest backend/tests                             |
+-----------------------------------------------------------+
```
