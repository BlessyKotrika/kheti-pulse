# Requirements Document

## Overview

**Project Name:** Kheti Pulse

**Vision:** A one-stop AI-powered decision-support application that empowers farmers with timely, multilingual, and responsible guidance for agricultural decisions in rural ecosystems.

**Core Value Proposition:** Farmers receive actionable insights on weather risks, market prices, and agricultural best practices through an intelligent assistant that provides verified, cited information in their local language.

**Technology Stack:** AI/ML (LLM + RAG), multilingual NLP, public agricultural data APIs, Android mobile platform.

**Timeline:** 7-day hackathon build with demo-ready MVP.

---

## Problem Statement

Farmers in rural areas face critical information asymmetry that impacts their livelihoods:

1. **Weather Uncertainty:** Lack of timely, actionable weather alerts leads to crop damage and poor planning decisions.
2. **Market Opacity:** Limited visibility into mandi (market) prices across regions results in suboptimal selling decisions and reduced income.
3. **Knowledge Gaps:** Difficulty accessing reliable, localized agricultural knowledge in native languages, leading to reliance on word-of-mouth or outdated practices.
4. **Information Fragmentation:** Agricultural information scattered across multiple sources, apps, and government portals with no unified access point.
5. **Language Barriers:** Most digital agricultural tools available only in English, excluding non-English speaking farmers.

**Impact:** These challenges result in reduced farm income, increased crop losses, and missed opportunities for government schemes and modern farming practices.

---

## Goals and Non-Goals

### Goals

1. Provide a unified dashboard for daily agricultural decision-making
2. Enable price discovery across mandis with simple forecasts and recommendations
3. Deliver verified, cited agricultural knowledge through conversational AI in local languages
4. Build trust through transparent sourcing and clear limitation statements
5. Create a demo-ready MVP within 7 days that showcases core value

### Non-Goals

1. **Not** providing authoritative agricultural advice (decision-support only)
2. **Not** handling financial transactions or e-commerce
3. **Not** replacing agricultural extension services or expert consultation
4. **Not** supporting offline-first functionality
5. **Not** covering livestock, fisheries, or non-crop agriculture in MVP
6. **Not** building custom weather prediction models (use public APIs)

---

## Target Users / Personas

### Primary Persona: Progressive Farmer (Rajesh)

- **Age:** 35-50
- **Location:** Maharashtra (Vidarbha region)
- **Land:** 5-10 acres
- **Crops:** Cotton, Soybean
- **Language:** Marathi, basic Hindi
- **Tech:** Owns smartphone, moderate digital literacy
- **Pain Points:** Struggles to get timely market prices, relies on middlemen, wants to maximize income
- **Goals:** Sell crops at best prices, reduce weather-related losses, learn modern practices

### Secondary Persona: Young Farmer (Priya)

- **Age:** 25-35
- **Location:** Maharashtra (Vidarbha region)
- **Land:** 2-5 acres
- **Crops:** Cotton, Soybean
- **Language:** Marathi, Hindi, some English
- **Tech:** Smartphone-savvy, uses WhatsApp, YouTube
- **Pain Points:** Limited access to expert advice, wants to adopt sustainable practices
- **Goals:** Increase productivity, access government schemes, connect with knowledge sources

---

## User Stories

### US-1: Daily Dashboard Check

**As a** farmer  
**I want to** see today's critical actions and risks on my home screen  
**So that** I can make timely decisions without navigating multiple apps

**Acceptance Criteria:**
- **Given** I open the app in the morning
- **When** the home screen loads
- **Then** I see weather risk alerts for today (if any)
- **And** I see actionable "Do" and "Don't" cards based on weather and crop stage
- **And** all content is displayed in my selected language (Marathi or Hindi)

---

### US-2: Find Best Mandi Price

**As a** farmer ready to sell my harvest  
**I want to** see current mandi prices and get a recommendation for where to sell  
**So that** I can maximize my income

**Acceptance Criteria:**
- **Given** I navigate to "Sell Smart" module
- **When** I select my crop (Cotton or Soybean) and quantity
- **Then** I see a list of nearby mandis with current prices
- **And** I see a simple price trend (up/down/stable) for the past week
- **And** I see a "Best Mandi" recommendation with net estimate (price minus transport cost)
- **And** the recommendation explains why it's the best option

---

### US-3: Ask Agricultural Question

