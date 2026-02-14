# Kheti Pulse - System Design Document

**Project:** Kheti Pulse (One-Stop Rural Farmer Copilot)  
**Version:** 1.0  
**Date:** 2026-02-14  
**Status:** Design Phase

---

## Architecture Overview

Kheti Pulse is a mobile-first AI copilot for rural farmers, providing weather risk alerts, mandi price intelligence, and conversational agricultural guidance. The system uses a three-tier architecture with a Flutter mobile frontend, FastAPI backend, and RAG-powered knowledge retrieval.

### Key Design Principles

- **Simplicity First:** Prefer proven, simple components over complex ML pipelines
- **Offline Resilience:** Cache critical data for intermittent connectivity scenarios
- **Multilingual Core:** Marathi and Hindi support baked into every layer
- **Responsible AI:** Citations, refusals, and explainability as first-class features
- **Demo-Ready:** All components functional within 7-day timeline

### Technology Stack

| Layer | Technology | Rationale |
|-------|-----------|-----------|
| **Frontend** | Flutter | Cross-platform (Android focus), rich UI, good offline support |
| **Backend** | FastAPI (Python) | Fast development, async support, excellent for ML integration |
| **LLM** | Google Gemini 1.5 Flash | Free tier, strong multilingual (Hindi/Marathi), fast responses |
| **RAG Framework** | LlamaIndex | Simpler than LangChain, good FAISS integration, clear abstractions |
| **Vector Store** | FAISS (local) | No external dependencies, fast, sufficient for <1000 documents |
| **Translation** | Google Translate API | Reliable fallback; IndicTrans2 for production |
| **Weather API** | OpenWeatherMap | Free tier, reliable, 5-day forecast |
| **Mandi Data** | AGMARKNET CSV | Public data, cached locally, daily refresh |
| **Voice STT** | Android Speech Recognition | Native, no API costs, works offline |
| **Deployment** | Railway/Render (backend), APK (mobile) | Free tiers, simple deployment |

---

## High-Level System Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        MOBILE APP (Flutter)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │   Home       │  │  Sell Smart  │  │   Ask Assistant        │ │
│  │  Dashboard   │  │  (Prices)    │  │   (RAG Chat)           │ │
│  └──────┬───────┘  └──────┬───────┘  └──────┬─────────────────┘ │
│         │                 │                  │                    │
│         └─────────────────┴──────────────────┘                    │
│                           │                                       │
│                    ┌──────▼──────┐                                │
│                    │  API Client │                                │
│                    │  (HTTP/REST)│                                │
│                    └──────┬──────┘                                │
└───────────────────────────┼────────────────────────────────────────┘
                            │ HTTPS
                            │
┌───────────────────────────▼────────────────────────────────────────┐
│                    BACKEND (FastAPI)                               │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐  │
│  │  Weather     │  │  Mandi Price │  │   RAG Assistant        │  │
│  │  Service     │  │  Service     │  │   Service              │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬─────────────────┘  │
│         │                 │                  │                     │
│         │                 │          ┌───────▼────────┐            │
│         │                 │          │  LlamaIndex    │            │
│         │                 │          │  Query Engine  │            │
│         │                 │          └───────┬────────┘            │
│         │                 │                  │                     │
└─────────┼─────────────────┼──────────────────┼─────────────────────┘
          │                 │                  │
          │                 │          ┌───────▼────────┐
          │                 │          │  FAISS Vector  │
          │                 │          │  Store (local) │
          │                 │          └────────────────┘
          │                 │
  ┌───────▼────────┐ ┌──────▼──────┐
  │ OpenWeatherMap │ │  AGMARKNET  │
  │      API       │ │  CSV Cache  │
  └────────────────┘ └─────────────┘

External Services:
  - Google Gemini API (LLM)
  - Google Translate API (fallback translation)
