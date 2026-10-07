# AI Cricket Match Predictor Agent

**An IBM Granite-powered multi-agent cricket prediction system.**

> Academic Major Project | Placement Demonstration | AI · Django · RAG · ML

---

## Overview

The AI Cricket Match Predictor Agent is a complete web-based application that analyzes historical cricket match data using a **4-agent AI architecture** to generate intelligent predictions and explainable insights for upcoming cricket matches.

**Predicted teams include:** India, Australia, England, Pakistan, New Zealand, South Africa, West Indies, Sri Lanka.

---

## Architecture

```
User (Browser)
    ↓
Django Dashboard (HTML/CSS/JS + Chart.js)
    ↓
REST API Layer (Django REST Framework)
    ↓
┌────────────────────────────────────────────┐
│  Agent 1: Data Collection Agent            │
│  → Queries SQLite DB for match history     │
│  → Retrieves team/player/venue stats       │
└────────────────────┬───────────────────────┘
                     ↓
┌────────────────────────────────────────────┐
│  RAG Layer: Cricket Knowledge Base         │
│  → team_profiles.json                      │
│  → venue_profiles.json                     │
│  → player_profiles.json                    │
│  → Keyword retrieval (FAISS-ready)         │
└────────────────────┬───────────────────────┘
                     ↓
┌────────────────────────────────────────────┐
│  Agent 2: Performance Analysis Agent       │
│  → Recent form, H2H, batting/bowling       │
│  → Produces normalized feature vector     │
└────────────────────┬───────────────────────┘
                     ↓
┌────────────────────────────────────────────┐
│  Agent 3: Prediction Agent                 │
│  → RandomForest ML Model (trained on DB)  │
│  → Statistical fallback formula           │
│  → Win probability (0–100%)               │
└────────────────────┬───────────────────────┘
                     ↓
┌────────────────────────────────────────────┐
│  Agent 4: Insight Generation Agent         │
│  → IBM Granite via Groq API               │
│  → RAG-augmented prompt                   │
│  → Natural language explanation           │
└────────────────────┬───────────────────────┘
                     ↓
Interactive Dashboard (Win Probability, Charts, Analysis)
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| **Backend** | Python 3.11+, Django 4.2 |
| **Frontend** | HTML5, CSS3, JavaScript (ES6+), Chart.js |
| **Database** | SQLite (dev) → PostgreSQL-compatible |
| **Data Processing** | Pandas, NumPy |
| **ML Model** | scikit-learn RandomForestClassifier |
| **Generative AI** | IBM Granite via Groq API |
| **RAG** | JSON knowledge base + keyword retrieval (FAISS-upgradeable) |
| **API** | Django REST Framework |
| **Deployment** | Gunicorn + WhiteNoise |

---

## Folder Structure

```
cricket_predictor/
├── manage.py
├── requirements.txt
├── .env.example                  # Copy to .env and fill API keys
├── cricket_predictor/            # Django project settings
│   ├── settings.py
│   └── urls.py
├── apps/
│   ├── core/                     # Shared models (Team, Player, Venue, Match, Prediction)
│   ├── data_collection/          # Agent 1 — collects match data from DB
│   ├── analysis/                 # Agent 2 — performance analysis & feature engineering
│   ├── prediction/               # Agent 3 — ML model + statistical predictor
│   ├── insights/                 # Agent 4 — IBM Granite AI explanation
│   ├── rag/                      # RAG retriever + knowledge base JSONs
│   └── dashboard/                # Django views serving the HTML dashboard
├── static/
│   ├── css/dashboard.css         # Dark-mode responsive dashboard styles
│   └── js/dashboard.js           # Chart.js rendering + API calls
├── templates/
│   └── dashboard/index.html      # Main dashboard template
└── data/
    ├── models/                   # Saved ML model (.pkl)
    └── rag_index/                # FAISS index (optional, for semantic RAG)