**As a** farmer with a specific agricultural question  
**I want to** ask the AI assistant in my language  
**So that** I get verified answers with sources

**Acceptance Criteria:**
- **Given** I open the "Ask Assistant" feature
- **When** I type or speak a question in Marathi or Hindi
- **Then** the assistant responds in the same language
- **And** the answer includes citations to source documents
- **And** if no reliable source is found, the assistant says "I cannot answer this confidently"
- **And** the assistant never provides medical or legal advice

---

### US-4: Voice Interaction (Optional)

**As a** farmer with limited literacy  
**I want to** speak my questions instead of typing  
**So that** I can use the app more easily

**Acceptance Criteria:**
- **Given** voice input is enabled
- **When** I tap the microphone icon and speak in Marathi
- **Then** my speech is converted to text accurately
- **And** the assistant responds with both text and optional audio output

---

### US-5: Explore Government Schemes (Stretch)

**As a** farmer  
**I want to** discover relevant government schemes and subsidies  
**So that** I can access financial support

**Acceptance Criteria:**
- **Given** I navigate to "Schemes" section
- **When** I filter by crop type and location
- **Then** I see a list of applicable schemes with eligibility criteria
- **And** I can view required documents and application links

---

## Feature List (MVP vs Stretch)

| Feature | Priority | Description | Platform |
|---------|----------|-------------|----------|
| **Home Dashboard** | MVP | Today's weather risk + action cards (do/don't) | Android |
| **Language Selection** | MVP | Toggle between Marathi and Hindi | Android |
| **Sell Smart - Price List** | MVP | Current mandi prices for Cotton, Soybean | Android |
| **Sell Smart - Trends** | MVP | 7-day price trend visualization (line chart) | Android |
| **Sell Smart - Best Mandi** | MVP | Recommendation with net estimate | Android |
| **Ask Assistant - Text Chat** | MVP | Multilingual Q&A with RAG + citations | Android |
| **Ask Assistant - Refusal Logic** | MVP | "I don't know" when sources unavailable | Android |
| **Knowledge Base** | MVP | Curated agricultural documents (synthetic/public) | Backend |
| **Voice Input** | MVP | Speech-to-text in Marathi, Hindi | Android |
| **Voice Output** | Stretch | Text-to-speech responses | Android |
| **Schemes Explorer** | Stretch | Government scheme discovery + documents | Android |
| **Crop Calendar** | Stretch | Stage-based recommendations | Android |
| **Push Notifications** | Stretch | Weather alerts, price spikes | Android |

---

## Functional Requirements (FR)

### FR-1: Home Dashboard

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-1.1 | Display current date, location, and selected language | P0 |
| FR-1.2 | Fetch and display today's weather forecast for Maharashtra (Vidarbha region) | P0 |
| FR-1.3 | Generate weather risk alerts (e.g., "Heavy rain expected - postpone spraying") | P0 |
| FR-1.4 | Display 2-3 actionable "Do" cards (e.g., "Check drainage systems") | P0 |
| FR-1.5 | Display 2-3 actionable "Don't" cards (e.g., "Don't apply fertilizer today") | P0 |
| FR-1.6 | Refresh dashboard data on app launch or manual pull-to-refresh | P0 |

### FR-2: Sell Smart Module

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-2.1 | Allow user to select crop from list (Cotton, Soybean) | P0 |
| FR-2.2 | Allow user to input quantity (in quintals or kg) | P0 |
| FR-2.3 | Fetch current mandi prices from public API for Maharashtra mandis | P0 |
| FR-2.4 | Display list of mandis with prices, sorted by price (descending) | P0 |
| FR-2.5 | Show 7-day price trend as line chart for selected crop | P0 |
| FR-2.6 | Calculate net estimate: (price × quantity) - estimated transport cost | P0 |
| FR-2.7 | Recommend "Best Mandi" based on net estimate | P0 |
| FR-2.8 | Provide explanation for recommendation (e.g., "Highest net return after transport") | P0 |
| FR-2.9 | Handle missing price data gracefully (show "Data unavailable") | P1 |