```


---

## Component Design

### 1. Mobile App (Flutter)

**Responsibilities:**
- User interface and interaction
- Local caching (SharedPreferences, SQLite)
- Voice input capture (Android Speech Recognition)
- Language preference management
- Offline-first data display

**Key Modules:**
- **Home Dashboard:** Weather cards, action cards (do/don't), risk alerts
- **Sell Smart:** Mandi price list, trend charts, best mandi recommendation
- **Ask Assistant:** Chat interface, voice input, citation display
- **Settings:** Language toggle, location selection, about/disclaimer

**State Management:** Provider pattern (simple, sufficient for MVP)

**Why Flutter:**
- Single codebase for Android (iOS future)
- Rich widget library for charts, cards
- Good offline support with local storage
- Fast development cycle

---

### 2. Backend API (FastAPI)

**Responsibilities:**
- Orchestrate external API calls (weather, LLM)
- Serve mandi price data from local cache
- Handle RAG query pipeline (retrieval + generation)
- Apply responsible AI guardrails
- Log requests for analytics

**Endpoints:** (See APIs section below)

**Why FastAPI:**
- Async/await for concurrent API calls
- Auto-generated OpenAPI docs
- Easy integration with Python ML libraries
- Fast enough for demo scale (100 concurrent users)


---

### 3. Weather Service

**Responsibilities:**
- Fetch 5-day forecast from OpenWeatherMap
- Generate risk alerts based on thresholds
- Create action cards (do/don't) based on weather + crop stage

**Data Flow:**
1. Receive location (lat/lon or district name)
2. Call OpenWeatherMap API (cached for 6 hours)
3. Parse forecast: temp, precipitation, wind, humidity
4. Apply risk rules (see Weather Risk Logic section)
5. Return structured response with alerts + action cards

**Caching Strategy:** Redis or in-memory cache (6-hour TTL)

---

### 4. Mandi Price Service

**Responsibilities:**
- Serve current and historical mandi prices
- Calculate price trends (7-day moving average)
- Recommend best mandi based on net estimate
- Handle missing data gracefully

**Data Source:** AGMARKNET CSV downloaded daily, stored in SQLite

**Data Flow:**
1. Receive crop type, quantity, user location
2. Query SQLite for prices (today + last 7 days)
3. Calculate transport cost (distance × ₹10/km × quantity/100)
4. Compute net estimate for each mandi
5. Rank mandis, return top 5 with recommendation

**Why SQLite:** Simple, no external DB needed, sufficient for <10K rows


---

### 5. RAG Assistant Service

**Responsibilities:**
- Accept user query in Marathi/Hindi
- Retrieve relevant documents from vector store
- Generate answer using LLM with retrieved context
- Apply refusal logic (low confidence, out-of-scope)
- Return answer with citations

**Components:**
- **Query Preprocessor:** Language detection, query cleaning
- **Retriever:** FAISS semantic search (top-k=3)
- **Generator:** Gemini API with prompt template
- **Citation Extractor:** Parse source metadata from retrieved docs
- **Guardrail Filter:** Check for refusal conditions

**Why LlamaIndex:**
- Clean abstractions for RAG pipeline
- Built-in FAISS integration
- Easy prompt customization
- Good documentation

---

## Data Flow

### Flow 1: Home Dashboard Load

```
User opens app
    │
    ▼
[Mobile] Check cache (last updated < 6 hours?)
    │
    ├─ Yes: Display cached data
    │
    └─ No: Call /api/dashboard
            │
            ▼
        [Backend] Fetch weather (OpenWeatherMap)
            │
            ▼
        [Backend] Apply risk rules
            │
            ▼
        [Backend] Generate action cards
            │
            ▼
        [Backend] Return JSON response
            │
            ▼
        [Mobile] Cache response + display
```


### Flow 2: Sell Smart - Best Mandi Recommendation

```
User selects crop (Cotton) + quantity (10 quintals)
    │
    ▼
[Mobile] Call /api/mandi/recommend
    │
    ▼
[Backend] Query SQLite for today's prices (all mandis)
    │
    ▼
[Backend] For each mandi:
    │       - Get price (modal)
    │       - Calculate distance from user location
    │       - Compute transport cost = distance × ₹10 × (quantity/100)
    │       - Compute net estimate = (price × quantity) - transport_cost
    │
    ▼
[Backend] Rank mandis by net estimate (descending)
    │
    ▼
[Backend] Select top mandi + generate explanation
    │       "Akola Mandi offers ₹6,200/quintal. After transport (₹500),
    │        your net return is ₹61,500 - highest among nearby mandis."
    │
    ▼
