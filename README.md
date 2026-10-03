# Financial News Sentiment Analyzer

A production-ready system for automated financial news monitoring, sentiment classification, and AI-powered intelligence generation.

**Automatically collects → analyzes → summarizes → visualizes** financial news sentiment for companies over time.

---

## 🎯 Problem Statement

Financial analysts and investors face **information overload**. Companies generate thousands of news articles monthly. Key question: *How is sentiment trending? What events caused sentiment shifts?*

**Manual approach:** Read articles one-by-one, use ChatGPT per article, no historical tracking.

**Our system:** Automated collection → FinBERT classification → AI summarization → interactive dashboard with historical persistence.

---

## ✨ Core Features

### 1. **Automated News Collection & Processing**
- Fetches financial news from **GDELT** (unlimited historical depth, real-time coverage)
- Enriches articles with metadata descriptions (via BeautifulSoup)
- Deduplicates automatically
- Cleans and normalizes data

### 2. **FinBERT Sentiment Classification**
- Domain-specific financial sentiment analysis
- Classifies each article: **Positive | Negative | Neutral**
- Confidence scores with each classification
- Batch processing for speed

### 3. **Daily Aggregation & Shift Detection**
- Aggregates per-article sentiment into daily scores
- Automatically detects **significant sentiment shifts** (threshold-based or statistical)
- Tracks shift magnitude and direction

### 4. **AI-Powered Explanations**
- **Overall Summary:** Gemini/OpenAI summarizes key developments from the period
- **Shift Explanations:** LLM explains what caused each detected sentiment shift
- On-demand generation (not computed unless user requests)

### 5. **Interactive Dashboard**
- **Sentiment Trend Chart:** Line chart showing sentiment evolution over time
- **Sentiment Distribution:** Pie/donut chart (positive/negative/neutral breakdown)
- **Shift Timeline:** Cards explaining each significant sentiment change
- Responsive design (desktop-first, mobile-friendly)

### 6. **Intelligent Caching**
- **Session State:** Instant results on repeat tab clicks within same session
- **Backend Cache:** Results persist across page refreshes and users
- **Cache Strategy:** Historical data cached forever, recent data refreshed every 4 hours
- Cost-efficient: Second user queries same company/range = instant, $0 LLM cost

---

## 🏗️ Architecture

┌──────────────────────────────────────────────────────┐
│ STREAMLIT UI (Dashboard) │
│ ┌─ Sentiment Trend ─ Overview ─ Explain Shifts ┐ │
└──┴──────────────────────────────────────────────┬───┘
│
├─ Session State Cache
│ (graphs, summaries, shift explanations)
│
└─ Backend Cache (Redis/SQLite)
│
└─ Pipeline Service
│
├─ 1. GDELT Fetcher
│ └─ Fetch articles by company + date range
│
├─ 2. Article Enricher
│ └─ Parallel HTTP: fetch meta descriptions
│
├─ 3. FinBERT Sentiment Analyzer
│ └─ Batch sentiment classification
│
├─ 4. Data Aggregator
│ └─ Daily sentiment + shift detection
│
├─ 5. OpenAI Summarizer (On-Demand)
│ ├─ Overall summary
│ └─ Shift explanations
│
└─ 6. SQLite Database
├─ articles (raw + scored)
├─ daily_sentiment (aggregates)
└─ sentiment_shifts (with explanations)


### Data Flow

GDELT API
↓ (metadata only: title, URL, domain, date)
Article Enricher (BeautifulSoup)
↓ (fetch og:description tags in parallel)
Clean & Deduplicate
↓
FinBERT Sentiment Scoring
↓ (batch processing)
Daily Aggregation
↓
Shift Detection
↓
Database Persistence (3 tables)
↓
Streamlit Dashboard (3 tabs)
├─ Tab 1: Charts (no LLM) → instant
├─ Tab 2: Summary (OpenAI) → on-demand, cached
└─ Tab 3: Shifts (OpenAI) → on-demand, cached


---

## 🛠️ Tech Stack & Design Rationale

| Component | Technology | Why? |
|-----------|-----------|------|
| **News Source** | GDELT (artlist API) | Unlimited volume, historical depth, free. Pre-cleaned APIs hide real data work. |
| **Sentiment Model** | FinBERT (ProsusAI) | Domain-specific to finance, pre-trained on 3.5B financial texts, runs locally (no API calls). |
| **LLM (Summaries)** | OpenAI GPT-3.5/4o-mini | Fast, cheap token pricing, large context window, on-demand only (no pipeline bloat). |
| **Enrichment** | BeautifulSoup + Requests | Lightweight, fetches meta descriptions (not full articles). Parallel HTTP for speed. |
| **Aggregation** | Pandas | Powerful groupby/rolling averages, industry standard. |
| **Visualization** | Plotly + Streamlit | Interactive charts, instant deployment, no frontend code needed. |
| **Database** | SQLite | Lightweight, portable, sufficient for single-user/small-team scale. |
| **Caching** | Session State + Backend | Session for instant repeat clicks, backend for persistence across page refreshes. |