### FR-3: Ask Assistant (RAG-based Chat)

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-3.1 | Accept text input in Marathi or Hindi | P0 |
| FR-3.2 | Detect input language automatically or use user-selected language | P0 |
| FR-3.3 | Retrieve relevant documents from knowledge base using semantic search | P0 |
| FR-3.4 | Generate answer using LLM with retrieved context | P0 |
| FR-3.5 | Include citations (document name, section) in response | P0 |
| FR-3.6 | If no relevant sources found (confidence < threshold), respond: "I cannot answer this confidently. Please consult an agricultural expert." | P0 |
| FR-3.7 | Refuse to answer medical, legal, or financial advice questions | P0 |
| FR-3.8 | Display chat history within session | P1 |
| FR-3.9 | Accept voice input (speech-to-text) using Android Speech Recognition | P0 (MVP) |
| FR-3.10 | Provide voice output (text-to-speech) | P2 (Stretch) |

### FR-4: Multilingual Support

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-4.1 | Support Marathi and Hindi for all UI text | P0 |
| FR-4.2 | Provide language toggle in settings or header | P0 |
| FR-4.3 | Persist language preference across sessions | P0 |
| FR-4.4 | Translate LLM responses to selected language | P0 |

### FR-5: Data Management

| ID | Requirement | Priority |
|----|-------------|----------|
| FR-5.1 | Cache weather and price data for offline viewing (limited connectivity scenarios) | P1 |
| FR-5.2 | Refresh data automatically when internet is available | P1 |
| FR-5.3 | Log user queries (anonymized) for analytics | P2 |

---

## Non-Functional Requirements (NFR)

| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-1 | Performance | Home dashboard loads within 3 seconds on 3G network | <3s |
| NFR-2 | Performance | Chat response time (including LLM + RAG retrieval) | <5s |
| NFR-2.1 | Performance | App size (APK) | <50 MB |
| NFR-3 | Usability | App usable by farmers with basic smartphone literacy | N/A |
| NFR-4 | Usability | Font size readable in outdoor sunlight | ≥16px |
| NFR-5 | Accessibility | Support screen readers for visually impaired users | P2 |
| NFR-6 | Reliability | Handle API failures gracefully (show cached data or error message) | 100% |
| NFR-7 | Reliability | App uptime during demo | 99% |
| NFR-8 | Security | No storage of personally identifiable information (PII) | 100% |
| NFR-9 | Security | HTTPS for all API calls | 100% |
| NFR-10 | Scalability | Support 100 concurrent users during demo | 100 users |
| NFR-11 | Localization | All UI strings externalized for translation (Marathi, Hindi) | 100% |
| NFR-12 | Maintainability | Code documented with inline comments | ≥80% coverage |
| NFR-13 | Compatibility | Support Android 8.0 (API 26) and above | 95% device coverage |
| NFR-14 | Battery | Minimal battery drain (no continuous background processes) | <5% per hour |

---

## Data Sources (public/synthetic only) + Data Handling

### Weather Data

- **Source:** India Meteorological Department (IMD) public API or OpenWeatherMap API
- **Data Points:** Temperature, precipitation probability, humidity, wind speed, 5-day forecast
- **Update Frequency:** Every 6 hours
- **Handling:** Cache latest forecast; display "Last updated" timestamp; show alerts for extreme weather (heavy rain, heatwave)
- **Demo Coverage:** Nagpur, Akola, Amravati, Yavatmal districts (Vidarbha region)

### Mandi Price Data

