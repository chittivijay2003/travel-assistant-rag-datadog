# 🌍 RAG Travel Assistant with LangGraph & Observability

A production-ready RAG (Retrieval Augmented Generation) powered travel assistant built with **Qdrant** vector search, **Gemini 2.0** LLM, **LangGraph** orchestration, **FastAPI**, and comprehensive observability through **LangFuse** and **Datadog**.

## 🌟 Features

### Core Capabilities
- **Advanced Hybrid Search**: 
  - Combines semantic (dense vectors) and keyword (sparse vectors) search
  - Country name boosting (3x weight) for accurate retrieval
  - Minimum score threshold (0.45) to filter irrelevant results
  - Reciprocal Rank Fusion (RRF) for optimal result ranking
  
- **LangGraph Workflow**: 
  - Multi-step orchestrated pipeline with conditional routing
  - 5 nodes: Input → Retrieval → Generation → Error → Output
  - Async execution for optimal performance
  
- **Gemini 2.0 Flash Integration**: 
  - Natural language generation with context awareness
  - Token usage tracking for cost monitoring
  - Configurable temperature and max tokens
  - Simple, emoji-enhanced responses for better UX
  
- **Dual API Endpoints**:
  - JSON endpoint with structured responses
  - Plain text endpoint for proper line break rendering
  
- **Comprehensive Observability**: 
  - **LangFuse**: RAG pipeline tracing, token tracking, cost monitoring
  - **Datadog**: APM for application performance monitoring
  
- **Travel Knowledge Base**: 15 destinations with detailed information:
  - Visa requirements for Indian citizens
  - Processing times and required documents
  - Attractions and best time to visit
  - Climate, currency, and language information
  
- **Production-Ready**:
  - FastAPI backend with automatic documentation
  - Comprehensive error handling and logging
  - Health check endpoints
  - Environment-based configuration

## 📋 Prerequisites

- Python 3.13+
- Qdrant (running locally or cloud)
- Gemini API key
- (Optional) LangFuse account for tracing
- (Optional) Datadog account for APM

## 🚀 Quick Start

### 1. Install Dependencies

Using uv (recommended):
```bash
uv sync
```

Or using pip:
```bash
pip install -r requirements.txt
```

### 2. Start Qdrant

Using Docker:
```bash
docker run -p 6333:6333 qdrant/qdrant
```

Or use Qdrant Cloud and update `QDRANT_URL` in `.env`

### 3. Configure Environment Variables

Create a `.env` file in the project root:
```env
# Required - Gemini API
GEMINI_API_KEY=your_gemini_api_key_here

# Required - Qdrant Configuration
QDRANT_URL=http://localhost:6333
QDRANT_COLLECTION_NAME=travel_destinations

# Optional - LangFuse (for observability)
LANGFUSE_PUBLIC_KEY=your_langfuse_public_key_here
LANGFUSE_SECRET_KEY=your_langfuse_secret_key_here
LANGFUSE_HOST=https://us.cloud.langfuse.com

# Optional - Datadog (for APM)
DATADOG_API_KEY=your_datadog_api_key_here
DATADOG_APP_KEY=your_datadog_app_key_here

# Optional - Server Configuration
APP_HOST=0.0.0.0
APP_PORT=8000
LOG_LEVEL=INFO
```

### 4. Load Travel Data into Qdrant

```bash
uv run python scripts/ingest_data.py
```

### 5. Start the API Server

```bash
uv run python main.py
```

The API will be available at: `http://localhost:8000`

## 📚 API Documentation

Once the server is running, visit:
- **Swagger UI**: http://localhost:8000/docs
- **ReDoc**: http://localhost:8000/redoc

### Key Endpoints

#### POST `/api/v1/rag-travel-assistant` (JSON Response)

Query the travel assistant with any travel-related question. Returns JSON with structured response.

**Request:**
```json
{
  "query": "What are visa requirements for Indians traveling to Japan?",
  "top_k": 3,
  "return_sources": true
}
```

