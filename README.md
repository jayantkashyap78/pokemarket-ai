# 🔮 PokeMarket.ai

> An AI agent that manages your Pokémon card collection through conversation — built with **Gemini**, **Google Cloud Agent Builder (ADK)**, and **MongoDB's MCP server**.

PokeMarket.ai goes beyond a chatbot: it *takes actions* on a real database. Ask it questions, add cards to your collection, value your portfolio, and track gain/loss — and it always pauses for your approval before changing anything. Built around **the newest Pokémon sets** so collectors can track cards the moment they release.

Built for the **Building Agents for Real-World Challenges** hackathon — **MongoDB partner track**.

🎥 **Demo video:** _<add your video link here>_

<!-- Add a screenshot: drop screenshot.png in the repo and it shows below -->
![PokeMarket.ai screenshot](screenshot.png)

---

## ✨ Features

- **Latest-set catalog** — searchable database of the newest Pokémon sets: **Mega Evolution, Ascended Heroes, Perfect Order, and Phantasmal Flames** — by name, set, number, rarity, type, and HP.
- **Collection management** — add, update, and remove the cards you own, with quantities consolidated cleanly (one record per card + condition, never duplicate rows).
- **Cost-basis tracking** — records what you paid per card.
- **Portfolio valuation** — reports total cards owned, total invested, current market value, and total gain/loss ($ and %).
- **Human-in-the-loop** — the agent describes every change and waits for your explicit approval before writing to the database.
- **Custom web UI** — a themed chat interface (PokeMarket.ai) that renders card images and tables.

---

## 🏗️ Architecture

```
┌─────────────┐     HTTP      ┌──────────────────┐
│ Web Frontend │ ───────────▶ │  ADK Agent       │
│ (pokemarket. │              │  (Gemini brain)  │
│  html)       │ ◀─────────── │                  │
└─────────────┘               └───────┬──────────┘
                                       │ tools
                          ┌────────────┴─────────────┐
                          ▼                          ▼
                  ┌────────────────┐        ┌─────────────────┐
                  │ MongoDB MCP    │        │ get_price tool  │
                  │ server →       │        │ (reads stored   │
                  │ Atlas (on GCP) │        │  market price)  │
                  └────────────────┘        └─────────────────┘
```

- **Brain:** Gemini, served via **Google Cloud Vertex AI**
- **Agent framework:** **ADK** (the dev kit inside Google Cloud Agent Builder)
- **Partner integration (MCP):** **MongoDB MCP server** for all database reads/writes
- **Database:** **MongoDB Atlas** (free M0 tier, hosted on a Google Cloud region)
- **Card data:** [TCGdex](https://tcgdex.dev) (free, open-source) — the four newest sets
- **Frontend:** single-file HTML/CSS/JS

Two MongoDB collections in one `pokemon` database:
- `cards` — the catalog (read-only), each with a stored `market_price`
- `holdings` — the cards the user owns

---

## 💰 A note on pricing

Pricing for **brand-new releases is genuinely sparse across all free data sources** (the newest chase cards simply have little market history yet). PokeMarket.ai handles this pragmatically: each card carries a **stored `market_price` in the database** — populated from live market data where it's available, and from **rarity-based estimates** for the newest cards that don't yet have market data. This keeps portfolio valuation working for every card while being transparent about where numbers come from. (Swapping in a paid real-time pricing source is a one-function change — see Future Work.)

---

## 📋 Prerequisites

- Python 3.11+
- Node.js 20.19+ (runs the MongoDB MCP server)
- A Google Cloud project with billing enabled and the **Vertex AI API** enabled
- A MongoDB Atlas account (free M0 cluster)
- `gcloud` CLI installed and authenticated

---

## ⚙️ Setup

**1. Clone and create a virtual environment**
```bash
git clone <your-repo-url>
cd pokemon-agent
python -m venv venv
venv\Scripts\activate          # Windows;  Mac/Linux: source venv/bin/activate
pip install -r requirements.txt
```

**2. Authenticate to Google Cloud (Vertex AI)**
```bash
gcloud auth application-default login
gcloud config set project YOUR_PROJECT_ID
gcloud services enable aiplatform.googleapis.com
```

**3. Create a `.env` file** in the project root (see `.env.example`):
```
GOOGLE_GENAI_USE_VERTEXAI=TRUE
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-central1
MDB_MCP_CONNECTION_STRING=mongodb+srv://USER:PASS@cluster0.xxxx.mongodb.net/pokemon
```
> ⚠️ **Never commit `.env`** — it contains secrets. It's already in `.gitignore`.

**4. Load the data**
```bash
python import_cards.py     # imports the 4 newest sets with full detail
python price_tool.py      # gives every card a market_price (live where available, else estimated)
```

---

## ▶️ Running the app

**Easiest (Windows):** double-click `start_pokemarket.bat` — it starts the agent, serves the page, and opens your browser.

**Manual (two terminals):**

Terminal 1 — the agent backend:
```bash
adk api_server --port 8000 --allow_origins=regex:.* .
```
Terminal 2 — the web server:
```bash
python -m http.server 5500
```
Then open: `http://127.0.0.1:5500/pokemon_agent/pokemarket.html`

> The **first** message is slow (the MCP server warms up). Send "how many cards are in the database?" once to warm it, then it's fast.

---

## 📁 Project structure

```
pokemon-agent/
├── pokemon_agent/
│   ├── __init__.py        # exposes the agent
│   ├── agent.py           # the ADK agent + instructions
│   ├── price_tool.py      # get_price() — reads stored market price
│   └── pokemarket.html    # the web frontend
├── import_cards.py         # imports the 4 newest sets from TCGdex
├── pokemarket.bat   # one-click launcher (Windows)
├── requirements.txt
├── .env                   # secrets (NOT committed)
└── .gitignore
```

---

## 💬 Example prompts

- "How many cards are in the database?"
- "Show me cards from the Mega Evolution set with their images."
- "Add 2 of [card] from Ascended Heroes, near-mint." *(agent asks the price, then pauses for approval)*
- "What's my collection worth, and how am I doing?"

---

## ⚖️ Data & legal

- Card data is sourced from TCGdex (open-source). Card pricing is provided by the PokeTrace API, which aggregates market data from TCGplayer, eBay, and CardMarket. Prices are used under PokeTrace's API terms of service. 
- This project is **not affiliated with, endorsed by, or sponsored by Nintendo or The Pokémon Company.** Pokémon and all related names are trademarks of their respective owners. This is a non-commercial hobbyist/hackathon project.

---

## 🚧 Future work

- Integrate a paid real-time pricing API (e.g. PokeTrace / PokemonPriceTracker) for accurate live values on the newest cards — a single-function swap in `price_tool.py`
- Streaming responses (`/run_sse`) for token-by-token replies
- Card-scanning via image recognition

---

## 🙏 Built with

Gemini · Google Cloud Agent Builder (ADK) · MongoDB Atlas + MCP · TCGdex