[Backend] Return JSON with ranked list + recommendation
    │
    ▼
[Mobile] Display list + highlight best mandi
```


### Flow 3: Ask Assistant (RAG Query)

```
User types/speaks: "कपास में सफेद मक्खी का उपचार क्या है?"
                   (What is the treatment for whitefly in cotton?)
    │
    ▼
[Mobile] Capture voice → Android STT → text
    │
    ▼
[Mobile] Call /api/assistant/query
    │       Body: {"query": "कपास में सफेद मक्खी...", "language": "hi"}
    │
    ▼
[Backend] Detect language (already provided: Hindi)
    │
    ▼
[Backend] Translate query to English (for better retrieval)
    │       → "What is the treatment for whitefly in cotton?"
    │
    ▼
[Backend] Generate embedding (Gemini embedding model)
    │
    ▼
[Backend] FAISS similarity search (top-k=3 documents)
    │       Retrieved:
    │       1. "IPM_Cotton.pdf" (score: 0.89)
    │       2. "Pest_Management_Guide.pdf" (score: 0.76)
    │       3. "Cotton_Cultivation.pdf" (score: 0.68)
    │
    ▼
[Backend] Check confidence: max_score >= 0.6? → Yes
    │
    ▼
[Backend] Build prompt:
    │       Context: [Retrieved document chunks]
    │       Query: [Original Hindi query]
    │       Instructions: Answer in Hindi, cite sources, be concise
    │
    ▼
[Backend] Call Gemini API
    │
    ▼
[Backend] Parse response + extract citations
    │       Answer: "सफेद मक्खी के नियंत्रण के लिए..."
    │       Citations: ["IPM_Cotton.pdf, Section 4.2"]
    │
    ▼
[Backend] Return JSON response
    │
    ▼