---

## 📊 Database Schema

### `articles` table
```sql
article_id (PK)
company_name
title
url
published_date
source
language
text_for_sentiment (title + meta description)
sentiment_label (positive/negative/neutral)
sentiment_score (float, -1 to +1)
created_at (TIMESTAMP)
```

### `daily_sentiment` table
```sql
id (PK)
company_name
date
avg_sentiment (float, -1 to +1)
article_count
positive_count
negative_count
neutral_count
created_at (TIMESTAMP)
UNIQUE(company_name, date)
```

### `sentiment_shifts` table
```sql
shift_id (PK)
company_name
date
direction (positive/negative)
magnitude (float, 0-1)
summary (TEXT, NULL until user requests explanation)
created_at (TIMESTAMP)
updated_at (TIMESTAMP)
UNIQUE(company_name, date)
```

---

## 🚀 Quick Start

### Prerequisites
- Python 3.9+
- pip

### Installation

```bash
# Clone the repository
git clone https://github.com/Harshada-pande/financial-news-sentiment-analyzer.git
cd financial-news-sentiment-analyzer

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Get API keys
# 1. GNews: https://gnews.io/dashboard (free tier)
# 2. OpenAI: https://platform.openai.com/account/api-keys
```

### Configuration

Create `.env` file in project root:

GNEWS_API_KEY=your_gnews_key_here
OPENAI_API_KEY=your_openai_key_here
OPENAI_MODEL=gpt-3.5-turbo


### Run

```bash
streamlit run main.py
```

Opens at `http://localhost:8501`

---

## 📖 Usage Guide

### Basic Workflow

1. **Select Company & Date Range**
   - Choose company name (e.g., "Apple")
   - Select date range (last 7 days, last 30 days, custom)
   - Click "Analyze"

2. **Pipeline Runs** (~45 seconds)
   - Fetches articles from GDELT
   - Enriches with meta descriptions (parallel HTTP)
   - Scores with FinBERT
   - Aggregates daily sentiment
   - Detects shifts
   - Saves to database

3. **View Results** (Interactive Dashboard)
   - **Tab 1: Sentiment Trend**
     - Line chart: daily sentiment over time
     - Pie chart: positive/negative/neutral distribution
     - Instant (no LLM cost)
   
   - **Tab 2: Overview**
     - AI-generated summary of key developments
     - Click to generate (OpenAI), results cached
     - Reuse across sessions
   
   - **Tab 3: Explain Shifts**
     - List of detected sentiment shifts
     - Click to generate explanations (OpenAI batch call)
     - Explains what caused each shift
     - Cached permanently (same shift always same explanation)

---

## 🎓 Learning Outcomes

### What You'll Learn By Building This

**API Integration:**
- REST API design (GDELT, OpenAI)
- Request/response handling
- JSON parsing
- Rate limiting and timeouts

**Data Processing:**
- ETL pipeline design
- Deduplication logic
- Data cleaning and normalization
- Pandas groupby/aggregation
- Time-series data handling

**Machine Learning:**
- NLP fundamentals (tokenization, embeddings)
- Pre-trained models (FinBERT)
- Inference vs training
- Batch processing

**Database Design:**
- SQLite schema design
- Primary/foreign keys
- Indexing for query performance
- Normalization principles

**Backend Architecture:**
- Caching strategies (session + persistent)
- TTL-based cache invalidation
- Async vs sync processing
- Cost optimization (LLM calls)

**Frontend:**
- Streamlit app structure
- Multi-page navigation
- Interactive charts (Plotly)
- Session state management

**Software Engineering:**
- Modular code structure
- Single responsibility principle
- Error handling
- Logging and debugging

---

## 📁 Project Structure

financial-news-sentiment-analyzer/
├── main.py # Streamlit entry point
├── config.py # Configuration (API keys, constants)
├── requirements.txt # Python dependencies
├── .env # API keys (git-ignored)
├── .gitignore
│
├── src/
│ ├── init.py
│ ├── gdelt_fetcher.py # GDELT API integration
│ ├── article_enricher.py # Metadata description fetching (parallel)
│ ├── sentiment_analyzer.py # FinBERT batch scoring
│ ├── data_aggregator.py # Daily aggregation + shift detection
│ ├── summarizer.py # OpenAI API calls (on-demand)
│ ├── database.py # SQLite operations
│ ├── pipeline.py # Orchestrates all components
│ └── visualizer.py # Plotly charts
│
├── data/
│ └── articles.db # SQLite database (created at runtime)
│
└── README.md # This file


