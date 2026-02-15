# System Design: SATHI Platform

## 1. Overview

SATHI is an AI-powered civic opportunity discovery platform helping underserved communities access scholarships, government schemes, internships, and skill-development programs through intelligent matching, multilingual support, and voice-first interaction.

**Key Differentiators**: AI eligibility evaluation, personalized recommendations, RAG-based chatbot, multilingual summarization, fake opportunity detection.

## 2. System Architecture

```
┌─────────────────────────────────────────────────┐
│         CLIENT LAYER (Mobile/Web/PWA)           │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│    API GATEWAY (Auth, Rate Limiting, Routing)   │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  APPLICATION SERVICES                           │
│  • User Service      • Opportunity Service      │
│  • Content Service   • Search Service           │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  AI SERVICES                                    │
│  • Eligibility Engine    • Recommendation       │
│  • RAG Chatbot          • Voice (STT/TTS)       │
│  • Translation          • Fake Detection        │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│  DATA LAYER                                     │
│  • PostgreSQL  • Redis  • S3  • Vector DB       │
└─────────────────────────────────────────────────┘
```

**Architecture Principles**:
- Microservices with clear domain boundaries
- Stateless services for horizontal scaling
- AI-first design (AI in every major feature)
- Polyglot persistence (right database for each use case)

## 3. Core Components

### 3.1 Application Services

**User Service** (Node.js, Express)
- Authentication (JWT-based)
- Profile management (age, education, location, goals)
- User preferences and activity tracking

**Opportunity Service** (Python, FastAPI)
- CRUD for opportunities (scholarships, schemes, jobs)
- Eligibility criteria management
- Deadline tracking and source verification

**Content Service** (Node.js, Express)
- Video metadata and storage (S3)
- Creator verification workflow
- Content moderation queue

**Search Service** (Python, FastAPI)
- Full-text search via Elasticsearch
- Faceted filtering and ranking
- Auto-suggestions

### 3.2 AI Services

**Eligibility Engine** (Python, FastAPI)
- Rule-based evaluation of user eligibility
- Generates human-readable explanations
- Suggests next steps for partial eligibility

**How it works**:
```
1. Load user profile + opportunity criteria
2. Evaluate rules (age, education, income, location)
3. Generate explanation for each criterion
4. Return: eligible/not eligible/partial + reasons + next steps
```

**Recommendation Engine** (Python, scikit-learn)
- Hybrid approach: content-based + collaborative filtering
- Scores opportunities based on user profile and behavior
- Real-time personalization

**Scoring**: `0.4 × content_similarity + 0.3 × collaborative + 0.2 × popularity + 0.1 × recency`

**RAG Chatbot** (Python, LangChain)
- Retrieval-Augmented Generation for accurate responses
- Vector search for relevant opportunities
- Source attribution for transparency

**Flow**:
```
Query → Embed → Vector Search → Retrieve Context → LLM Generate → Response + Sources
```

**Voice Service** (Python, Google/Azure APIs)
- Speech-to-text for voice input
- Text-to-speech for voice output
- Multilingual support (10+ Indian languages)

**Translation & Summarization** (Python, Transformers)
- mT5/mBART models for multilingual summarization
- Extracts key info: benefits, eligibility, deadline
- Max 150 words per summary

**Fake Detection** (Python, BERT classifier)
- Source verification against official databases
- Text analysis for scam indicators
- Confidence scoring (0-100)

**Thresholds**: 80+ verified, 50-79 unverified, <50 suspicious

## 4. Database Design

### Core Entities

```sql
users (id, phone_number, email, password_hash, preferred_language, created_at)

user_profiles (user_id FK, age, education_level, location, interests JSONB, goals JSONB)

opportunities (id, title, description, type, source_url, verification_status, 
               confidence_score, deadline, benefits, created_at)

eligibility_criteria (id, opportunity_id FK, criterion_type, operator, value JSONB)

videos (id, opportunity_id FK, creator_id FK, title, video_url, thumbnail_url, 
        duration, language, validation_score, view_count)

user_interactions (id, user_id FK, opportunity_id FK, interaction_type, timestamp)

chat_sessions (id, user_id FK, started_at, ended_at)

chat_messages (id, session_id FK, sender, message_text, language, timestamp)
```