[Mobile] Display answer + citations
```


---

## RAG Design

### Knowledge Base

**Content Sources:**
- ICAR agricultural extension documents (public PDFs)
- Maharashtra Agriculture Department FAQs
- Krishi Vigyan Kendra (KVK) training materials
- Synthetic documents for demo (if public sources insufficient)

**Coverage:**
- Cotton cultivation (varieties, sowing, spacing, irrigation)
- Soybean crop management (fertilizer, pest control)
- Integrated Pest Management (IPM) for bollworm, whitefly, pod borer
- Soil health for black cotton soil (Vidarbha)
- Water conservation (drip irrigation, mulching)
- Post-harvest handling

**Document Count:** 30-50 curated documents (~10-15 MB total)

**Format:** PDF, TXT, Markdown

---

### Chunking Strategy

**Approach:** Semantic chunking with overlap

**Parameters:**
- **Chunk Size:** 512 tokens (~400 words)
- **Overlap:** 50 tokens (to preserve context across boundaries)
- **Splitting Logic:** Prefer paragraph boundaries over mid-sentence splits

**Why 512 tokens:**
- Balances context richness vs retrieval precision
- Fits within Gemini's context window comfortably
- Reduces noise from overly large chunks

**Implementation:** LlamaIndex `SentenceSplitter` with custom separators


---

### Embeddings

**Model:** Google Gemini Embedding Model (`text-embedding-004`)

**Why Gemini Embeddings:**
- Multilingual support (Hindi, Marathi, English)
- Free tier available
- Consistent with LLM choice (same provider)
- Good performance on semantic similarity tasks

**Dimension:** 768 (standard for Gemini embeddings)

**Embedding Process:**
1. Translate non-English chunks to English (for better embedding quality)
2. Generate embedding via Gemini API
3. Store in FAISS index with metadata (source doc, page, language)

**Fallback:** If Gemini unavailable, use `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` (local)

---

### Retrieval

**Vector Store:** FAISS (Facebook AI Similarity Search)

**Index Type:** `IndexFlatL2` (exact search, sufficient for <1000 documents)

**Retrieval Parameters:**
- **Top-k:** 3 documents
- **Similarity Metric:** L2 distance (Euclidean)
- **Threshold:** Minimum score 0.6 (below this, trigger refusal)

**Query Process:**
1. User query → Translate to English (if needed)
2. Generate query embedding (Gemini)
3. FAISS similarity search → top-3 chunks
4. Return chunks with metadata (source, score)

**Why FAISS:**
- Fast, local, no external dependencies
- Sufficient for demo scale (<1000 docs)
- Easy integration with LlamaIndex
- No cost


---

### Citations

**Approach:** Metadata-based citation extraction

**Citation Format:**
```
"Source: [Document Name], Section [Section Number/Page]"
Example: "Source: IPM Cotton Guide, Section 4.2"
```

**Implementation:**
1. Store metadata with each chunk: `{doc_name, page, section, language}`
2. After retrieval, extract metadata from top-k chunks
3. Include in LLM prompt: "Cite your sources using the format: Source: [doc_name], [section]"
4. Parse LLM response to extract citations
5. Display citations below answer in mobile app

**Fallback:** If LLM doesn't generate citations, append retrieved doc names automatically

---

## Multilingual Design

### Translation Strategy

**Primary Languages:** Marathi (mr), Hindi (hi)

**Fallback:** English (en)

**Translation Points:**
1. **UI Strings:** Hardcoded translations (JSON files)
2. **User Queries:** Translate to English for retrieval (if needed)
3. **LLM Responses:** Generate directly in target language (Gemini supports Hindi/Marathi)
4. **Knowledge Base:** Store in English, translate chunks on-demand

**Translation Service:**
- **Primary:** Google Translate API (free tier: 500K chars/month)
- **Future:** IndicTrans2 (open-source, better for Indic languages)

**Why Google Translate for MVP:**
- Reliable, fast
- Free tier sufficient for demo
- Easy integration
- IndicTrans2 requires model hosting (adds complexity)


---

### Voice Input (STT)

**Technology:** Android Speech Recognition API (native)

**Why Native:**
- No API costs
- Works offline (with downloaded language packs)
- Good accuracy for Hindi/Marathi
- Simple integration with Flutter (`speech_to_text` package)

**Flow:**
1. User taps microphone icon
2. Android STT captures audio
3. Convert to text (Hindi/Marathi)
4. Display text in chat input
5. User confirms/edits → send query

**Limitations:** Requires Android 5.0+, language pack download

---

### Voice Output (TTS) - Stretch Goal

**Technology:** Android Text-to-Speech API (native)

**Languages:** Hindi, Marathi

**Flow:**
1. LLM response received
2. User taps "Listen" button
3. Android TTS reads response aloud

**Priority:** P2 (nice-to-have, not critical for demo)

---

## Weather Risk & Action Cards Logic

### Risk Rules

**Data Inputs:**
- Temperature (°C)
- Precipitation probability (%)
- Wind speed (km/h)
- Humidity (%)
- Crop stage (user-provided or inferred from calendar)


**Risk Thresholds:**

| Condition | Threshold | Risk Level | Alert Message |
|-----------|-----------|------------|---------------|
| Heavy Rain | Precipitation > 70% | High | "Heavy rain expected - postpone spraying" |
| Heatwave | Temp > 40°C | Medium | "Extreme heat - increase irrigation" |
| High Wind | Wind > 30 km/h | Medium | "Strong winds - avoid pesticide application" |
| Frost Risk | Temp < 10°C | High | "Cold weather - protect young plants" |
| High Humidity | Humidity > 85% | Low | "High humidity - monitor for fungal diseases" |

**Action Card Generation:**

**Rule-Based Logic:**
```python
if precipitation > 70%:
    do_cards.append("Check drainage systems")
    dont_cards.append("Don't apply fertilizer or pesticides")
    
if temp > 40°C:
    do_cards.append("Irrigate crops in early morning")
    dont_cards.append("Don't work in fields during midday")
    
if wind > 30 km/h:
    do_cards.append("Secure loose equipment")
    dont_cards.append("Don't spray pesticides")
    
if humidity > 85% and crop_stage == "flowering":
    do_cards.append("Monitor for fungal diseases")
    dont_cards.append("Don't irrigate excessively")
```

**Explainability:**
- Each action card includes a "Why?" tooltip
- Example: "Why avoid spraying? High winds cause pesticide drift, reducing effectiveness and harming nearby crops."

