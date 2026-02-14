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
| **Frontend** | Flutter (Mobile/Web) | Cross-platform (Android focus), rich UI, good offline support |
| **API Gateway** | Amazon API Gateway | Managed, scalable, integrates with Lambda |
| **Compute** | AWS Lambda / ECS | Serverless for API logic, auto-scaling |
| **Database** | Amazon DynamoDB | NoSQL, serverless, fast for user profiles/history |
| **Storage** | Amazon S3 | Object storage for cached datasets, documents |
| **LLM** | Amazon Bedrock (Claude 3 Haiku) | Cost-effective, strong multilingual, fast responses |
| **RAG** | Amazon Bedrock Knowledge Bases | Managed RAG with built-in vector store (OpenSearch Serverless) |
| **Embeddings** | Amazon Bedrock (Titan Embeddings) | Integrated with Knowledge Bases |
| **Guardrails** | Amazon Bedrock Guardrails | Built-in content filtering, refusal policies |
| **Translation** | Amazon Translate | Native AWS service, multilingual support |
| **Voice STT/TTS** | Amazon Transcribe / Amazon Polly | Native AWS services for speech |
| **Weather API** | OpenWeatherMap | Free tier, reliable, 5-day forecast |
| **Mandi Data** | AGMARKNET CSV | Public data, cached in S3, daily refresh |
| **Scheduler** | Amazon EventBridge | Cron jobs for data ingestion |
| **Deployment** | AWS (Lambda/ECS), APK (mobile) | Fully managed AWS infrastructure |

---

## High-Level System Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│              CLIENT / EXPERIENCE LAYER                          │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Mobile App / Web App (Flutter)                          │  │
│  │  • Today Cards  • Ask KhetiPulse  • Sell Smart          │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Multilingual + Voice (optional)                         │  │
│  │  • Amazon Translate (text)                               │  │
│  │  • Amazon Transcribe (STT) • Amazon Polly (TTS)         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  User Context                                            │  │
│  │  • Language • Location • Crop • Sowing date             │  │
│  │  • Saved advisories / preferences                        │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS
                             │
┌────────────────────────────▼────────────────────────────────────┐
│              AWS BACKEND + DATA LAYER                           │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  API Layer                                               │  │
│  │  Amazon API Gateway → AWS Lambda / ECS                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Operational Data Store                                  │  │
│  │  • Amazon DynamoDB (profiles, crops, history)           │  │
│  │  • Amazon S3 (cached datasets, documents)               │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  External Data Ingestion + Caching                       │  │
│  │  • Weather API (forecast, alerts)                        │  │
│  │  • AGMARKNET (mandi prices) → periodic cache to S3      │  │
│  │  • Govt advisories/schemes → curated docs store         │  │
│  │  • Scheduler: EventBridge / cron                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Recommendation Logic (Deterministic)                    │  │
│  │  • Spray window rules (wind/rain thresholds)            │  │
│  │  • Best mandi = price – distance proxy                  │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │
┌────────────────────────────▼────────────────────────────────────┐
│              AI LAYER ON AMAZON BEDROCK                         │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Amazon Bedrock (LLM - Claude 3 Haiku)                   │  │
│  │  • Generate: action checklist + Do/Don't cards          │  │
│  │  • Structured output for Cards + Chat                   │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Bedrock Knowledge Bases (RAG)                           │  │
│  │  • Retrieve from curated docs + cached datasets         │  │
│  │  • Embeddings + Vector store (OpenSearch Serverless)    │  │
│  │  • Return citations/snippets for "proof"                │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Bedrock Guardrails + Safety                             │  │
│  │  • Refuse unsafe requests (e.g., pesticide dosage)      │  │
│  │  • Policy checks + "I don't know" fallback              │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Confidence + Explainability                             │  │
│  │  • Confidence score (retrieval strength)                │  │
│  │  • Show sources/citations in UI                         │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```


---

## Component Design

### 1. Mobile App (Flutter)

**Responsibilities:**
- User interface and interaction
- Local caching (SharedPreferences, SQLite)
- Voice input/output via AWS services (Transcribe/Polly)
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

### 2. API Layer (AWS Lambda / ECS)

**Responsibilities:**
- Handle HTTP requests via API Gateway
- Orchestrate calls to data stores, external APIs, and Bedrock
- Apply business logic and validation
- Return structured JSON responses

**Compute Options:**
- **AWS Lambda:** Serverless, auto-scaling, pay-per-request (preferred for MVP)
- **ECS/Fargate:** Container-based, for more complex workloads (future)

**Why Lambda:**
- No server management
- Auto-scales with demand
- Cost-effective for demo (free tier: 1M requests/month)
- Easy integration with API Gateway and other AWS services

---

### 3. Operational Data Store

**Amazon DynamoDB:**
- User profiles (language, location, crop preferences)
- Query history (for analytics)
- Saved advisories and recommendations
- Crop calendar data (sowing dates, stages)

**Amazon S3:**
- Cached weather data (JSON files)
- Mandi price datasets (CSV files from AGMARKNET)
- Curated agricultural documents (PDFs for Knowledge Base)
- Static assets (images, icons)

**Why DynamoDB:**
- Serverless, no capacity planning
- Fast key-value lookups
- Pay-per-request pricing
- Good for user profiles and history

**Why S3:**
- Cheap object storage
- Integrates with Bedrock Knowledge Bases
- Versioning and lifecycle policies
- Easy to cache external data

---

### 4. External Data Ingestion + Caching

**Responsibilities:**
- Fetch weather forecasts from OpenWeatherMap
- Download mandi prices from AGMARKNET API
- Curate government advisories and schemes
- Schedule periodic updates

**Scheduler:** Amazon EventBridge (cron expressions)

**Data Flow:**
1. EventBridge triggers Lambda function (daily at 11 AM IST)
2. Lambda fetches data from external APIs
3. Parse and validate data
4. Store in S3 (weather, mandi prices) and DynamoDB (metadata)
5. Update Bedrock Knowledge Base (if new documents added)

**Why EventBridge:**
- Managed cron service
- Reliable scheduling
- Easy Lambda integration

---

### 5. Recommendation Logic (Deterministic)

**Responsibilities:**
- Apply rule-based logic for weather-based recommendations
- Calculate best mandi based on price and distance
- Generate action cards (do/don't) based on thresholds

**Implementation:** Python functions in Lambda

**Why Deterministic:**
- Transparent and explainable
- No ML training needed
- Fast and reliable
- Easy to debug and adjust rules

---

### 6. Amazon Bedrock (LLM)

**Model:** Claude 3 Haiku

**Responsibilities:**
- Generate natural language responses for chat
- Create structured action checklists
- Format do/don't cards from weather data
- Respond in Hindi/Marathi based on user preference

**Why Claude 3 Haiku:**
- Fast inference (low latency)
- Cost-effective ($0.25 per 1M input tokens)
- Strong multilingual support
- Good at structured output (JSON mode)

---

### 7. Bedrock Knowledge Bases (RAG)

**Responsibilities:**
- Store and index agricultural documents
- Perform semantic search on user queries
- Return relevant document chunks with citations
- Provide context to LLM for grounded responses

**Components:**
- **Data Source:** S3 bucket with curated PDFs/documents
- **Embeddings:** Amazon Titan Embeddings (automatic)
- **Vector Store:** Amazon OpenSearch Serverless (managed)
- **Retrieval:** Top-k semantic search

**Why Bedrock Knowledge Bases:**
- Fully managed RAG solution
- No need to manage FAISS or vector DB
- Automatic chunking and embedding
- Built-in citation extraction
- Integrates seamlessly with Bedrock LLM

---

### 8. Bedrock Guardrails + Safety

**Responsibilities:**
- Filter harmful or unsafe requests
- Refuse medical, legal, financial advice
- Apply content policies (no hate speech, violence)
- Implement "I don't know" fallback for low confidence

**Configuration:**
- Define denied topics (medical dosage, legal advice)
- Set content filters (hate, violence, sexual)
- Configure PII redaction (if needed)

**Why Bedrock Guardrails:**
- Built-in safety layer
- No custom code needed
- Configurable policies
- Audit logs for compliance

---

### 9. Confidence + Explainability

**Responsibilities:**
- Calculate confidence score based on retrieval quality
- Display citations and sources in UI
- Provide explanations for recommendations
- Show "why" tooltips for action cards

**Implementation:**
- Bedrock Knowledge Bases returns relevance scores
- Lambda checks score threshold (>0.6 = high confidence)
- Include citations in API response
- Frontend displays sources prominently

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
    └─ No: Call API Gateway → /dashboard
            │
            ▼
        [Lambda] Fetch weather from S3 cache (or OpenWeatherMap)
            │
            ▼
        [Lambda] Apply deterministic risk rules
            │
            ▼
        [Lambda] Call Bedrock LLM to generate action cards
            │       Prompt: "Given weather: {data}, generate do/don't cards"
            │
            ▼
        [Lambda] Return JSON response
            │
            ▼
        [Mobile] Cache response + display
```