**Relationships**:
- Users 1:1 Profiles
- Users 1:N Videos (as creators)
- Opportunities 1:N Eligibility Criteria
- Opportunities 1:N Videos
- Users 1:N Interactions
- Chat Sessions 1:N Messages

**Indexes**: Foreign keys, (user_id, timestamp), full-text on opportunity title/description

## 5. API Design

### Authentication
```
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
```

### User
```
GET /api/v1/users/profile
PUT /api/v1/users/profile
```

### Opportunities
```
GET /api/v1/opportunities?type=scholarship&language=hi&page=1
GET /api/v1/opportunities/:id
POST /api/v1/opportunities/:id/check-eligibility
```

### Recommendations
```
GET /api/v1/recommendations
```

### Search
```
GET /api/v1/search?q=engineering+scholarship
```

### Chatbot
```
POST /api/v1/chat/sessions
POST /api/v1/chat/sessions/:id/messages
```

### Voice
```
POST /api/v1/voice/speech-to-text
POST /api/v1/voice/text-to-speech
```

### Videos
```
GET /api/v1/videos/feed?language=hi
POST /api/v1/videos/:id/interact
```

## 6. Data Flows

### User Onboarding
```
1. User opens app → selects language
2. POST /api/v1/auth/register (phone, password)
3. User Service creates account in PostgreSQL
4. JWT tokens issued
5. User completes profile
6. PUT /api/v1/users/profile
7. Recommendation Engine generates initial suggestions
8. User sees personalized feed
```

### Eligibility Check
```
1. User views opportunity → clicks "Check Eligibility"
2. POST /api/v1/opportunities/:id/check-eligibility
3. Eligibility Engine loads user profile + criteria
4. Evaluates each rule (age ✓, education ✓, income ✗)
5. Generates explanation: "Income exceeds limit"
6. Returns: not eligible + reasons + suggestions
7. Logs interaction for analytics
```

### Recommendation Generation
```
1. GET /api/v1/recommendations
2. Load user profile + interaction history
3. Content-based: match profile to opportunity features
4. Collaborative: find similar users, get their interactions
5. Combine scores with popularity and recency
6. Rank and filter (remove expired, already applied)
7. Cache in Redis (TTL: 1 hour)
8. Return top 20 recommendations
```

### Chat Interaction (RAG)
```
1. User asks: "What scholarships for engineering students?"
2. POST /api/v1/chat/sessions/:id/messages
3. Embed query using sentence transformer
4. Vector search in opportunity embeddings
5. Retrieve top 5 relevant opportunities
6. Construct prompt: system + context + query
7. LLM generates response with sources
8. Store message pair in PostgreSQL
9. Return response + opportunity links
```

### Voice Search
```
1. User speaks in Hindi: "मुझे छात्रवृत्ति चाहिए"
2. POST /api/v1/voice/speech-to-text
3. Google Speech API transcribes
4. Translation Service converts to English
5. Search Service queries Elasticsearch
6. Results ranked and translated back to Hindi
7. Optional: TTS reads results aloud
```

## 7. Security

### Authentication
- JWT tokens (access: 1 hour, refresh: 30 days)
- Passwords hashed with bcrypt (cost: 12)
- Token rotation on refresh

### Authorization (RBAC)
- **User**: View, check eligibility, chat, interact
- **Creator**: User + upload videos, view analytics
- **Admin**: All + moderation, user management

### Data Security
- TLS 1.3 for all communication
- Database encryption at rest
- Sensitive fields encrypted
- Rate limiting: 100 req/min per user

### Input Validation
- Schema validation on all requests
- SQL injection prevention (parameterized queries)
- XSS prevention (content security policies)