**Response:**
```json
{
  "answer": "**Japan** 🌍\n\n**Do I need a visa?**\n• Yes - Tourist visa required for Indian citizens\n• Processing time: 5-7 working days\n• Stay duration: Up to 15 days\n\n**Required Documents:**\n✈️ Valid passport (6 months minimum validity)\n📄 Completed visa application form\n📸 Recent passport-size photograph\n✈️ Flight itinerary\n🏨 Hotel reservation or invitation letter\n💰 Bank statements (last 6 months)\n📋 Employment proof or business registration\n💵 Income tax returns\n\n**Best time to visit:**\n🌸 Spring (March-May) - Cherry blossoms\n🍂 Autumn (September-November) - Pleasant weather\n\n**Popular destinations:** Tokyo, Kyoto, Osaka, Hiroshima",
  "query": "What are visa requirements for Indians traveling to Japan?",
  "sources_count": 1,
  "usage": {
    "model": "gemini-2.0-flash-exp",
    "input_tokens": 450,
    "output_tokens": 180,
    "total_tokens": 630
  },
  "sources": [
    {
      "country": "Japan",
      "title": "Japan Travel Guide",
      "score": 0.6247,
      "id": "dest_001"
    }
  ]
}
```

#### POST `/api/v1/rag-travel-assistant/text` (Plain Text Response)

Same as above but returns plain text with proper line breaks (no JSON escaping).

**Request:**
```json
{
  "query": "Tell me about visa requirements for Singapore",
  "top_k": 2,
  "return_sources": false
}
```

**Response (Plain Text):**
```
**Singapore** 🌍

**Do I need a visa?**
• No visa required for Indian citizens
• Visa-free entry for stays up to 30 days
• Just need a valid passport

**Best time to visit:**
☀️ Year-round destination with tropical climate
🎉 Visit during festivals for cultural experience

**Popular attractions:** Marina Bay Sands, Gardens by the Bay, Sentosa Island
```

#### GET `/api/v1/health`

Health check endpoint.

**Response:**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "qdrant_connected": true
}
```

## 🏗️ Architecture

### Project Structure

```
travel-assistant-rag-datadog/
├── app/
│   ├── api/
│   │   └── routes.py              # FastAPI routes
│   ├── graph/
│   │   ├── state.py               # LangGraph state definition
│   │   ├── nodes.py               # LangGraph nodes (input, retrieval, generation)
│   │   └── travel_assistant.py   # Complete graph workflow
│   ├── observability/
│   │   ├── langfuse.py           # LangFuse tracing integration
│   │   └── datadog.py            # Datadog APM integration
│   ├── rag/
│   │   ├── vector_store.py       # Qdrant vector store management
│   │   ├── retriever.py          # Hybrid search implementation
│   │   └── pipeline.py           # RAG pipeline with Gemini
│   ├── config.py                 # Configuration management
│   ├── logging_config.py         # Logging setup
│   └── models.py                 # Pydantic models
├── data/
│   └── destinations.json         # Travel destination data
├── scripts/
│   └── ingest_data.py           # Data ingestion script
├── logs/                        # Application logs
├── main.py                      # FastAPI application
├── pyproject.toml              # Dependencies
└── README.md
```

### LangGraph Workflow

```
Input → Retrieval → Generation → Output
         ↓           ↓
       Error ←─────┘