---

### Flow 2: Sell Smart - Best Mandi Recommendation

```
User selects crop (Cotton) + quantity (10 quintals)
    │
    ▼
[Mobile] Call API Gateway → /mandi/recommend
    │
    ▼
[Lambda] Query DynamoDB for today's prices (all mandis)
    │    (or read from S3 cache if DynamoDB not used)
    │
    ▼
[Lambda] For each mandi:
    │       - Get price (modal)
    │       - Calculate distance from user location
    │       - Compute transport cost = distance × ₹10 × (quantity/100)
    │       - Compute net estimate = (price × quantity) - transport_cost
    │
    ▼
[Lambda] Rank mandis by net estimate (descending)
    │
    ▼
[Lambda] Select top mandi + generate explanation
    │       "Akola Mandi offers ₹6,200/quintal. After transport (₹500),
    │        your net return is ₹61,500 - highest among nearby mandis."
    │
    ▼
[Lambda] Return JSON with ranked list + recommendation
    │
    ▼
[Mobile] Display list + highlight best mandi
```

---

### Flow 3: Ask Assistant (RAG Query with Bedrock Knowledge Bases)

```
User types/speaks: "कपास में सफेद मक्खी का उपचार क्या है?"
                   (What is the treatment for whitefly in cotton?)
    │
    ▼
[Mobile] Capture voice → Amazon Transcribe → text (optional)
    │
    ▼
[Mobile] Call API Gateway → /assistant/query
    │       Body: {"query": "कपास में सफेद मक्खी...", "language": "hi"}
    │
    ▼
[Lambda] Detect language (already provided: Hindi)
    │
    ▼
[Lambda] Optional: Translate query to English via Amazon Translate
    │       (for better retrieval if needed)
    │
    ▼
[Lambda] Call Bedrock Knowledge Bases RetrieveAndGenerate API
    │       - Knowledge Base ID: kb-xxxxx
    │       - Query: user question
    │       - Model: Claude 3 Haiku
    │       - Language: Hindi
    │
    ▼
[Bedrock KB] Automatic workflow:
    │       1. Generate embedding (Titan Embeddings)
    │       2. Search OpenSearch vector store (top-k=3)
    │       3. Retrieve document chunks with citations
    │       4. Build prompt with context
    │       5. Call Claude 3 Haiku LLM
    │       6. Apply Guardrails (safety checks)
    │       7. Return answer + citations + confidence
    │
    ▼
[Lambda] Parse Bedrock response
    │       Answer: "सफेद मक्खी के नियंत्रण के लिए..."
    │       Citations: [{source: "IPM_Cotton.pdf", page: 42}]
    │       Confidence: 0.89 (high)
    │
    ▼
[Lambda] Check confidence threshold
    │       If < 0.6: Return refusal message
    │       If >= 0.6: Return answer with citations
    │
    ▼
[Lambda] Return JSON response
    │
    ▼
[Mobile] Display answer + citations
    │       Optional: Amazon Polly TTS for voice output
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

**Storage:** Amazon S3 bucket (configured as Bedrock Knowledge Base data source)

---

### Chunking Strategy

**Approach:** Automatic chunking by Bedrock Knowledge Bases

**Parameters (configurable):**
- **Chunk Size:** Default (300 tokens) or custom (512 tokens)
- **Overlap:** Default (20%) or custom (50 tokens)
- **Splitting Logic:** Semantic boundaries (paragraphs, sections)

**Why Automatic:**
- No manual chunking code needed
- Bedrock optimizes for retrieval quality
- Consistent with embedding model

---

### Embeddings

**Model:** Amazon Titan Embeddings G1 - Text (`amazon.titan-embed-text-v1`)

**Why Titan Embeddings:**
- Fully managed by Bedrock Knowledge Bases
- Multilingual support (100+ languages including Hindi, Marathi)
- Cost-effective ($0.0001 per 1K tokens)
- Optimized for semantic search
- No manual embedding generation needed

**Dimension:** 1536

**Embedding Process:**
- Automatic: Bedrock Knowledge Bases handles embedding generation
- Documents uploaded to S3 → Bedrock syncs → generates embeddings → stores in OpenSearch

---

### Retrieval

**Vector Store:** Amazon OpenSearch Serverless (managed by Bedrock)

**Retrieval Parameters:**
- **Top-k:** 3-5 documents (configurable)
- **Similarity Metric:** Cosine similarity
- **Threshold:** Minimum relevance score 0.6

**Query Process:**
1. User query → Bedrock Knowledge Bases API
2. Automatic embedding generation (Titan)
3. OpenSearch vector search → top-k chunks
4. Return chunks with metadata (source, page, score)

**Why Bedrock Knowledge Bases:**
- Fully managed (no OpenSearch cluster management)
- Automatic sync from S3
- Built-in citation extraction
- Integrated with Bedrock LLM
- No infrastructure management

---

### Citations

**Approach:** Automatic citation extraction by Bedrock Knowledge Bases

**Citation Format:**
```json
{
  "retrievedReferences": [
    {
      "content": { "text": "..." },
      "location": {
        "type": "S3",
        "s3Location": { "uri": "s3://bucket/IPM_Cotton.pdf" }
      },
      "metadata": { "page": "42", "section": "4.2" }
    }
  ]
}
```

**Display Format:**
```
"Source: IPM Cotton Guide, Page 42, Section 4.2"
```

**Implementation:**
- Bedrock returns citations automatically
- Lambda parses and formats for mobile app
- Frontend displays citations below answer

---

## Multilingual Design

### Translation Strategy

**Primary Languages:** Marathi (mr), Hindi (hi)

**Fallback:** English (en)

**Translation Points:**
1. **UI Strings:** Hardcoded translations (JSON files in Flutter)
2. **User Queries:** Optional translation to English for better retrieval
3. **LLM Responses:** Generate directly in target language (Claude supports Hindi/Marathi)
4. **Knowledge Base:** Store in English, retrieve and generate in user language

**Translation Service:** Amazon Translate

**Why Amazon Translate:**
- Native AWS service (easy integration)
- Supports 75+ languages including Hindi, Marathi
- Pay-per-character pricing ($15 per 1M characters)
- Real-time translation API
- Good quality for Indic languages

**LLM Multilingual Support:**
- Claude 3 Haiku supports Hindi and Marathi natively
- Generate responses directly in target language
- Better quality than translate-after-generation
- Prompt: "Answer in Hindi: {query}"

---

### Voice Input (STT)

**Technology:** Amazon Transcribe

**Languages:** Hindi, Marathi, English

**Why Amazon Transcribe:**
- Native AWS service
- Good accuracy for Indic languages
- Real-time streaming or batch processing
- Pay-per-second pricing ($0.024 per minute)
- Easy integration with Lambda

**Flow:**
1. User taps microphone icon in mobile app
2. Record audio (Flutter audio recorder)
3. Upload audio to S3 or stream to Transcribe
4. Transcribe converts to text (Hindi/Marathi)
5. Display text in chat input
6. User confirms/edits → send query

**Alternative:** Android native Speech Recognition (offline, free) for MVP

---

### Voice Output (TTS) - Optional

**Technology:** Amazon Polly

**Languages:** Hindi (Aditi voice), English (Indian accent)

**Note:** Marathi not directly supported; use Hindi voice or skip TTS for Marathi

**Flow:**
1. LLM response received
2. User taps "Listen" button
3. Lambda calls Polly API
4. Polly generates audio (MP3)
5. Stream audio to mobile app
6. Play audio

**Priority:** P2 (nice-to-have, not critical for demo)

**Cost:** $4 per 1M characters

**
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
- **Generator:** Amazon Bedrock (Claude 3 Haiku) with prompt template
- **Citation Extractor:** Parse source metadata from retrieved docs
- **Guardrail Filter:** Check for refusal conditions

**Why LlamaIndex:**
- Clean abstractions for RAG pipeline
- Built-in FAISS integration
- Easy prompt customization
- Good documentation
- Supports Amazon Bedrock out of the box

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
[Backend] Generate embedding (Amazon Titan Embeddings)
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
[Backend] Call Amazon Bedrock (Claude 3 Haiku)
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

**Model:** Amazon Bedrock Titan Embeddings (`amazon.titan-embed-text-v1`)

**Why Titan Embeddings:**
- Multilingual support (100+ languages including Hindi, Marathi)
- Cost-effective ($0.0001 per 1K tokens)
- Consistent with LLM choice (same provider)
- Good performance on semantic similarity tasks
- Enterprise-grade reliability

**Dimension:** 1536 (standard for Titan Embeddings)

**Embedding Process:**
1. Translate non-English chunks to English (for better embedding quality)
2. Generate embedding via Bedrock API
3. Store in FAISS index with metadata (source doc, page, language)

**Fallback:** If Bedrock unavailable, use `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2` (local)

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
2. Generate query embedding (Amazon Titan Embeddings)
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

**LLM Multilingual Support:**
- Amazon Bedrock Claude 3 Haiku supports Hindi and Marathi natively
- Generate responses directly in target language without translation
- Better quality than translate-after-generation approach


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


---

## Mandi Price Intelligence Design

### Data Ingestion

**Source:** AGMARKNET API (https://agmarknet.gov.in)

**Endpoint:** `/PriceAndArrival/DateWisePriceAndArrivalReport`

**Parameters:**
- State: Maharashtra
- District: Nagpur, Akola, Amravati, Yavatmal, Wardha
- Commodity: Cotton, Soybean
- Date: Today

**Ingestion Schedule:** Daily at 11 AM IST (after mandi data updates)

**Process:**
1. Cron job triggers daily
2. Fetch CSV from AGMARKNET
3. Parse and validate data
4. Insert into SQLite (upsert on date + mandi + crop)
5. Retain 30 days of history (rolling window)

**Fallback:** If API fails, use cached data + display "Last updated: [timestamp]"

---

### Storage Schema

**Table: `mandi_prices`**

```sql
CREATE TABLE mandi_prices (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    date DATE NOT NULL,
    mandi_name VARCHAR(100) NOT NULL,
    district VARCHAR(50) NOT NULL,
    crop_name VARCHAR(50) NOT NULL,
    variety VARCHAR(50),
    min_price DECIMAL(10,2),
    max_price DECIMAL(10,2),
    modal_price DECIMAL(10,2) NOT NULL,
    arrival_quantity DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(date, mandi_name, crop_name, variety)
);