## 8. Deployment

### Infrastructure (AWS)
- **Compute**: Docker containers on ECS/EKS, auto-scaling
- **Database**: RDS PostgreSQL (Multi-AZ), ElastiCache Redis
- **Storage**: S3 for videos, CloudFront CDN
- **Search**: Elasticsearch Service
- **Vector DB**: Pinecone or Weaviate

### CI/CD
- GitHub Actions for build/test/deploy
- Automated tests (unit, integration)
- Rolling updates for zero downtime
- Canary releases (10% → 50% → 100%)

### Monitoring
- CloudWatch for metrics and logs
- Alerts for errors (>5%), latency (>2s), downtime
- Health checks: liveness + readiness probes

## 9. AI Model Details

### Recommendation Model
- **Type**: Hybrid (content + collaborative)
- **Training**: Weekly on historical data
- **Inference**: Batch (nightly) + real-time scoring
- **Metrics**: Precision@k, recall@k

### Chatbot Model
- **Type**: RAG (Retrieval-Augmented Generation)
- **Embedding**: sentence-transformers/all-MiniLM-L6-v2
- **LLM**: GPT-4 or Llama-2-13B
- **Knowledge Base**: Opportunity docs in vector DB

### Summarization Model
- **Type**: Abstractive (mT5/mBART)
- **Input**: Full opportunity document
- **Output**: 150-word summary in user's language

### Fake Detection Model
- **Type**: BERT-based binary classifier
- **Features**: Source domain, text patterns, structure
- **Output**: Confidence score + classification

## 10. Performance

### Caching
- Redis: sessions (1h), recommendations (1h), eligibility (24h)
- CDN: videos (7d), static assets (30d)

### Response Time Targets
- Auth: <500ms, Eligibility: <1s, Recommendations: <2s, Chat: <3s, Search: <1s

### Optimization
- Pagination (20 items/page)
- Lazy loading for videos
- Gzip compression
- Connection pooling (20 max)

### Low-Bandwidth Mode
- Progressive image loading
- Video quality selection (360p/480p/720p)
- Offline caching
- Lightweight payloads

## 11. MVP Phasing

### Phase 1 (Months 1-3): Core MVP
- User auth, opportunity listing, search
- AI eligibility checker
- Basic recommendations (content-based)
- Simple chatbot (rule-based + RAG)
- English + Hindi support

### Phase 2 (Months 4-6): AI Enhancement
- Hybrid recommendations
- Advanced RAG chatbot
- Voice support (STT/TTS)
- 5 languages
- Video content + moderation
- Fake detection

### Phase 3 (Months 7-12): Scale
- 10+ languages
- Creator program
- Analytics dashboard
- Mobile optimization
- Community features

## 12. Success Metrics

**User Engagement**: 10K MAU (Month 6), 100K (Month 12), 8+ min session duration

**AI Performance**: 30% CTR (recommendations), 85% accuracy (chatbot), 95% accuracy (eligibility), 90% precision (fake detection)

**Impact**: 50K opportunities discovered (Month 6), 5K applications initiated, 500 success stories (Month 12)

**Technical**: <2s response time (95th percentile), 99% uptime, <1% error rate

## 13. Risks & Mitigations

**AI Accuracy**: Regular evaluation, human-in-the-loop, fallback to rules
**Scalability**: Load testing, auto-scaling, caching, optimization
**Data Quality**: Source verification, validation, audits
**Content Moderation**: Automated + manual review, clear policies
**Fake Opportunities**: Multi-layer detection, confidence scoring
**Privacy**: Data minimization, encryption, clear policies

## 14. Conclusion

SATHI is an AI-first platform democratizing access to civic opportunities through intelligent eligibility matching, personalized recommendations, conversational assistance, and multilingual support. The MVP-focused architecture ensures rapid deployment while maintaining scalability for growth. AI is central to every feature, providing real value to underserved communities.