```

**Nodes:**
1. **Input Node**: Validates query and prepares state
2. **Retrieval Node**: Performs hybrid search (semantic + keyword)
3. **Generation Node**: Generates answer using Gemini with retrieved context
4. **Error Node**: Handles errors gracefully
5. **Output Node**: Formats final response

### Hybrid Search Implementation

The retriever combines multiple techniques for accurate results:

- **Dense Vectors**: Semantic similarity using sentence transformers (all-MiniLM-L6-v2, 384 dimensions)
- **Sparse Vectors**: Keyword matching using term frequency with country name boosting
- **Country Boosting**: 3x weight for country keywords (Japan, Thailand, Singapore, etc.)
- **Score Filtering**: Minimum score threshold of 0.45 to filter irrelevant results
- **Fusion Strategy**: Reciprocal Rank Fusion (RRF) to combine results optimally
- **Fallback**: Semantic-only search when hybrid search fails

**Example:**
Query: "Japan visa requirements" → Only returns Japan (score ~0.62), filters out irrelevant countries

## 🔍 Sample Queries

Try these example queries to see the RAG system in action:

### Visa-Related Queries
1. "What are visa requirements for Indians traveling to Japan?"
2. "Which countries offer visa-free entry for Indian citizens?"
3. "What documents do I need for a US tourist visa?"
4. "How long does it take to process a Singapore visa?"

### Travel Information
5. "Best time to visit Switzerland?"
6. "Tell me about attractions in Thailand"
7. "What is the climate like in Maldives?"
8. "What currency is used in Dubai?"

### Comparative Queries
9. "Compare visa requirements for Thailand and Maldives"
10. "Which is better for beaches - Thailand or Bali?"

### General Travel
11. "What languages are spoken in France?"
12. "Tell me about visiting Nepal from India"

## 📊 Observability & Monitoring

### LangFuse Tracing

All RAG operations are automatically traced with the `@observe` decorator:

**Traces Created:**
- `hybrid_retrieval` - Retrieval operations with query and results
- `llm_generation` - LLM calls with token usage and costs
- `rag_pipeline` - Complete end-to-end pipeline

**Metrics Tracked:**
- Token usage (input, output, total)
- Model costs (calculated from token usage)
- Query latency
- Retrieval scores
- Error rates

**Setup LangFuse Cost Tracking:**

1. **Configure Model Pricing in LangFuse Dashboard:**
   - Go to: https://us.cloud.langfuse.com → Settings → Models
   - Click "Add Model"
   - Model ID: `gemini-2.0-flash-exp`
   - Input cost: `$0.00001875` per 1K tokens
   - Output cost: `$0.000075` per 1K tokens
   - Save

2. **Token Usage Auto-Tracked:**
   - Code automatically captures token counts from Gemini API
   - Sends to LangFuse with model name
   - Costs calculated automatically based on pricing configuration

**View in Dashboard:**
- Traces: See all RAG pipeline executions
- Model costs: View total spend and per-query costs
- Scores: Add manual feedback scores or use API

### Datadog APM

Application performance monitoring:
- API endpoint latency and throughput
- Error rates and stack traces
- Custom tags for queries and operations
- Service dependencies and traces
- Resource utilization metrics

## 🧪 Testing

### Using cURL

**JSON Response:**
```bash
curl -X POST "http://localhost:8000/api/v1/rag-travel-assistant" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are visa requirements for Indians traveling to Japan?",
    "top_k": 3,
    "return_sources": true
  }'
```

**Plain Text Response:**
```bash
curl -X POST "http://localhost:8000/api/v1/rag-travel-assistant/text" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "Tell me about Singapore visa requirements",
    "top_k": 2
  }'
```

**Health Check:**
```bash
curl http://localhost:8000/api/v1/health
```

### Using Swagger UI

Visit http://localhost:8000/docs for interactive API documentation where you can:
- Test all endpoints directly from the browser
- See request/response schemas
- View example requests
- Download OpenAPI specification

## 📝 Logging

Logs are written to:
- **Console**: INFO level and above
- **logs/app.log**: All logs with rotation (10MB, 5 backups)
- **logs/error.log**: ERROR level only

## 🔧 Configuration

Key settings in `app/config.py`:

```python
# Qdrant
QDRANT_URL=http://localhost:6333
QDRANT_COLLECTION_NAME=travel_destinations

# Gemini
GEMINI_MODEL=gemini-2.0-flash-exp
GEMINI_TEMPERATURE=0.7
GEMINI_MAX_TOKENS=2048

# RAG
EMBEDDING_MODEL=sentence-transformers/all-MiniLM-L6-v2
TOP_K_RESULTS=5