CREATE INDEX idx_date_crop ON mandi_prices(date, crop_name);
CREATE INDEX idx_mandi ON mandi_prices(mandi_name);
```


---

### Forecasting Approach

**Method:** Simple 7-day moving average (no complex ML)

**Why Simple:**
- Agricultural prices are volatile and hard to predict
- Complex models require more data and tuning
- Moving average sufficient for "trend" indicator (up/down/stable)
- Transparent and explainable

**Calculation:**
```python
def calculate_trend(prices_last_7_days):
    if len(prices_last_7_days) < 3:
        return "insufficient_data"
    
    avg_first_3 = mean(prices_last_7_days[:3])
    avg_last_3 = mean(prices_last_7_days[-3:])
    
    change_pct = ((avg_last_3 - avg_first_3) / avg_first_3) * 100
    
    if change_pct > 5:
        return "rising"
    elif change_pct < -5:
        return "falling"
    else:
        return "stable"
```

**Display:**
- Rising: ↑ Green arrow + "Prices rising (+8%)"
- Falling: ↓ Red arrow + "Prices falling (-6%)"
- Stable: → Gray arrow + "Prices stable"

---

### Best Mandi Scoring

**Scoring Formula:**
```
Net Estimate = (Modal Price × Quantity) - Transport Cost

