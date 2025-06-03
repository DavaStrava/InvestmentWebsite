# InvestmentWebsite

# Investment Portfolio Website

## Overview

This repository contains the source and documentation for a personal, comprehensive investment portfolio website. The primary goal is to enable a single user (yourself) to:

* Track and manage your investment portfolio
* Monitor real-time market activity and relevant company data (e.g., earnings, news)
* Generate insights (charts, summaries) about market trends
* Leverage “investment agents” (programmatic trading assistants) to receive tips and strategies
* Simulate hypothetical portfolios and backtest strategies

While the long-term vision encompasses a full suite of analytics, agent-driven recommendations, and simulations, the **Minimum Viable Product (MVP)** focuses on:

1. Looking up and retrieving stock quote data
2. Persisting a watchlist/portfolio that you can create/update/delete
3. Displaying historical price charts and key financial metrics (e.g., earnings dates, P/E ratio)

This README outlines the project structure, MVP scope, architecture, technology stack, usage instructions, and a roadmap for future enhancements.

---

## Table of Contents

1. [MVP Features](#mvp-features)
2. [High-Level Architecture](#high-level-architecture)
3. [Technology Stack](#technology-stack)
4. [Data Sources & APIs](#data-sources--apis)
5. [Installation & Setup](#installation--setup)
6. [Usage & Configuration](#usage--configuration)
7. [Future Roadmap](#future-roadmap)
8. [Directory Structure](#directory-structure)
9. [Contributing & Personal Customization](#contributing--personal-customization)

---

## MVP Features

The MVP focuses on core functionality to get you started quickly. Subsequent sections outline how to extend each feature.

1. **User Authentication (Optional for MVP)**

   * Since the only user is yourself, authentication can be disabled or limited to a simple password-protected admin interface (e.g., HTTP basic auth or a single-user login).

2. **Stock Lookup & Real-Time Quotes**

   * A search bar (or ticker input) to fetch the latest price, day’s high/low, volume, etc.
   * Display a summary card for each lookup, showing:

     * Current Price
     * 52-Week High / Low
     * Market Capitalization
     * P/E Ratio
     * Earnings Date (next scheduled)

3. **Portfolio Management**

   * **Watchlist:** Create a simple “Favorites” view where you can add or remove tickers.
   * **Holdings Tracking:** Enter the number of shares purchased, purchase price, and purchase date.
   * Automatically calculate unrealized gain/loss per position and overall portfolio P\&L.

4. **Historical Performance Chart**

   * Integrate a charting library (e.g., Chart.js, Recharts) to plot historical price data (e.g., 1 mo, 6 mo, YTD, 1 yr).
   * Enable toggling between line chart, candlestick, or area chart views (basic implementation).

5. **Key Financial Data Section**

   * For each stock in your portfolio/watchlist, display:

     * Next Earnings Date
     * Recent Earnings (EPS, Revenue vs. consensus)
     * Dividend Yield (if applicable)
     * Analyst Consensus Price Target (if available)

6. **Basic Insights Dashboard (MVP)**

   * Show top 5 gainers/losers of the day (from a major index).
   * Display a small summary: e.g., “S\&P 500 is up 0.5% today.”
   * Provide a simple table of your portfolio’s performance vs. benchmark (e.g., S\&P 500).

---

## High-Level Architecture

Below is a proposed high-level architecture. You can adapt based on personal preference (monolith vs. microservices) and your AWS familiarity.

```
┌─────────────────────────────────────────────┐
│                  Frontend                  │
│  (React or Vue.js single-page application) │
│  • Portfolio Dashboard                     │
│  • Stock Lookup/Search                      │
│  • Historical Charts                       │
│  • Watchlist & Holdings UI                 │
└─────────────────────────────────────────────┘
                   ▲          │
                   │ HTTPS    │ REST / GraphQL
                   ▼          │
┌─────────────────────────────────────────────┐
│                  Backend                   │
│                                           │
│  ┌────────────┐   ┌───────────────────┐    │
│  │  API Layer │   │   Agent Module    │    │
│  │ (Node.js / │◀──│ (Python / Node.js │    │
│  │  Python /  │   │   scripts to call │    │
│  │  Go / .NET)│   │  trading APIs, ML │    │
│  └────────────┘   │  models, etc.)    │    │
│                    └───────────────────┘    │
│                                           │
│  ┌───────────────┐   ┌──────────────────┐  │
│  │ Data Ingestion│   │    Database      │  │
│  │  & Scheduler  │   │ (RDS / DynamoDB) │  │
│  │ (Cron jobs or │   │                  │  │
│  │  Lambda to    │   │ - User Portfolios│  │
│  │  refresh data)│   │ - Watchlists     │  │
│  │               │   │ - Historical Data│  │
│  └───────────────┘   └──────────────────┘  │
└─────────────────────────────────────────────┘
```

1. **Frontend**

   * A single-page application (SPA) built with React (preferred) or Vue.js.
   * Interfaces with the backend through a RESTful API or GraphQL.

2. **Backend & API Layer**

   * Exposes endpoints to:

     * Fetch real-time stock data
     * Fetch historical price/earnings data
     * Add/remove tickers from watchlist
     * CRUD portfolio positions
     * Retrieve portfolio analytics (gain/loss, allocation)

3. **Agent Module (Phase 2+ feature)**

   * A set of scripts or microservices (e.g., Python) that call third-party trading tip APIs or run simple ML-based signals.
   * Schedules periodic runs (e.g., daily at market close) to generate “tips” stored in a separate table or database collection.

4. **Data Ingestion & Scheduler**

   * Cron jobs or AWS Lambda functions to poll data sources (see [Data Sources & APIs](#data-sources--apis)).
   * Populate a local database cache of historical prices (e.g., daily OHLC) for quick chart rendering.

5. **Database**

   * Use a relational database (e.g., PostgreSQL on AWS RDS) or a NoSQL store (e.g., DynamoDB).
   * Tables/Collections for:

     1. **Users** (for MVP, a single user record or skip entirely)
     2. **WatchlistItems** (ticker symbols + user reference)
     3. **PortfolioPositions** (ticker, shares, avg\_price, purchase\_date)
     4. **HistoricalPrices** (ticker, date, open, high, low, close, volume)
     5. **AgentRecommendations** (ticker, date, recommendation\_text, confidence\_score)

---

## Technology Stack

Below is a suggested stack. You can swap components based on your preferences, but this aligns with AWS best practices and typical modern web applications.

* **Frontend**

  * Framework: React (with Create React App or Vite)
  * UI Library: Tailwind CSS or Chakra UI (for rapid styling)
  * Charting: Chart.js or Recharts for line/candlestick charts

* **Backend**

  * Language/Runtime: Node.js (Express or NestJS) or Python (Flask / FastAPI)
  * REST API or GraphQL (Apollo Server if using Node)
  * Authentication: JSON Web Tokens (JWT) or simple API key

* **Database**

  * Primary: PostgreSQL (Amazon RDS)
  * Secondary (optional/noSQL): DynamoDB for time-series stock data caching

* **Data Ingestion & Scheduling**

  * AWS Lambda functions triggered by EventBridge (cron expressions)
  * Scripts to fetch:

    * Historical OHLC data
    * Upcoming earnings dates
    * Analyst consensus metrics

* **Deployment & Hosting**

  * **Frontend**: AWS Amplify Hosting or S3 + CloudFront
  * **Backend**: AWS Elastic Beanstalk or containerized with ECS/Fargate
  * **Lambda Functions**: Use AWS Serverless Application Model (SAM) or Serverless Framework
  * **CI/CD**: GitHub Actions to:

    * Deploy frontend on push to `main`
    * Deploy backend services (ECS task or EB) on push to `main`

* **Data & Market API Providers**

  * **MVP Data**: [Alpha Vantage](https://www.alphavantage.co/) (free tier, rate-limited) or [IEX Cloud](https://iexcloud.io/) (paid, more reliable)
  * **Alternative**: \[Yahoo Finance unofficial endpoints / RapidAPI wrappers]
  * **Earnings & Financial Metrics**: [Finnhub](https://finnhub.io/) or [Financial Modeling Prep](https://financialmodelingprep.com/)

---

## Data Sources & APIs

| Purpose                     | Provider / API                                                      | Notes                                                                     |
| --------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| Real-time stock quotes      | IEX Cloud or Alpha Vantage                                          | IEX Cloud has free tier (50,000 messages/mo). AV rate limits 5 calls/min. |
| Historical OHLC data        | Alpha Vantage TIME\_SERIES\_DAILY\_ADJUSTED or IEX Cloud historical | Cache daily data in your DB to avoid repeated calls.                      |
| Earnings calendar & metrics | Financial Modeling Prep or Finnhub                                  | Setup a scheduled job to fetch upcoming earnings dates.                   |
| Analyst consensus & targets | Finnhub or FMP                                                      | Some endpoints require paid subscription.                                 |
| News / Market Sentiment     | Finnhub (News Endpoint) or NewsAPI                                  | Optional for MVP; can add as “Market Insights” section.                   |

**Key Considerations**

* **Rate Limits**: Be mindful of API call quotas. Implement a caching layer (e.g., local database) to reduce repeated hits.
* **API Key Security**: Store API keys/secrets in AWS Secrets Manager or environment variables (never hard-code).

---

## Installation & Setup

> **Prerequisites:**
>
> * Node.js (>=14.x) and npm (or Yarn)
> * Python 3.8+ (if using Python for backend/agent modules)
> * PostgreSQL database (local or AWS RDS)
> * AWS CLI configured (for Lambda/Amplify deploy, if using AWS)
> * API keys for your chosen data providers (Alpha Vantage, IEX Cloud, etc.)

1. **Clone the Repository**

   ```bash
   git clone https://github.com/<your-username>/InvestmentWebsite.git
   cd InvestmentWebsite
   ```

2. **Backend Setup**

   * Navigate to `/backend`
   * Copy `.env.example` → `.env` and populate:

     ```
     PORT=5000
     DATABASE_URL=postgres://<user>:<pass>@<host>:<port>/<db_name>
     ALPHA_VANTAGE_API_KEY=your_alpha_vantage_key
     IEX_CLOUD_API_KEY=your_iex_cloud_key
     ```
   * Install dependencies & run migrations (if applicable):

     ```bash
     cd backend
     npm install           # or `yarn install`
     npm run migrate       # if using a migration tool (e.g., Sequelize, TypeORM)
     npm run seed          # optional: seed sample data
     npm start             # starts Express/FastAPI server on PORT
     ```

3. **Frontend Setup**

   * Navigate to `/frontend`
   * Copy `.env.example` → `.env` and populate:

     ```
     REACT_APP_API_URL=https://api.yourdomain.com    # or http://localhost:5000
     ```
   * Install & start:

     ```bash
     cd frontend
     npm install           # or `yarn install`
     npm start             # starts dev server on http://localhost:3000
     ```

4. **Database Initialization**

   * Ensure your PostgreSQL instance is running.
   * Create a database (e.g., `investment_portfolio`).
   * Run migrations (see step 2).
   * Verify tables:

     * `users` (if used)
     * `watchlist_items`
     * `portfolio_positions`
     * `historical_prices`

5. **Scheduler / Lambda Functions (MVP)**

   * If you prefer local cron, set up a `cron` job to run `node scripts/fetch_historical.js` and `node scripts/fetch_earnings.js` daily.
   * If deploying to AWS:

     1. Navigate to `/lambdas`
     2. Configure `template.yaml` (SAM) or `serverless.yml` (Serverless Framework) with your environment variables.
     3. Deploy:

        ```bash
        sam build && sam deploy --guided   # for AWS SAM
        ```
   * Verify that scheduled Lambda (or cron) successfully populates the `historical_prices` and `earnings_calendar` tables.

---

## Usage & Configuration

1. **Running Locally**

   * Start backend: `cd backend && npm start`
   * Start frontend: `cd frontend && npm start`
   * Navigate to `http://localhost:3000` in your browser.

2. **Creating Your First Watchlist / Portfolio Entry**

   * In the UI, enter a valid ticker (e.g., `AAPL`) into the “Add Ticker” field.
   * Click “Add to Watchlist.”
   * For positions, navigate to “My Portfolio,” click “Add Position,” and provide:

     * Ticker: `AAPL`
     * Shares: `10`
     * Purchase Price: `150.00`
     * Purchase Date: `2025-05-15`

3. **Viewing Historical Charts**

   * Click on any ticker in your watchlist or portfolio.
   * The chart component will pull historical prices from your local DB (fallback: fetch from API if older than X days).
   * Toggle time range (1M / 6M / YTD / 1Y).

4. **Configuring Data Refresh**

   * By default, the “historical\_prices” table refresh runs once daily at 6 PM Eastern (configurable in `cron` or EventBridge rule).
   * Adjust the schedule in `/lambdas/template.yaml` (for AWS) or your local `crontab`.

5. **Environment Variables**

   * **Backend `.env`**

     ```ini
     PORT=5000
     DATABASE_URL=postgres://user:pass@host:port/dbname
     ALPHA_VANTAGE_API_KEY=your_key
     IEX_CLOUD_API_KEY=your_key
     ```
   * **Frontend `.env`**

     ```ini
     REACT_APP_API_URL=http://localhost:5000
     ```

---

## Directory Structure

```
InvestmentWebsite/
│
├── README.md                     # ← You are here
│
├── frontend/
│   ├── public/
│   ├── src/
│   │   ├── components/           # Reusable UI components
│   │   ├── pages/                # Page-level components (Dashboard, Watchlist, Portfolio)
│   │   ├── services/             # API client, data-fetching utilities
│   │   └── App.js
│   ├── .env.example
│   └── package.json
│
├── backend/
│   ├── src/
│   │   ├── controllers/          # Express / FastAPI route handlers
│   │   ├── models/               # Database models (Sequelize, TypeORM, or SQLAlchemy)
│   │   ├── routes/               # API route definitions
│   │   ├── services/             # Business logic & data-fetching from 3rd-party APIs
│   │   ├── scripts/              # Data ingestion scripts (fetch_historical.js, fetch_earnings.js)
│   │   └── index.js              # Main server entrypoint
│   ├── migrations/               # Database migration files
│   ├── seeders/                  # Seed data scripts (optional)
│   ├── .env.example
│   └── package.json
│
├── lambdas/                      # (Optional) AWS SAM / Serverless functions
│   ├── fetchHistorical/
│   │   ├── index.js
│   │   └── template.yaml
│   ├── fetchEarnings/
│   │   ├── index.js
│   │   └── template.yaml
│   └── .gitignore
│
└── docs/                         # Additional documentation (design docs, ER diagrams)
    └── architecture.md
```

---

## Future Roadmap

Below is a non-exhaustive list of features and enhancements to implement after the MVP. They are organized by priority.

### Phase 2 (Short-Term Enhancements)

1. **User Authentication & Profiles**

   * Implement OAuth (e.g., Google) or simple email/password login.
   * Support multiple portfolios per user (e.g., “Retirement” vs. “Taxable”).

2. **Real-Time Quote Streaming**

   * WebSocket or Server-Sent Events to push live quotes (e.g., IEX Cloud’s SSE).
   * Highlight intraday price changes in portfolio dashboard.

3. **Enhanced Dashboard & Analytics**

   * Portfolio allocation chart (pie chart showing sector breakdown).
   * Performance metrics: CAGR, Sharpe Ratio, Beta vs. benchmark.
   * Interactive watchlist alerts (e.g., price crosses above/below threshold).

4. **Investment Agents & Strategy Recommendations**

   * Simple rule-based agents (e.g., “If a stock’s 50-day MA crosses above 200-day MA → Suggest Buy”).
   * Integrate external “signal” APIs (e.g., TipRanks, Zacks).
   * Store agent recommendations with history and “thumbs up/down” feedback.

5. **Backtesting & Simulation Engine**

   * Allow creation of a “virtual portfolio” with historical rebalancing.
   * Backtest simple strategies (e.g., monthly rebalancing, equal-weight vs. market-cap).
   * Visualize backtest results vs. benchmark.

### Phase 3 (Medium-Term / Advanced)

1. **Options & Derivatives Tracking**

   * Enable logging of options positions (calls, puts), greeks, expiration dates.
   * Compute theoretical P\&L using Black-Scholes or other models.

2. **Advanced Charting & Annotations**

   * Candlestick, Bollinger Bands, RSI, MACD overlays.
   * Ability to draw trendlines/annotations on charts.

3. **News & Sentiment Analysis**

   * Integrate a news feed (e.g., Reuters, Bloomberg via APIs) for each ticker.
   * Implement basic sentiment analysis (e.g., positive/negative scoring) on news headlines.

4. **Mobile-Friendly / PWA Support**

   * Optimize UI for mobile devices.
   * Add PWA support (installable “app-like” experience) for quick portfolio check on the go.

5. **Machine Learning–Driven Insights**

   * Simple ML models to predict short-term price movements (e.g., logistic regression on technical indicators).
   * Anomaly detection for your portfolio (e.g., sudden uncharacteristic price swings).

6. **Tax Lot Accounting & Reporting**

   * Track multiple purchase lots and calculate cost basis (FIFO, LIFO).
   * Generate tax reports (e.g., realized gains/losses for a given tax year).

7. **Paper Trading Integration**

   * Connect to paper trading APIs (e.g., Alpaca, Interactive Brokers) to execute simulated trades.
   * Mirror simulated positions against historical data.

---

## Contributing & Personal Customization

> **Note:** Since this is a personal project, you may not need a formal “Contributing” section. However, if you plan to open-source or invite collaborators, consider adding:

1. **Code Style & Linting**

   * ESLint / Prettier for JavaScript/TypeScript
   * Black / Flake8 for Python (if applicable)

2. **Pull Request Workflow**

   * Feature branches named `feature/<description>`
   * Commit messages following Conventional Commits (e.g., `feat: add historical chart component`)

3. **Issue Templates**

   * Bug report, feature request templates to track requests

4. **Environment-Specific Configurations**

   * `.env.development`, `.env.production` for API keys and endpoints
   * CI/CD workflows that automatically swap environment variables

Feel free to modify any part of this project structure or roadmap to fit your personal workflow and priorities.

---

## Acknowledgments & References

* **APIs & Data Providers**

  * [Alpha Vantage](https://www.alphavantage.co/)
  * [IEX Cloud](https://iexcloud.io/)
  * [Finnhub](https://finnhub.io/)
  * [Financial Modeling Prep](https://financialmodelingprep.com/)

* **Libraries & Frameworks**

  * [React](https://reactjs.org/), [Vue.js](https://vuejs.org/)
  * [Express](https://expressjs.com/) / [FastAPI](https://fastapi.tiangolo.com/)
  * [AWS SAM](https://docs.aws.amazon.com/serverless-application-model/index.html)

* **UI / Charting**

  * [Tailwind CSS](https://tailwindcss.com/)
  * [Chart.js](https://www.chartjs.org/) / [Recharts](https://recharts.org/)

---

*Last updated: June 3, 2025*
