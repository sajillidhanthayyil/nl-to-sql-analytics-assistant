# Natural Language to SQL Analytics Assistant

Ask questions about a MySQL database in plain English and get back SQL, result tables and charts, either in the terminal or through a Telegram bot.

![Python](https://img.shields.io/badge/Python-3.9%2B-3776AB?logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-8-4479A1?logo=mysql&logoColor=white)
![Groq](https://img.shields.io/badge/LLM-Groq%20API-F55036)
![Telegram](https://img.shields.io/badge/Telegram-Bot-26A5E4?logo=telegram&logoColor=white)

## The problem

Business users often need numbers that live in a relational database, but they can't write SQL. Every question turns into a ticket for a developer or an analyst. This project lets them ask the question directly, for example "What were total sales per region last month?", and get the answer back as a table or a chart.

## Features

- **Schema-aware text-to-SQL.** At startup it reads the database schema from `information_schema` and gives it to the LLM, so the generated queries use real table and column names.
- **Confirm before running.** The assistant explains the query it generated and runs it only after the user types `confirm query`, so nothing reaches the database without a human check.
- **Remembers the conversation.** Follow-up messages such as "only for 2024" refine the current query. The Telegram bot keeps a separate history for each user.
- **Charts on request.** After a query runs, the user can ask for a bar, line or pie chart. The LLM writes the Pandas and Matplotlib code, and the chart comes back as an image.
- **Two interfaces:**
  - `chatbotmain.py` is an interactive command-line chat.
  - `telebot.py` is a Telegram bot that replies with result previews and chart images.
- **No secrets in the code.** API keys and database credentials are read from a `.env` file.

## Tech stack

| Area | Technology |
| --- | --- |
| Language | Python |
| LLM | Groq API (OpenAI-compatible chat completions; Llama 3 and Gemma 2 models) |
| Database | MySQL (`mysql-connector-python`) |
| Data and charts | Pandas, Matplotlib |
| Chat interface | Telegram Bot API (`python-telegram-bot`) |
| Workflow automation | n8n (conversational analytics and scheduled query flows) |

## How it works

```
User question ──► Groq LLM (system prompt includes the DB schema)
                     │
                     ▼
          Proposed SQL + plain-English explanation
                     │  user types "confirm query"
                     ▼
          Run on MySQL ──► Pandas DataFrame ──► table preview
                     │  user asks for a chart (bar / line / pie)
                     ▼
          LLM writes plot_graph() ──► Matplotlib ──► PNG sent back to the chat
```

## Getting started

### Prerequisites

- Python 3.9 or later
- A MySQL server with a database to query (the default name is `chatbot`)
- A [Groq API key](https://console.groq.com/keys)
- A Telegram bot token from [@BotFather](https://t.me/BotFather), needed only for the Telegram bot

### Installation

```bash
git clone https://github.com/sajillidhanthayyil/chatbot.git
cd chatbot
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

### Configuration

Copy `.env.example` to `.env` and fill in your values:

```env
GROQ_API_KEY=your_groq_api_key_here
TELEGRAM_BOT_TOKEN=your_telegram_bot_token_here
DB_HOST=localhost
DB_USER=root
DB_PASSWORD=your_db_password_here
DB_PORT=3306
DB_NAME=chatbot
```

> Use a **read-only** database user. The LLM writes the SQL, so the account it runs under should not be able to change data.

### Run

```bash
# Command-line chat
python chatbotmain.py

# Telegram bot
python telebot.py
```

Then send the bot `/start` on Telegram.

## Sample session

The example below uses a sample `sales` table. The output from your own database will differ.

```text
You: What is the total sales amount for each region?

Bot: Here is a query that adds up the sales amount for each region:

     SELECT region, SUM(amount) AS total_sales
     FROM sales
     GROUP BY region;

     It groups the sales table by region and sums the amount column.
     Is this description correct? Type "confirm query" to run it, or tell me what to change.

You: confirm query

Bot: Query executed. Here's a preview:
       region  total_sales
        North     482300.0
        South     391750.0
         East     305120.0
         West     287940.0

Bot: Would you like a graph representation? (yes/no)
You: yes
Bot: What type of graph? (bar, line, pie)
You: bar
Bot: [sends a bar chart of total_sales by region]
```

## Screenshots

To add screenshots, put the images in a `docs/` folder and link them here:

```markdown
![Telegram conversation](docs/telegram-chat.png)
![Generated bar chart](docs/bar-chart.png)
```

## Project structure

```
├── chatbotmain.py     # Command-line text-to-SQL chat
├── telebot.py         # Telegram bot interface
├── requirements.txt   # Python dependencies
└── .env.example       # Template for API keys and DB settings
```

## Limitations and next steps

- The chart code is written by the LLM and run as-is, so it should only be used in a trusted environment. Sandboxing that step is the next improvement.
- The schema is read once at startup. Restart the app after changing the database schema.
- Planned: allowing only `SELECT` queries, a web UI, and support for PostgreSQL.

## Author

**Sajil Lidhan**, Java Backend Engineer. See the [portfolio](https://sajillidhanthayyil.github.io).