Transport Cost = Distance (km) × ₹10/km × (Quantity in quintals / 100)
```

**Distance Calculation:**
- Use static lookup table (district-to-mandi distances)
- Example: Nagpur to Akola = 250 km

**Ranking:**
1. Calculate net estimate for all mandis
2. Sort by net estimate (descending)
3. Select top mandi
4. Generate explanation

**Explanation Template:**
```
"{Mandi Name} offers ₹{price}/quintal. After transport costs (₹{transport_cost}), 
your net return is ₹{net_estimate} - the highest among nearby mandis."
```


---

## APIs

### Base URL
```
Production: https://kheti-pulse-api.railway.app
Development: http://localhost:8000
```

### Authentication
**MVP:** No authentication (public endpoints)  
**Future:** API key or JWT for rate limiting

---

### Endpoint 1: Get Dashboard Data

**Endpoint:** `GET /api/dashboard`

**Query Parameters:**
- `lat` (float, required): Latitude
- `lon` (float, required): Longitude
- `language` (string, optional): `hi` or `mr` (default: `hi`)

**Request Example:**
```http
GET /api/dashboard?lat=21.1458&lon=79.0882&language=mr
```

**Response Example:**
```json
{
  "date": "2026-02-14",
  "location": "Nagpur, Maharashtra",
  "weather": {
    "temp": 28,
    "condition": "Partly Cloudy",
    "precipitation_prob": 20,
    "humidity": 65,
    "wind_speed": 12
  },
  "risk_alerts": [
    {
      "level": "medium",
      "message": "उच्च आर्द्रता - बुरशीजन्य रोगांवर लक्ष ठेवा",
      "message_en": "High humidity - monitor for fungal diseases"
    }
  ],
  "action_cards": {
    "do": [
      "सकाळी लवकर सिंचन करा",
      "जल निचरा तपासा"
    ],
    "dont": [
      "मध्यान्हात शेतात काम करू नका",
      "आज फवारणी टाळा"
    ]
  }
}
```


---

### Endpoint 2: Get Mandi Prices

**Endpoint:** `GET /api/mandi/prices`

**Query Parameters:**
- `crop` (string, required): `cotton` or `soybean`
- `district` (string, optional): Filter by district
- `date` (string, optional): Date in YYYY-MM-DD format (default: today)

**Request Example:**
```http
GET /api/mandi/prices?crop=cotton&district=Nagpur
```

**Response Example:**
```json
{
  "crop": "cotton",
  "date": "2026-02-14",
  "prices": [
    {
      "mandi_name": "Akola",
      "district": "Akola",
      "modal_price": 6200,
      "min_price": 6000,
      "max_price": 6400,
      "arrival_quantity": 1500,
      "trend": "rising",
      "trend_pct": 8.2
    },
    {
      "mandi_name": "Nagpur",
      "district": "Nagpur",
      "modal_price": 6100,
      "min_price": 5900,
      "max_price": 6300,
      "arrival_quantity": 2000,
      "trend": "stable",
      "trend_pct": 2.1
    }
  ]
}
```


---

### Endpoint 3: Get Best Mandi Recommendation

**Endpoint:** `POST /api/mandi/recommend`

**Request Body:**
```json
{
  "crop": "cotton",
  "quantity": 10,
  "user_location": {
    "lat": 21.1458,
    "lon": 79.0882,
    "district": "Nagpur"
  },
  "language": "hi"
}
```

**Response Example:**
```json
{
  "crop": "cotton",
  "quantity": 10,
  "recommendations": [
    {
      "rank": 1,
      "mandi_name": "Akola",
      "district": "Akola",
      "modal_price": 6200,
      "distance_km": 250,
      "transport_cost": 2500,
      "gross_revenue": 62000,
      "net_estimate": 59500,
      "explanation": "अकोला मंडी ₹6,200/क्विंटल देते. वाहतूक खर्च (₹2,500) वजा केल्यानंतर, तुमचा निव्वळ परतावा ₹59,500 आहे - जवळच्या मंड्यांमध्ये सर्वाधिक."
    },
    {
      "rank": 2,
      "mandi_name": "Nagpur",
      "district": "Nagpur",
      "modal_price": 6100,
      "distance_km": 0,
      "transport_cost": 0,
      "gross_revenue": 61000,
      "net_estimate": 61000,
      "explanation": "नागपूर मंडी जवळ आहे (वाहतूक खर्च नाही), परंतु किंमत कमी आहे."
    }
  ],
  "best_mandi": "Akola"
}
```


---

### Endpoint 4: Ask Assistant (RAG Query)

**Endpoint:** `POST /api/assistant/query`

**Request Body:**
```json
{
  "query": "कपास में सफेद मक्खी का उपचार क्या है?",
  "language": "hi",
  "session_id": "optional-session-id"
}
```

**Response Example (Success):**
```json
{
  "query": "कपास में सफेद मक्खी का उपचार क्या है?",
  "answer": "सफेद मक्खी के नियंत्रण के लिए निम्नलिखित उपाय करें:\n1. नीम आधारित कीटनाशक का छिड़काव करें (1500 ppm)\n2. पीले चिपचिपे ट्रैप लगाएं\n3. इमिडाक्लोप्रिड 17.8% SL @ 0.5 ml/लीटर का उपयोग करें\n4. प्राकृतिक शत्रुओं को संरक्षित करें",
  "citations": [
    {
      "source": "IPM_Cotton_Guide.pdf",
      "section": "Section 4.2: Whitefly Management",
      "relevance_score": 0.89
    },
    {
      "source": "Pest_Management_Handbook.pdf",
      "section": "Chapter 3: Sucking Pests",
      "relevance_score": 0.76
    }
  ],
  "confidence": "high",
  "language": "hi"
}
```

**Response Example (Low Confidence / Refusal):**
```json
{
  "query": "मुझे कौन सा बीमा लेना चाहिए?",
  "answer": "मैं इस सवाल का जवाब आत्मविश्वास से नहीं दे सकता। कृपया कृषि विशेषज्ञ या बीमा सलाहकार से परामर्श करें।",
  "citations": [],
  "confidence": "low",
  "refusal_reason": "out_of_scope",
  "language": "hi"
}
```


---

## Data Storage

### SQLite Database Schema

**Database File:** `kheti_pulse.db`

**Table 1: `mandi_prices`** (see Mandi Price Intelligence section above)

**Table 2: `weather_cache`**

```sql
CREATE TABLE weather_cache (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    location_key VARCHAR(100) NOT NULL,
    lat DECIMAL(10,6),
    lon DECIMAL(10,6),
    forecast_date DATE NOT NULL,
    temp DECIMAL(5,2),
    condition VARCHAR(50),
    precipitation_prob INTEGER,
    humidity INTEGER,
    wind_speed DECIMAL(5,2),
    fetched_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE(location_key, forecast_date)
);