# API
APP_HOST=0.0.0.0
APP_PORT=8000
LOG_LEVEL=INFO
```

## 🎯 Assignment Tasks Completion

### Task 1: Qdrant Setup with Hybrid Search ✅
- ✅ Collection created with dense vectors (384 dimensions)
- ✅ Sparse vectors configured for keyword search
- ✅ 15 travel destination documents loaded
- ✅ Hybrid search configuration verified
- **Location**: `app/rag/vector_store.py`, `scripts/ingest_data.py`

### Task 2: Hybrid Search Implementation ✅
- ✅ Semantic search using dense vectors (sentence-transformers)
- ✅ Keyword search using sparse vectors with TF weighting
- ✅ Country name boosting (3x weight) for accuracy
- ✅ RRF (Reciprocal Rank Fusion) for result combination
- ✅ Minimum score threshold (0.45) to filter irrelevant results
- ✅ Semantic fallback when hybrid search fails
- ✅ Accurate retrieval verified (Japan query returns only Japan)
- **Location**: `app/rag/retriever.py`

### Task 3: RAG Pipeline with Gemini ✅
- ✅ Hybrid retrieval integrated
- ✅ Context formatting with document metadata
- ✅ Prompt engineering for travel-specific answers
- ✅ Gemini 2.0 Flash integration
- ✅ Token usage tracking for cost monitoring
- ✅ Simple, emoji-enhanced response format
- ✅ Error handling with graceful fallbacks
- ✅ High-quality, accurate answers
- **Location**: `app/rag/pipeline.py`

### Task 4: LangFuse Integration ✅
- ✅ LangFuse client initialized with credentials
- ✅ `@observe` decorators on all RAG operations
- ✅ Retrieval traces (`hybrid_retrieval`)
- ✅ Generation traces (`llm_generation`) with token usage
- ✅ Pipeline-level traces (`rag_pipeline`)
- ✅ Model name and usage metadata sent to LangFuse
- ✅ Traces visible in LangFuse dashboard
- ✅ Cost tracking configured (requires model pricing setup)
- **Location**: `app/observability/langfuse.py`, `app/rag/pipeline.py`

### Task 5: LangGraph Workflow ✅
- ✅ StateGraph defined with proper state management
- ✅ 5 nodes implemented:
  - Input node (validation)
  - Retrieval node (hybrid search with tracing)
  - Generation node (LLM with tracing)
  - Error node (error handling)
  - Output node (formatting)
- ✅ Conditional routing based on state
- ✅ RAG pipeline integrated in generation node
- ✅ Async execution support
- **Location**: `app/graph/state.py`, `app/graph/nodes.py`, `app/graph/travel_assistant.py`

### Task 6: FastAPI Endpoint ✅
- ✅ `/api/v1/rag-travel-assistant` endpoint (JSON response)
- ✅ `/api/v1/rag-travel-assistant/text` endpoint (plain text response)
- ✅ LangGraph workflow integrated
- ✅ Async route handler
- ✅ Request validation with Pydantic
- ✅ Error handling
- ✅ Health check endpoint
- ✅ Swagger UI documentation
- ✅ ReDoc documentation
- **Location**: `app/api/routes.py`, `main.py`

## 📈 Additional Features Implemented

### Enhanced User Experience
- ✅ Dual response formats (JSON and plain text)
- ✅ Emoji-enhanced responses for better readability
- ✅ Structured format with clear sections
- ✅ Simple, friendly language (max 300 words)

### Production-Ready Features
- ✅ Comprehensive logging (file + console)
- ✅ Environment-based configuration
- ✅ Health check endpoints
- ✅ Error tracking and handling
- ✅ Token usage monitoring
- ✅ Cost tracking integration

### Search Accuracy Improvements
- ✅ Country name keyword boosting
- ✅ Score threshold filtering
- ✅ Semantic fallback mechanism
- ✅ Query validation

## 🚨 Troubleshooting

**Qdrant connection error:**
- Ensure Qdrant is running: `docker ps`
- Check QDRANT_URL in `.env`

**No results returned:**
- Run data ingestion: `uv run python scripts/ingest_data.py`
- Check collection: `GET /api/v1/collection-info`

**Gemini API errors:**
- Verify GEMINI_API_KEY is set correctly
- Check API quota and billing

## 📦 Repository

**GitHub**: [https://github.com/chittivijay2003/travel-assistant-rag-datadog](https://github.com/chittivijay2003/travel-assistant-rag-datadog)

Clone the repository:
```bash
git clone https://github.com/chittivijay2003/travel-assistant-rag-datadog.git
cd travel-assistant-rag-datadog
```

## 📄 License

MIT

## 👥 Author

**Chitti Vijay**

Developed as part of GenAI Developer Assignment