- **Source:** Government of India Agmarknet API (https://agmarknet.gov.in) or Maharashtra State Agricultural Marketing Board
- **Data Points:** Crop name, mandi name, price (min/max/modal), arrival quantity, date
- **Update Frequency:** Daily (updated by 10 AM IST)
- **Handling:** Store 30 days of historical data for trend analysis; fallback to synthetic data if API unavailable
- **Demo Mandis:** Nagpur, Akola, Amravati, Yavatmal, Wardha (major cotton/soybean markets in Vidarbha)

### Agricultural Knowledge Base

- **Source:** Public agricultural extension documents, FAQs from ICAR, Maharashtra Agriculture Department, Krishi Vigyan Kendras (KVKs), or curated synthetic documents
- **Data Points:** PDF/text documents covering:
  - Cotton cultivation practices (sowing, spacing, irrigation)
  - Soybean crop management (varieties, fertilizer, pest control)
  - Integrated Pest Management (IPM) for bollworm, whitefly, pod borer
  - Soil health management for black cotton soil (Vidarbha)
  - Water conservation techniques (drip irrigation, mulching)
  - Post-harvest handling and storage
- **Processing:** Convert to embeddings using multilingual models; chunk documents for semantic search (RAG)
- **Handling:** Version-controlled document repository; periodic updates
- **Initial Size:** 30-50 curated documents (10-15 MB total)

### Government Schemes (Stretch)

- **Source:** Scraped from official government portals or manually curated list
- **Data Points:** Scheme name, eligibility, benefits, application process, documents required
- **Handling:** Static JSON file updated manually

### Transport Cost Estimates

- **Source:** Synthetic data based on average fuel costs and distances in Maharashtra
- **Data Points:** Distance between user location and mandi, cost per km (₹8-12/km for truck transport)
- **Handling:** Simple lookup table with district-to-mandi distances; formula: `transport_cost = distance_km × ₹10 × (quantity_quintals / 100)`
- **Assumptions:** Shared truck transport; costs averaged across vehicle types

---

## Responsible AI & Safety Requirements

### R-1: Citation and Transparency

- **Requirement:** Every AI-generated answer must include citations to source documents.
- **Implementation:** Display source document name and relevant section/page number below each answer.
- **Example:** "Source: ICAR Pest Management Guide, Section 3.2"

### R-2: Refusal Policy

- **Requirement:** The assistant must refuse to answer when:
  1. No relevant sources are found in the knowledge base (confidence score < 0.6)
  2. Question is outside agricultural domain (medical, legal, financial advice)
  3. Question asks for predictions beyond model capability (e.g., exact future prices)
- **Response Template:** "I cannot answer this confidently. Please consult [appropriate expert type]."

### R-3: "I Don't Know" Responses

- **Requirement:** The assistant must explicitly state uncertainty rather than hallucinating information.
- **Implementation:** 
  - If retrieval returns no relevant documents: "I don't have information on this topic in my knowledge base."
  - If question is ambiguous: "Could you please clarify your question? For example, are you asking about [option A] or [option B]?"

### R-4: Limitation Disclaimers

- **Requirement:** Display prominent disclaimer on first app launch and in assistant interface.
- **Disclaimer Text:** "This app provides decision-support information only. It is not a substitute for professional agricultural advice. Always verify critical decisions with local experts."

### R-5: Bias Mitigation

- **Requirement:** Ensure knowledge base includes diverse sources and perspectives.
- **Implementation:** Review documents for regional bias; include content relevant to small and marginal farmers.

### R-6: Data Privacy

- **Requirement:** Do not collect or store personally identifiable information (PII).
- **Implementation:** Anonymize query logs; no user accounts or authentication in MVP.

### R-7: Harmful Content Prevention

- **Requirement:** Prevent generation of harmful advice (e.g., unsafe pesticide use).
- **Implementation:** Content filtering layer; manual review of knowledge base documents.

---

## Assumptions & Constraints

### Assumptions

1. Target users have access to Android smartphones with intermittent internet connectivity (2G/3G/4G)
2. Users are familiar with basic smartphone gestures (tap, scroll, swipe)
3. Public APIs for weather and mandi prices are available and reliable
4. Curated knowledge base documents are accurate and up-to-date
5. Users understand that the app provides decision-support, not authoritative advice
6. Demo will focus on Maharashtra (Vidarbha region) and crops Cotton, Soybean

### Constraints

1. **Timeline:** 7-day development window for MVP
2. **Data:** Only public or synthetic data; no proprietary datasets
3. **Platform:** Android only (no iOS or web in MVP)
4. **Languages:** Marathi and Hindi only (English for fallback)
5. **Geography:** Maharashtra (Vidarbha region) only for demo
6. **Crops:** Cotton and Soybean only for demo
7. **Internet:** Demo assumes intermittent connectivity (3G/4G)
8. **Team:** Hackathon team size and skill constraints
9. **Budget:** No paid APIs or services (use free tiers only)
10. **Scope:** No user authentication, payments, or e-commerce features

---

## Out of Scope

The following are explicitly out of scope for the MVP:

1. **User Accounts & Authentication:** No login, profiles, or personalization
2. **E-commerce:** No buying/selling transactions within the app
3. **Social Features:** No farmer community, forums, or chat between users
4. **IoT Integration:** No sensor data, soil monitors, or smart farming devices
5. **Livestock & Fisheries:** Focus on crop agriculture only
6. **Financial Services:** No loans, insurance, or payment processing
7. **Supply Chain:** No logistics, warehousing, or distribution features
8. **Custom ML Models:** Use pre-trained LLMs and public APIs only
9. **Multi-region Support:** Indian regions, Telangana only for demo
10. **Advanced Analytics:** No predictive modeling beyond simple price trends
11. **Full Offline Mode:** Basic caching only; not a fully offline-first architecture
12. **Desktop/Web Version:** Android mobile app only

---

## Success Metrics

### Demo Success Criteria

1. **Functional Completeness:** All MVP features working end-to-end
2. **User Flow:** Smooth navigation through home → sell smart → ask assistant
3. **Response Quality:** AI assistant provides relevant, cited answers for 80% of test questions
4. **Multilingual:** All features work in both Marathi and Hindi
5. **Performance:** No crashes or freezes during 15-minute demo
6. **Wow Factor:** At least one "delightful" moment (e.g., accurate voice input, insightful price recommendation)

### Post-Demo Metrics (if deployed)

1. **Adoption:** 100+ farmers onboarded in first month
2. **Engagement:** 60% weekly active users (WAU)
3. **Retention:** 40% users return after 7 days
4. **Assistant Usage:** 50% of users ask at least one question
5. **Satisfaction:** 4+ star rating from user feedback
6. **Impact:** Farmers report 10%+ improvement in selling decisions (survey)

---

## Open Questions

The following technical decisions need to be finalized:

1. **LLM Choice:** Which LLM will be used for the assistant? Options:
   - Google Gemini (free tier, good multilingual support)
   - GPT-3.5/4 (OpenAI API)
   - Open-source (Llama 3, Mistral) via Hugging Face
2. **Weather API:** Confirm access to IMD API or use OpenWeatherMap free tier?
3. **Mandi Price API:** Use Agmarknet API or create synthetic data for demo?
4. **Backend Hosting:** Firebase (easy Android integration) vs AWS/GCP free tier?
5. **Voice Recognition:** Use Google Speech-to-Text API or Android native speech recognition?
6. **Knowledge Base Size:** Target 20-50 curated documents or more?
7. **Vector Database:** Use Pinecone (free tier), ChromaDB (local), or Firebase Vector Search?
8. **Authentication:** Skip for MVP or add simple phone-based auth for personalization?
9. **Analytics:** Integrate Firebase Analytics or skip for hackathon?
10. **Testing Strategy:** Manual testing only or include basic automated tests?
11. **Design Assets:** Use Material Design components or custom UI kit?
12. **Transport Cost Data:** Use Google Maps Distance Matrix API or static lookup table?

---

## Appendix: Glossary

| Term | Definition |
|------|------------|
| **Mandi** | Agricultural market or wholesale market where farmers sell produce |
| **Quintal** | Unit of weight equal to 100 kg, commonly used in Indian agriculture |
| **RAG** | Retrieval-Augmented Generation; AI technique that retrieves relevant documents before generating answers |
| **LLM** | Large Language Model; AI model trained on text data for natural language understanding and generation |
| **MVP** | Minimum Viable Product; core features needed for initial launch |
| **ICAR** | Indian Council of Agricultural Research; government body for agricultural research |
| **IMD** | India Meteorological Department; national weather forecasting agency |
| **Agmarknet** | Government of India portal for agricultural marketing information |
| **Modal Price** | Most frequently occurring price in a mandi on a given day |
| **Decision-Support** | Providing information to aid decision-making, not prescriptive advice |
| **Citation** | Reference to source document that supports an AI-generated answer |
| **Refusal Policy** | Rules for when AI assistant should decline to answer a question |
| **Semantic Search** | Search technique that understands meaning, not just keyword matching |
| **Embedding** | Numerical representation of text used for semantic similarity |
| **P0/P1/P2** | Priority levels (P0 = critical, P1 = important, P2 = nice-to-have) |
| **Kheti Pulse** | App name; "Kheti" means farming in Hindi/Marathi; "Pulse" signifies real-time updates |
| **Vidarbha** | Eastern region of Maharashtra known for cotton cultivation; includes Nagpur, Akola, Amravati |
| **KVK** | Krishi Vigyan Kendra; agricultural extension centers providing farmer training |
| **Bollworm** | Major cotton pest in India; requires integrated pest management |
| **Black Cotton Soil** | Predominant soil type in Vidarbha; high clay content, moisture retention |

---

**Document Version:** 1.0  
**Last Updated:** 2026-02-14  
**Status:** Ready for Development