CREATE INDEX idx_location_date ON weather_cache(location_key, forecast_date);
```

**Table 3: `query_logs`** (for analytics)

```sql
CREATE TABLE query_logs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    query_text TEXT NOT NULL,
    language VARCHAR(10),
    confidence VARCHAR(20),
    refusal_reason VARCHAR(50),
    response_time_ms INTEGER,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_created_at ON query_logs(created_at);
```


---

### File Storage

**Knowledge Base Documents:**
```
/data/knowledge_base/
  ├── cotton/
  │   ├── IPM_Cotton_Guide.pdf
  │   ├── Cotton_Cultivation_Practices.pdf
  │   └── Bollworm_Management.pdf
  ├── soybean/
  │   ├── Soybean_Crop_Management.pdf
  │   └── Pod_Borer_Control.pdf
  ├── soil/
  │   ├── Black_Cotton_Soil_Management.pdf
  │   └── Soil_Health_Card.pdf
  └── general/
      ├── Water_Conservation.pdf
      └── Post_Harvest_Handling.pdf
```

**FAISS Index:**
```
/data/vector_store/
  ├── faiss_index.bin
  └── metadata.json
```

**Mandi CSV Cache:**
```
/data/mandi_cache/
  └── agmarknet_YYYY-MM-DD.csv
