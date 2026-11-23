# 📈 Equity Portfolio Rebalancer

An autonomous **AI agent workflow built in [n8n](https://n8n.io)** that rebalances an investment portfolio from a plain-English instruction. Type something like *"Ensure the portfolio is 60% equity and 40% fixed income"* into a form, and the agent reads your portfolio from Google Sheets, pulls live market prices, decides on new position sizes, verifies its own work, and notifies you by email and push notification.

---

## ✨ Features

- **Natural-language input**: describe the target allocation in a simple web form.
- **Autonomous tool-using agent**: an OpenAI-powered agent plans and executes an 8-step process on its own.
- **Live market data**: fetches the latest end-of-day prices per ticker via Marketstack.
- **Self-verification loop**: the agent re-reads the sheet after rebalancing and keeps adjusting until the equity / fixed income split meets the goal.
- **Instant notifications**: a detailed trading-decisions email (Gmail) plus a concise push notification (Pushover).
- **Success / failure signalling**: a final `If` node checks that the agent completed the full task and sends a "Rebalancing Successful" or "Rebalancer Failed!" push.

---

## 🧠 How It Works

```
┌──────────────────┐     ┌──────────────────────────────────────────┐     ┌────────┐     ┌─────────────────────────┐
│ Form Submission  │ ──► │               AI Agent (GPT-4.1)          │ ──► │  If    │ ──► │ Pushover: Success / Fail │
│ (rebalance goal) │     │ Tools: Sheets · Marketstack · Gmail · Push│     │ == OK? │     └─────────────────────────┘
└──────────────────┘     └──────────────────────────────────────────┘     └────────┘
```

### The agent's 8-step mission

1. Read the portfolio from the Google Sheet
2. Fetch the latest prices for each position
3. Write the prices back to the sheet (`update_price`)
4. Decide on the rebalancing trades
5. Write the new quantities to the sheet (`update_new_quantity`)
6. Re-read the sheet to confirm the target allocation was achieved
7. Make further adjustments if the target isn't met
8. Send the trading-decisions email and a push notification

The agent replies with exactly `OK` only once every step is complete. The `If` node uses that as a completion check.

---

## 🧰 Tech Stack

| Component | Purpose |
|---|---|
| **n8n** | Workflow orchestration |
| **OpenAI GPT-4.1** | LLM powering the agent's reasoning |
| **n8n AI Agent (LangChain)** | Tool-calling agent framework |
| **Google Sheets** | Portfolio data store (read and write) |
| **Marketstack API** | Latest end-of-day stock prices |
| **Gmail** | Trading-decisions email |
| **Pushover** | Push notifications |

---

## 📊 Google Sheet Structure

The workflow expects a sheet with the following columns:

| Column | Description |
|---|---|
| `Ticker` | Security symbol (used as the matching key) |
| `Quantity` | Current number of units held |
| `Equity ratio` | Share of the position that counts as equity |
| `Fixed income ratio` | Share of the position that counts as fixed income |
| `Price` | Latest price (written by the agent) |
| `Total Value` | Current position value |
| `New Quantity After Rebalancing` | Target quantity (written by the agent) |
| `New Total Value` | Position value after rebalancing |

> 💡 `Total Value` and `New Total Value` work best as spreadsheet formulas so the agent can verify the split by simply re-reading the sheet.
