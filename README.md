# Real-Time Algorithmic Risk & Anomaly Alert Engine

## 📌 The Business Problem
Trading desks and risk managers face severe operational risk due to "dashboard fatigue." Traditional Business Intelligence (BI) dashboards are fundamentally passive—requiring human analysts to manually inspect reports to identify anomalies. In high-frequency and volatile financial markets, discovering liquidity crises, flash crashes, or sudden capital drawdowns hours after they occur leads to irreversible financial losses.

This project is an automated, event-driven anomaly detection and alert system engineered to replace passive reporting. It continuously monitors equity price action, computes dynamic statistical support boundaries, and dispatches instant outbound alerts to operational communication channels the second an asset breaches normal historical variance.

## 🛠️ Tech Stack
* **Core Engine & Statistical Modeling:** Python (Pandas, NumPy)
* **Market Data Ingestion:** REST API (Alpha Vantage via Requests)
* **Operational Delivery & Alerting:** Slack API (Incoming Webhooks, JSON payloads)
* **Configuration & Security:** `python-dotenv` for environment variable isolation
* **Development Environment:** Jupyter Notebook / Python 3.x

## ⚙️ The Pipeline Architecture

### 1. Market Data Ingestion & Baseline Modeling
The engine connects securely to Alpha Vantage to pull daily time-series equity data (HDFC Bank ADR - `HDB`). It cleans and standardizes the datetime index, enforces strict numeric datatypes for pricing series, and sorts the records chronologically to build an accurate rolling statistical baseline.

### 2. The Math Engine (Bollinger Band Modeling)
Rather than relying on brittle, arbitrary fixed-dollar drop thresholds, the pipeline models dynamic price volatility using institutional Bollinger Band statistical theory:
* **Trend Equilibrium:** Computes a 20-Day Simple Moving Average (SMA) to capture rolling price momentum and establish baseline central tendency.
* **Volatility Dispersion:** Calculates the 20-day rolling sample standard deviation to quantify active market volatility.
* **Lower Risk Floor:** Establishes the critical lower threshold: `Lower Risk Band = 20-Day SMA - (2 * 20-Day STD)`. Statistically, under a normal distribution, asset prices are expected to remain within these boundaries 95% of the time. A downward breach signals a high-severity statistical outlier.

### 3. The Anomaly Detection & Trigger Logic
The engine evaluates live intraday asset prices against the dynamically generated lower risk floor. To validate pipeline resilience against severe market stress, the system was stress-tested against a simulated flash crash (a sudden drop to $20.50 against a $21.92 risk floor).
* **Spread Calculation:** Computes the exact monetary deviation: `Deviation = Lower Risk Band - Current Price`.
* **State Trigger:** If `Current Price < Lower Risk Band`, the script halts routine execution and automatically initializes an emergency dispatch sequence.

### 4. Event-Driven Alert Delivery (Slack Webhooks)
To bypass human latency, the system constructs a structured, markdown-formatted JSON payload and dispatches it via an authenticated HTTP POST request to Slack's Incoming Webhook API. 

![Automated Slack Risk Alert](slack_alert.png)

* **Payload Structure:** Formats critical incident telemetry, including ticker symbol, breached price, baseline risk floor, and exact dollar deviation.
* **Operational Routing:** Pushes the alert with high-priority visual markers (siren emojis, bold callouts) into the dedicated `#risk-alerts` trading channel within milliseconds of detection.

## 🚀 Business Impact
This pipeline successfully transforms a passive monitoring workflow into an active risk mitigation engine. By detecting an acute flash crash where `HDB` plummeted $1.42 below its statistical floor ($20.50 live price vs. $21.92 floor), it proves that automated event-driven alerts protect capital by reducing incident notification latency from hours to sub-second execution.

## 💼 Financial Mechanics & Operational Handoff
To bridge the gap between quantitative engineering and live market operations, this architecture is designed around institutional risk workflows.

### 1. The Business Context
In capital markets, extreme statistical breakdowns often signal unexpected liquidity black holes, broader market contagion, or institutional execution liquidations. Quantitative desks cannot wait for end-of-day reconciliations; they require real-time alerts to pause automated buy orders, initiate delta hedging, or execute stop-loss protections.

### 2. The Metrics Translated
* **20-Day SMA ($22.84):** The rolling average price over roughly one trading month, representing the asset's baseline fair-value equilibrium.
* **Lower Risk Floor ($21.92):** The 2-sigma boundary. Breaching this level indicates selling volume that exceeds 95% of historical trading variance.
* **Live Price ($20.50):** The simulated intraday tick price evaluated against the baseline.
* **Deviation ($1.42):** The absolute monetary distance beyond statistical support, quantifying the severity of the market anomaly for desk supervisors.

### 3. Departmental Workflow (The Handoff)
* **Quantitative / Data Engineering (My Role):** Maintains the automated ingestion script, tunes rolling statistical window parameters, monitors webhook delivery health, and ensures API security.
* **Risk Management & Trading Desk:** Receives automated push notifications in Slack to execute discretionary stops, hedge active positions, or initiate immediate root-cause investigations.