```

---

## Observability

### Logging

**Framework:** Python `logging` module + structured logs (JSON)

**Log Levels:**
- **DEBUG:** Detailed flow (retrieval scores, prompt templates)
- **INFO:** API requests, cache hits/misses
- **WARNING:** Low confidence responses, API failures
- **ERROR:** Exceptions, data validation errors

**Log Format:**
```json
{
  "timestamp": "2026-02-14T10:30:45Z",
  "level": "INFO",
  "service": "rag_assistant",
  "message": "Query processed successfully",
  "query": "कपास में सफेद मक्खी...",
  "confidence": "high",
  "response_time_ms": 1250
}
```

**Log Storage:** Local files (rotated daily) + stdout (for cloud logging)


---

### Metrics

**Key Metrics:**

| Metric | Description | Target |
|--------|-------------|--------|
| **API Response Time** | P95 latency for all endpoints | <3s |
| **RAG Query Time** | Time from query to answer | <5s |
| **Cache Hit Rate** | % of requests served from cache | >70% |
| **Refusal Rate** | % of queries refused (low confidence) | <20% |
| **Error Rate** | % of requests resulting in 5xx errors | <1% |
| **Daily Active Users** | Unique users per day | 50+ (demo) |

**Monitoring Tool:** Simple in-memory counters (MVP), Prometheus (future)

**Dashboard:** Basic Flask admin page showing metrics

---

## Security & Privacy

### No Personal Data Collection

**Principle:** Zero PII storage

**Implementation:**
- No user accounts or authentication
- No storage of user names, phone numbers, locations (beyond district-level)
- Query logs anonymized (no user identifiers)
- No tracking cookies or device fingerprinting

**Compliance:** GDPR-friendly by design (no personal data = no compliance burden)

---

### Safe Data Handling

**API Keys:** Store in environment variables (`.env` file, not in code)

**HTTPS Only:** All API communication over TLS

**Input Validation:**
- Sanitize user queries (remove SQL injection attempts, XSS)
- Validate lat/lon ranges, crop names (whitelist)
- Rate limiting: 100 requests/hour per IP (simple in-memory counter)

**Knowledge Base Security:**
- Only curated, public documents
- No user-uploaded content
- Regular review for harmful content


---

## Responsible AI Guardrails

### 1. Refusal Logic

**Trigger Conditions:**

| Condition | Action | Response Template |
|-----------|--------|-------------------|
| Retrieval score < 0.6 | Refuse | "मैं इस सवाल का जवाब आत्मविश्वास से नहीं दे सकता। कृपया कृषि विशेषज्ञ से परामर्श करें।" |
| Medical question detected | Refuse | "मैं चिकित्सा सलाह नहीं दे सकता। कृपया डॉक्टर से संपर्क करें।" |
| Legal question detected | Refuse | "मैं कानूनी सलाह नहीं दे सकता। कृपया वकील से परामर्श करें।" |
| Financial advice requested | Refuse | "मैं वित्तीय सलाह नहीं दे सकता। कृपया वित्तीय सलाहकार से संपर्क करें।" |
| Harmful content detected | Refuse | "मैं इस प्रकार के प्रश्नों का उत्तर नहीं दे सकता।" |

**Detection Method:**
- Keyword matching (simple regex for MVP)
- LLM-based classification (future)

---

### 2. Low-Confidence Behavior

**When confidence is medium (score 0.6-0.75):**
- Provide answer with disclaimer: "यह जानकारी सीमित स्रोतों पर आधारित है। कृपया स्थानीय विशेषज्ञ से सत्यापित करें।"
- Show fewer citations (only top 1)

**When confidence is low (score < 0.6):**
- Refuse to answer
- Suggest alternative: "क्या आप अपना प्रश्न अधिक विशिष्ट तरीके से पूछ सकते हैं?"

---

### 3. Citation Requirement

**Rule:** Every answer must include at least one citation

**Enforcement:**
- Post-processing check: If no citations in LLM response, append retrieved doc names
- Display citations prominently in UI (not hidden in footnotes)

**Example Display:**
```
Answer: [LLM response]

📚 Sources:
• IPM Cotton Guide, Section 4.2
• Pest Management Handbook, Chapter 3
```


---

### 4. Disclaimer Display

**First Launch Disclaimer:**
```
⚠️ महत्वपूर्ण सूचना

यह ऐप केवल निर्णय-समर्थन जानकारी प्रदान करता है। 
यह पेशेवर कृषि सलाह का विकल्प नहीं है।

महत्वपूर्ण निर्णयों को हमेशा स्थानीय विशेषज्ञों से सत्यापित करें।

[समझ गया] [और जानें]
```

**Persistent Disclaimer:** Small text in assistant interface footer

---

## Performance & Caching Strategy

### Caching Layers

**1. Weather Data Cache**
- **TTL:** 6 hours
- **Storage:** SQLite + in-memory (Redis for production)
- **Key:** `weather:{lat}:{lon}:{date}`
- **Invalidation:** Time-based (6 hours)

**2. Mandi Price Cache**
- **TTL:** 24 hours (refreshed daily at 11 AM)
- **Storage:** SQLite
- **Key:** `mandi:{crop}:{date}`
- **Invalidation:** Daily refresh

**3. RAG Retrieval Cache**
- **TTL:** 1 hour
- **Storage:** In-memory (LRU cache, max 1000 entries)
- **Key:** `rag:{query_hash}`
- **Invalidation:** Time-based (1 hour) or knowledge base update

**4. Mobile App Cache**
- **Storage:** SharedPreferences (small data), SQLite (structured data)
- **Cached Data:** Last dashboard response, mandi prices, chat history
- **Invalidation:** Manual refresh or 6-hour timeout


---

### Performance Targets

| Component | Target | Strategy |
|-----------|--------|----------|
| **Dashboard Load** | <3s | Cache weather, precompute action cards |
| **Mandi Price List** | <2s | SQLite query + in-memory cache |
| **RAG Query** | <5s | Parallel retrieval + LLM call, cache frequent queries |
| **Mobile App Launch** | <2s | Lazy load non-critical components |
| **APK Size** | <50 MB | Exclude unused dependencies, compress assets |

**Optimization Techniques:**
- Async API calls (FastAPI async endpoints)
- Connection pooling for external APIs
- Batch embedding generation (if adding multiple docs)
- Lazy loading of FAISS index (load on first query)

---

## Deployment Plan

### Local Development

**Backend:**
```bash
# Setup
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run
uvicorn main:app --reload --port 8000
```

**Mobile:**
```bash
# Setup
flutter pub get

# Run on emulator/device
flutter run
```

**Environment Variables:**
```bash
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1
BEDROCK_MODEL_ID=anthropic.claude-3-haiku-20240307-v1:0
BEDROCK_EMBEDDING_MODEL_ID=amazon.titan-embed-text-v1
OPENWEATHER_API_KEY=your_key_here
GOOGLE_TRANSLATE_API_KEY=your_key_here
DATABASE_URL=sqlite:///kheti_pulse.db
```


---

### Cloud Deployment

**Backend Hosting:** Railway or Render (free tier)

**Deployment Steps:**
1. Push code to GitHub
2. Connect Railway to GitHub repo
3. Set environment variables in Railway dashboard
4. Deploy (automatic on git push)
5. Note deployed URL (e.g., `https://kheti-pulse-api.railway.app`)

**Database:** SQLite file persisted in Railway volume

**Static Files:** Knowledge base PDFs + FAISS index bundled with deployment

**Why Railway:**
- Free tier: 500 hours/month (sufficient for demo)
- Simple deployment (no Docker knowledge needed)
- Automatic HTTPS
- Good for Python apps

**Alternative:** Render (similar features, slightly different pricing)

---

### Mobile App Distribution

**Demo Distribution:**
- Build APK: `flutter build apk --release`
- Share via Google Drive, Dropbox, or direct download link
- No Play Store submission (not needed for hackathon)