```

---

## Database Schema

| Table | Key Fields |
|---|---|
| `core_team` | id, name, short_name, country_code, format |
| `core_player` | id, team_id, name, role, batting_style, bowling_style |
| `core_venue` | id, name, city, avg_first_innings_score, batting_first_win_pct |
| `core_match` | id, team1_id, team2_id, venue_id, format, match_date, winner_id, scores |
| `core_battingperformance` | id, match_id, player_id, runs, balls_faced, strike_rate |
| `core_bowlingperformance` | id, match_id, player_id, wickets, overs, economy |
| `core_prediction` | id, team1_id, team2_id, win_probabilities, ai_explanation, key_factors |

**To migrate to PostgreSQL:** Change `ENGINE` in `settings.py` to `django.db.backends.postgresql` and add connection details.

---

## Agents

### Agent 1 — Data Collection Agent (`apps/data_collection/agent.py`)
- Queries the database for historical ODI/T20/Test matches
- Retrieves head-to-head records, team form, top players
- Returns a structured data package

### Agent 2 — Performance Analysis Agent (`apps/analysis/agent.py`)
- Computes weighted form score (recent matches weighted more)
- Calculates batting/bowling strength indices
- Builds the 16-feature normalized feature vector

### Agent 3 — Prediction Agent (`apps/prediction/agent.py`)
- Loads the trained RandomForest model (`data/models/cricket_predictor.pkl`)
- Falls back to a weighted statistical formula if model not yet trained
- Returns win probabilities and confidence score

### Agent 4 — Insight Generation Agent (`apps/insights/agent.py`)
- Retrieves relevant context from the RAG knowledge base
- Builds a structured prompt with match statistics
- Calls IBM Granite (via Groq API) for a natural-language explanation
- Falls back to a template-based explanation if API is unavailable

---

## RAG Implementation

The RAG layer (`apps/rag/retriever.py`) retrieves relevant cricket context from three JSON knowledge bases:

- `team_profiles.json` — team strengths, history, playing styles
- `venue_profiles.json` — pitch characteristics, average scores, win patterns
- `player_profiles.json` — key player profiles and performance notes

**Current retrieval:** keyword overlap scoring (lightweight, zero dependencies)  
**Upgrade path:** Replace with FAISS + `sentence-transformers` for semantic retrieval (code scaffolded in `apps/rag/embeddings.py`)

---

## Prediction Model

**Algorithm:** RandomForestClassifier (scikit-learn)  
**Features (16 total):**
- Team recent form (weighted — last 5 matches, most recent = highest weight)
- Overall win rate
- Recent win rate (last 10 matches)
- Batting strength (average team score, normalized)
- Bowling penetration (avg wickets per match)
- Bowling economy score
- Top batter average (normalized)
- Head-to-head dominance ratio
- Venue batting-first win percentage

**Training:** `python manage.py train_model`  
**Fallback:** Weighted statistical formula (same features, fixed weights)

---

## Setup & Installation

### Prerequisites
- Python 3.10+
- pip

### 1. Clone and enter the project
```bash
cd cricket_predictor
```

### 2. Create and activate virtual environment
```bash
python -m venv venv
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment
```bash
copy .env.example .env       # Windows
# cp .env.example .env       # macOS/Linux
```
Edit `.env` and add your Groq API key:
```
GROQ_API_KEY=your_groq_api_key_here
LLM_MODEL=openai/gpt-oss-120b
SECRET_KEY=your-random-django-secret-key
```

### 5. Initialize the database
```bash
python manage.py migrate
```

### 6. Seed cricket data
```bash
python manage.py seed_data
```
This creates 8 teams, 64 players, 8 venues, and ~200 historical ODI matches.

### 7. Train the ML model
```bash
python manage.py train_model
```
Trains a RandomForestClassifier on the seeded match data.

### 8. Run the development server
```bash
python manage.py runserver
```
Open `http://127.0.0.1:8000` in your browser.

---

## Usage

1. Open the dashboard at `http://127.0.0.1:8000`
2. Select **Team 1** and **Team 2** from the dropdowns
3. Optionally select a **Venue** and **Format**
4. Click **⚡ Predict Match**
5. View:
   - Win probability bar
   - Predicted winner
   - Recent form badges
   - Head-to-head statistics
   - Batting and bowling comparison charts
   - Venue statistics
   - Key influencing factors
   - IBM Granite AI explanation

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/data/teams/` | List all teams |
| GET | `/api/data/venues/` | List all venues |
| POST | `/api/insights/full-prediction/` | Run full 4-agent prediction |
| POST | `/api/prediction/predict/` | Prediction only (no AI explanation) |
| POST | `/api/analysis/analyze/` | Analysis only |

**Full prediction POST body:**
```json
{
  "team1_id": 1,
  "team2_id": 2,
  "venue_id": 3,
  "format": "ODI"
}
```

---

## IBM Technologies

| Technology | Usage |
|---|---|
| **IBM Granite** | Natural language match analysis and explanation (via Groq API) |
| **IBM Cloud** | Deployment target for production |
| **IBM Bob** | AI assistant used for development |

---

## Production Notes

- Set `DEBUG=False` in `.env`
- Change `SECRET_KEY` to a strong random value
- Switch database to PostgreSQL: update `DATABASES` in `settings.py`
- Run `python manage.py collectstatic` for static files
- Deploy with Gunicorn behind Nginx

---

## Academic / Placement Notes

This project demonstrates:
- **Multi-agent AI architecture** (4 specialized agents)
- **RAG (Retrieval-Augmented Generation)** pipeline
- **ML model training and evaluation** (RandomForest, cross-validation)
- **Feature engineering** for sports prediction
- **Full-stack web development** (Django + responsive JS dashboard)
- **REST API design** (DRF)
- **IBM Granite integration** for explainable AI

---

*Predictions are statistical estimates based on historical data. Cricket is unpredictable — results on the day may differ.*
