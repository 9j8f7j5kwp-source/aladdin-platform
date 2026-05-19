# AI Copilot Service

Intelligent AI assistant for investment professionals using LLMs and RAG.

## Features

- **Natural Language Queries**: Ask questions about portfolios, risks, and markets
- **Investment Analysis**: AI-powered insights and recommendations
- **Document Understanding**: Analyze regulatory documents and research reports
- **Risk Scenarios**: Simulate "what-if" risk scenarios
- **Conversational Memory**: Context-aware multi-turn conversations

## Technology Stack

- **Framework**: FastAPI (Python 3.11+)
- **LLM**: OpenAI GPT-4 / Azure OpenAI
- **Orchestration**: LangChain
- **Vector DB**: Pinecone (for RAG)
- **Cache**: Redis
- **Database**: PostgreSQL

## API Endpoints

### Chat
```
POST /api/v1/chat/completions
Body: {"message": "Analyze my portfolio risk", "session_id": "uuid"}
```

### Portfolio Analysis
```
POST /api/v1/analyze/portfolio
Body: {"portfolio_id": "PORT123"}
```

### Risk Analysis
```
POST /api/v1/analyze/risk
Body: {"portfolio_id": "PORT123", "scenario": "market_crash"}
```

## Environment Variables

```bash
OPENAI_API_KEY=sk-...
PINECONE_API_KEY=...
PINECONE_ENVIRONMENT=us-west1-gcp
REDIS_URL=redis://localhost:6379
DATABASE_URL=postgresql://user:pass@localhost:5432/copilot
```

## Running Locally

```bash
pip install -r requirements.txt
uvicorn main:app --reload --port 8080
```

## Docker

```bash
docker build -t ai-copilot-service .
docker run -p 8080:8080 ai-copilot-service
```