---

## ⚙️ Caching Strategy

### Session State Cache
- **Scope:** Current browser session (until page refresh)
- **Use Case:** Instant results on repeat tab clicks
- **Example:** User clicks "Sentiment Trend" tab → instant. Clicks back to "Overview" → instant. Both cached in memory.

### Backend Cache (SQLite/Redis)
- **Scope:** Persistent across page refreshes and users
- **TTL Strategy (Option C):**
  - Historical data (end_date < yesterday): **Cache forever**
  - Recent data (yesterday/today): **Cache 4 hours**
  - Example: "Apple, Jan 1-31" (all past) → cached permanently. "Apple, last 7 days" (includes today) → refresh every 4 hours.

### Cost Impact
- First user, first query: Pays for LLM calls
- Same user, repeat tab click: $0 (session state, instant)
- Page refresh: $0 (backend cache, instant)
- Different user, same query: $0 (backend cache, instant)
- **Result:** For repeated queries, cost amortizes to ~$0.01 per additional user

---

## 🔄 Workflow Timeline

PIPELINE PHASE (Unconditional):
├─ GDELT fetch (5 sec)
├─ Metadata enrichment, parallel (20 sec)
├─ FinBERT scoring, batch (15 sec)
├─ Aggregation + shift detection (1 sec)
└─ Database save (2 sec)
└─ Total: ~45 seconds

UI PHASE (On-Demand):
├─ Tab 1 - Charts: ~1 sec (no LLM)
├─ Tab 2 - Summary: ~5 sec (LLM, first time) OR instant (cached)
└─ Tab 3 - Shifts: ~3 sec (LLM, first time) OR instant (cached)


---

## 🎯 Design Philosophy

### What This Project IS
- ✅ **Automated intelligence pipeline** for financial news
- ✅ **Real-world data processing** (GDELT, enrichment, dedup)
- ✅ **Domain-specific ML** (FinBERT, not generic sentiment)
- ✅ **Practical caching** (reduce LLM costs, interview-level thinking)
- ✅ **Modular architecture** (each component replaceable)

### What This Project IS NOT
- ❌ Stock price predictor
- ❌ Investment advisor
- ❌ Claim that sentiment causes prices
- ❌ Generic ChatGPT wrapper
- ❌ Unnecessarily complex (microservices, Docker, Kubernetes when not needed)

---

## 🚧 Future Enhancements

### Phase 2: Scaling
- [ ] Add Redis for distributed caching
- [ ] Horizontal scaling (multiple FinBERT workers)
- [ ] Message queue (SQS/Kafka) for async processing
- [ ] PostgreSQL for better concurrency

### Phase 3: Analytics
- [ ] Sentiment-stock price correlation (if applicable)
- [ ] Keyword/entity extraction (what drives sentiment)
- [ ] Source credibility scoring
- [ ] Sentiment prediction (ML model)

### Phase 4: Production
- [ ] Authentication (JWT, role-based access)
- [ ] API endpoint (FastAPI)
- [ ] Monitoring/logging (structured logs)
- [ ] Rate limiting
- [ ] Docker + cloud deployment

---

## 🤝 Contributing

This is a portfolio project. Contributions welcome for:
- Bug fixes
- Performance improvements
- Better error handling
- Documentation
- Feature suggestions

### Development Workflow
```bash
git checkout -b feature/your-feature
git commit -m "Add feature: description"
git push origin feature/your-feature
# Create pull request
```

---

## 📝 License

MIT License — see LICENSE file for details.

---

## 👤 Author

**Harshada Pande**
- BTech Computer Engineering, Cummins College
- Email: harshada.pande@cumminscollege.in
- LinkedIn: [Your LinkedIn](https://linkedin.com/in/your-profile)
- GitHub: [@Harshada-pande](https://github.com/Harshada-pande)

---

## 🙏 Acknowledgments

- **FinBERT:** ProsusAI for the domain-specific sentiment model
- **GDELT:** Global Database of Events, Language, and Tone
- **Streamlit:** For rapid dashboard prototyping
- **OpenAI:** For LLM API capabilities

---

## 📞 Questions?

Found a bug or have a suggestion? Open an issue on GitHub or reach out directly.