**Installation:**
- Enable "Install from Unknown Sources" on Android
- Download APK
- Install

**Future:** Publish to Google Play Store (requires developer account, ₹2000 one-time fee)


---

## Testing Strategy

### Backend Testing

**Unit Tests:**
- Weather risk rule logic
- Mandi price calculations (net estimate, transport cost)
- RAG retrieval (mock FAISS responses)
- Translation functions

**Integration Tests:**
- End-to-end API tests (mock external APIs)
- Database operations (SQLite CRUD)
- LlamaIndex query pipeline

**Tools:** pytest, pytest-asyncio

**Coverage Target:** >70% for core logic

---

### Mobile Testing

**Manual Testing:**
- UI flow testing (home → sell smart → assistant)
- Language switching
- Voice input (on real device)
- Offline behavior (airplane mode)

**Widget Tests:**
- Dashboard card rendering
- Price list display
- Chat message bubbles

**Tools:** Flutter test framework

**Devices:** Test on 2-3 Android devices (different screen sizes)

---

### End-to-End Testing

**Demo Scenarios:**
1. **Morning Check:** Open app → see weather alert → view action cards
2. **Price Discovery:** Navigate to Sell Smart → select Cotton → see prices → get recommendation
3. **Ask Question:** Open assistant → ask "कपास में सफेद मक्खी का उपचार?" → receive answer with citations
4. **Voice Input:** Tap mic → speak in Marathi → see transcription → get answer
5. **Offline Resilience:** Turn off internet → see cached data → turn on → data refreshes


---

### Load Testing

**Tool:** Locust (Python load testing)

**Scenario:** 100 concurrent users making API requests

**Endpoints to Test:**
- `/api/dashboard` (most frequent)
- `/api/mandi/prices`
- `/api/assistant/query` (most expensive)

**Success Criteria:**
- P95 latency < 5s
- Error rate < 1%
- No crashes

---

## Risks & Mitigations

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| **External API downtime** (OpenWeather, Bedrock) | High | Low | Cache data, fallback to synthetic data, display "Last updated" timestamp, use AWS multi-region |
| **LLM hallucination** (incorrect agricultural advice) | High | Medium | RAG with citations, refusal logic, disclaimer, manual review of knowledge base |
| **AWS costs exceed budget** | Medium | Medium | Set billing alerts, use Claude Haiku (cheapest), implement request caching, monitor usage |
| **Poor multilingual quality** (translation errors) | Medium | Medium | Use Google Translate (reliable), manual review of UI strings, test with native speakers |
| **Slow RAG responses** (>5s) | Medium | Low | Cache frequent queries, optimize FAISS index, use faster LLM (Gemini Flash) |
| **Knowledge base gaps** (missing topics) | Medium | High | Curate 30-50 diverse documents, implement "I don't know" responses, collect user feedback |
| **Mandi data unavailable** (AGMARKNET API fails) | Medium | Medium | Cache 30 days of data, fallback to synthetic data, display staleness warning |
| **Voice recognition errors** (STT inaccuracy) | Low | Medium | Allow text editing after STT, support text input as primary method |
| **Demo day technical issues** (internet, device) | High | Low | Pre-load all data, test on multiple devices, have backup device, record demo video |
| **Scope creep** (too many features) | High | High | Strict MVP definition, prioritize P0 features only, defer stretch goals |
| **Team bandwidth** (7-day timeline) | High | Medium | Daily standups, clear task assignments, cut features if needed |


---

## Future Enhancements

### Phase 2 (Post-Hackathon)

1. **User Accounts & Personalization**
   - Save crop preferences, location
   - Personalized action cards based on crop calendar
   - Chat history persistence

2. **Push Notifications**
   - Weather alerts (heavy rain, heatwave)
   - Price spike notifications
   - Government scheme deadlines

3. **Crop Calendar**
   - Stage-based recommendations (sowing, flowering, harvest)
   - Automated reminders for key activities

4. **Government Schemes Explorer**
   - Searchable database of schemes
   - Eligibility checker
   - Document upload for applications

5. **Community Features**
   - Farmer forums (moderated)
   - Success stories
   - Expert Q&A sessions

---

### Phase 3 (Scale)

1. **Expanded Geography**
   - All Indian states
   - Localized content for each region

2. **More Crops**
   - Wheat, rice, sugarcane, vegetables
   - Livestock and fisheries

3. **Advanced Analytics**
   - Predictive price modeling (ML-based)
   - Yield forecasting
   - Pest outbreak predictions

4. **IoT Integration**
   - Soil moisture sensors
   - Weather stations
   - Automated irrigation triggers

5. **Marketplace**
   - Direct buyer-seller connections
   - Quality certification
   - Logistics support


---

## Design Constraints

### Hard Constraints

1. **Public/Synthetic Data Only**
   - No proprietary datasets
   - No user-generated content (for MVP)
   - All knowledge base documents must be publicly available or synthetic

2. **7-Day Timeline**
   - MVP must be demo-ready in 7 days
   - No time for complex ML model training
   - Focus on integration over innovation

3. **Simple, Robust Components**
   - Prefer proven libraries over custom implementations
   - No experimental technologies
   - Prioritize reliability over cutting-edge features

4. **Multilingual Support (Marathi + Hindi)**
   - All features must work in both languages
   - UI strings externalized
   - LLM must support both languages

5. **Free Tier APIs Only**
   - No paid services (budget constraint)
   - Use free tiers: Gemini, OpenWeather, Google Translate
   - Self-hosted components where possible (FAISS, SQLite)

---

### Soft Constraints

1. **Android Focus** (iOS future)
2. **Maharashtra/Vidarbha Region** (expand later)
3. **Cotton & Soybean Only** (add crops later)
4. **No User Authentication** (add in Phase 2)
5. **Basic UI** (Material Design, no custom illustrations)


---

## Technology Choices - Rationale

### Why Flutter?
- **Cross-platform:** Android now, iOS later with same codebase
- **Rich UI:** Material Design widgets, charts, animations
- **Offline support:** Good local storage options (SharedPreferences, SQLite)
- **Fast development:** Hot reload, large package ecosystem
- **Team familiarity:** Assuming team has Flutter experience

