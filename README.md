# BlockIntel — Crypto Intelligence Platform

A crypto intelligence platform for investigators and compliance teams to analyze blockchain addresses, surface illicit-activity risk signals, and manage investigation cases — built at Smart India Hackathon.

BlockIntel enriches wallet addresses with third-party intelligence (Nansen) and AI-driven risk classification (Gemini), then visualizes the results as an interactive network graph so analysts can trace fund flows and connections between flagged addresses.

## Features

- **Address risk analysis** — look up any address and get an enriched profile: labels, tags, transaction history, and an AI-generated risk assessment
- **AI-powered classification** — Gemini-based scoring flags suspicious patterns and explains its reasoning in plain language
- **Nansen enrichment** — pulls entity labels and wallet intelligence for known addresses
- **Interactive network graph** — visualize how addresses connect to each other (`react-force-graph-2d`) instead of reading raw transaction lists
- **Case management** — group flagged addresses into investigations and track them over time
- **Data export** — pull results out as JSON or CSV for reporting
- **Mock-safe fallback** — runs fully offline against seed data when API keys aren't configured, so the whole flow is demoable without live credentials

## Architecture

```
blockintel/
├── frontend/          # React + Vite + Tailwind dashboard
│   └── src/
│       ├── components/  # Dashboard, NetworkGraph, CaseManagement, AddressDetail, ...
│       └── app/          # routing entry points
├── backend/
│   ├── main.py          # FastAPI backend (primary — serves the built frontend too)
│   ├── server.js         # Express backend (alternate/legacy API surface)
│   ├── services/          # Nansen + Gemini client wrappers, JSON-file store
│   ├── data/                # seed address dataset
│   └── seed/                  # seed scripts
└── build-and-deploy.ps1     # builds frontend and copies it into backend/static
```

> Note: there are currently two parallel backend implementations (FastAPI in `main.py` and Express in `server.js`) covering overlapping endpoints. `main.py` is the primary one — it also serves the built frontend as static files. The Express server predates it; consolidating onto one is on the cleanup list.

## Tech Stack

| Layer | Stack |
|---|---|
| Frontend | React 18, Vite, Tailwind CSS, React Router, Recharts, react-force-graph-2d |
| Backend (primary) | FastAPI (Python), served via Uvicorn |
| Backend (alternate) | Express (Node), Helmet, CORS |
| Intelligence sources | Nansen API (address labeling), Google Gemini (AI risk classification) |
| Data | Local JSON store (`backend/data/addresses.json`) — no external DB required |

## Setup

### Backend (FastAPI — recommended)
```bash
cd backend
pip install -r requirements.txt
cp .env.example .env   # fill in NANSEN_API_KEY / GEMINI_API_KEY, or leave blank for mock mode
uvicorn main:app --reload --port 8000
```

### Frontend
```bash
cd frontend
npm install
npm run dev
# optional: create frontend/.env.local with VITE_API_BASE_URL=http://localhost:8000
```

### Build frontend into the backend (single-server deploy)
```bash
./build-and-deploy.ps1
# builds the frontend and copies dist/ into backend/static — FastAPI then serves everything from :8000
```

## API Reference (FastAPI, `backend/main.py`)

| Method | Endpoint | Description |
|---|---|---|
| GET | `/addresses` | List all addresses in the dataset |
| GET | `/address/{addr}` | Enriched detail for a single address (Nansen + Gemini) |
| GET | `/graph/{addr}` | Star-graph edges centered on an address, for the network view |
| GET | `/export/json` | Export the full dataset as JSON |
| GET | `/export/csv` | Export the full dataset as CSV |

`NANSEN_API_URL` and `GEMINI_API_URL` are placeholders in `.env.example` — point them at the real provider endpoints per their docs. If either key/URL is left blank, calls fall back to the local mock dataset so the app stays fully functional in a demo setting.