### Why FastAPI?
- **Speed:** Fast development, fast runtime (async)
- **Python ecosystem:** Easy integration with ML libraries (LlamaIndex, FAISS)
- **Auto docs:** OpenAPI/Swagger UI out of the box
- **Type safety:** Pydantic models for request/response validation
- **Async support:** Handle concurrent API calls efficiently

### Why Amazon Bedrock?
- **Cost-effective:** Claude 3 Haiku is optimized for speed and cost
- **Multilingual:** Strong Hindi/Marathi support (Claude 3 family)
- **Enterprise-grade:** AWS infrastructure, high reliability
- **Security:** Built-in content filtering, compliance features
- **Embeddings included:** Titan Embeddings in same platform
- **No rate limits:** Pay-as-you-go, no free tier restrictions
- **Responsible AI:** Built-in guardrails and safety features

**Pricing (Approximate):**
- Claude 3 Haiku: $0.25 per 1M input tokens, $1.25 per 1M output tokens
- Titan Embeddings: $0.0001 per 1K tokens
- Estimated demo cost: ~$5-10 for 1000 queries (very affordable)

**Setup Requirements:**
- AWS account with Bedrock access enabled
- IAM user with Bedrock permissions
- Model access request (Claude 3 Haiku, Titan Embeddings) - usually instant approval

### Why LlamaIndex?
- **Simplicity:** Cleaner API than LangChain
- **FAISS integration:** Built-in support
- **Good docs:** Easy to get started
- **Flexibility:** Easy to customize retrieval pipeline

### Why FAISS?
- **Local:** No external dependencies, no cost
- **Fast:** Optimized for similarity search
- **Sufficient:** Handles <1000 documents easily
- **No setup:** Just load index file and query

### Why SQLite?
- **Simple:** No server setup, just a file
- **Sufficient:** Handles <10K rows easily
- **Portable:** Easy to backup, migrate
- **No cost:** Built into Python

### Why OpenWeatherMap?
- **Free tier:** 1000 calls/day (sufficient for demo)
- **Reliable:** Industry standard
- **Good coverage:** India well-supported
- **Simple API:** Easy integration


---

## Appendix: ASCII Diagrams

### Sequence Diagram: RAG Query Flow

```
User          Mobile App       Backend API      LlamaIndex      FAISS       Bedrock API
 |                |                 |                |             |             |
 |--speak query-->|                 |                |             |             |
 |                |--STT----------->|                |             |             |
 |                |<-text-----------|                |             |             |
 |                |                 |                |             |             |
 |                |--POST /query--->|                |             |             |
 |                |                 |                |             |             |
 |                |                 |--translate---->|             |             |
 |                |                 |<-English-------|             |             |
 |                |                 |                |             |             |
 |                |                 |--embed query-->|             |             |
 |                |                 |                |--Titan----->|             |
 |                |                 |                |<-embedding--|             |
 |                |                 |                |             |             |
 |                |                 |--search------->|             |             |
 |                |                 |                |--query----->|             |
 |                |                 |                |<-top-3------|             |
 |                |                 |<-docs----------|             |             |
 |                |                 |                |             |             |
 |                |                 |--check confidence            |             |
 |                |                 |  (score >= 0.6?)             |             |
 |                |                 |                |             |             |
 |                |                 |--build prompt->|             |             |
 |                |                 |                |             |             |
 |                |                 |--generate answer (Claude)--->|             |
 |                |                 |                |             |------------>|
 |                |                 |                |             |<-response---|
 |                |                 |<-answer + citations----------|             |
 |                |                 |                |             |             |
 |                |<-JSON response--|                |             |             |
 |                |                 |                |             |             |
 |<-display-------|                 |                |             |             |
 |                |                 |                |             |             |
```


---

### Data Flow Diagram: Mandi Price Intelligence

```
┌─────────────────────────────────────────────────────────────────┐
│                    AGMARKNET API (Daily)                        │
│                  (Government Mandi Data Source)                 │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             │ CSV Download (11 AM IST)
                             │
                             ▼
                    ┌────────────────┐
                    │  Cron Job      │
                    │  (Daily Sync)  │
                    └────────┬───────┘
                             │
                             │ Parse & Validate
                             │
                             ▼
                    ┌────────────────┐
                    │  SQLite DB     │
                    │  mandi_prices  │
                    │  (30 days)     │
                    └────────┬───────┘
                             │
                ┌────────────┼────────────┐
                │            │            │
                ▼            ▼            ▼
         ┌──────────┐ ┌──────────┐ ┌──────────┐
         │  Price   │ │  Trend   │ │  Best    │
         │  List    │ │  Calc    │ │  Mandi   │
         │  API     │ │  (7-day) │ │  Scorer  │
         └────┬─────┘ └────┬─────┘ └────┬─────┘
              │            │            │
              └────────────┼────────────┘
                           │
                           ▼
                  ┌────────────────┐
                  │  FastAPI       │
                  │  /mandi/*      │
                  └────────┬───────┘
                           │
                           │ JSON Response
                           │
                           ▼
                  ┌────────────────┐
                  │  Mobile App    │
                  │  Sell Smart    │
                  └────────────────┘
```

---

## Summary

This design document outlines a pragmatic, demo-ready architecture for Kheti Pulse, a rural farmer copilot. Key design decisions prioritize simplicity, reliability, and responsible AI practices within a 7-day development timeline.

The system leverages proven technologies (Flutter, FastAPI, Gemini, FAISS) and focuses on three core value propositions: weather risk alerts, mandi price intelligence, and conversational agricultural guidance. All features are multilingual (Marathi/Hindi) and designed with offline resilience.

Responsible AI guardrails—including citations, refusals, and explainability—are baked into every layer, ensuring farmers receive trustworthy, transparent information. The architecture is designed to scale beyond the MVP while maintaining the core principle: empowering farmers with timely, actionable insights.

---

**Document Status:** Ready for Implementation  
**Next Steps:** Backend scaffolding, knowledge base curation, mobile UI mockups  
**Review Date:** 2026-02-15

