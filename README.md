# Aladdin Platform

> Open-source Investment Management Platform — full analog of BlackRock Aladdin

[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)
[![Build Status](https://img.shields.io/badge/build-passing-brightgreen.svg)](#)

## 🎯 Vision

Aladdin Platform is an open-source, enterprise-grade investment management system inspired by BlackRock's Aladdin. It provides a complete front-to-back solution covering portfolio management, risk analytics, order execution, compliance, and AI-powered insights.

**Core Principles:**
- **Multi-language architecture** optimized for each domain (Java, C++, Python, Julia, TypeScript)
- **Microservices-based** design with event-driven architecture
- **Production-ready** with Kubernetes orchestration, observability, and CI/CD
- **Extensible** plugin model for custom analytics and integrations
- **AI-native** with built-in Copilot assistant for investment workflows

---

## 📐 Architecture Overview

### Layered Architecture

```
┌──────────────────────────────────────────────────────┐
│         Client & UI Layer (TypeScript/React)          │
│  Portfolio Dashboards │ Risk Analytics │ Trading Desk │
└──────────────────────────────────────────────────────┘
                           ↓↓
┌──────────────────────────────────────────────────────┐
│      API & Integration Layer (REST/gRPC/WebSocket)   │
│  API Gateway │ FIX Gateways │ SWIFT │ Data Feeds     │
└──────────────────────────────────────────────────────┘
                           ↓↓
┌──────────────────────────────────────────────────────┐
│              Domain Services Layer                    │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│ │Portfolio │ │  Risk    │ │   OMS    │ │Compliance│ │
│ │ Service  │ │ Engine   │ │ Service  │ │ Service  │ │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│ │ Market   │ │Reference │ │Reporting │ │   AI     │ │
│ │   Data   │ │   Data   │ │ Service  │ │ Copilot  │ │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘ │
└──────────────────────────────────────────────────────┘
                           ↓↓
┌──────────────────────────────────────────────────────┐
│      Messaging & Eventing (Kafka Event Bus)          │
│  Trades │ Positions │ Market Data │ Risk Events      │
└──────────────────────────────────────────────────────┘
                           ↓↓
┌──────────────────────────────────────────────────────┐
│        Data & Analytics Layer                         │
│  Data Lake │ Time-Series DB │ Analytics Engine       │
└──────────────────────────────────────────────────────┘
                           ↓↓
┌──────────────────────────────────────────────────────┐
│     Infrastructure & Platform (Kubernetes)            │
│  Service Mesh │ Observability │ CI/CD │ Secrets      │
└──────────────────────────────────────────────────────┘
```

---

## 🏗️ Core Domain Services

### 1. Portfolio Service (Java/Spring Boot)
**Purpose:** Investment Book of Records (IBOR) — single source of truth for positions

**Capabilities:**
- Multi-level portfolio hierarchy (Firm → Fund → Strategy → Account)
- Real-time position tracking with T+0/T+1/T+N settlement
- Cash management and FX exposure
- Corporate actions processing (splits, dividends, M&A)
- Performance attribution (Brinson-Fachler, factor-based)

**Tech Stack:**
- Java 21, Spring Boot 3.x, Spring Data JPA
- PostgreSQL (transactional), Redis (cache)
- gRPC for inter-service communication

**Key APIs:**
```
POST   /api/v1/portfolios
GET    /api/v1/portfolios/{id}/positions
GET    /api/v1/portfolios/{id}/performance
POST   /api/v1/transactions
```

---

### 2. Risk Engine (C++/Julia)
**Purpose:** Real-time and batch risk analytics

**Capabilities:**
- **Market Risk:** VaR (Historical, Monte Carlo, Parametric), Expected Shortfall (CVaR)
- **Stress Testing:** Historical scenarios (2008, COVID), custom shocks
- **Factor Risk:** Multi-factor attribution, PCA-based factor extraction
- **Greeks Calculation:** Delta, Gamma, Vega, Theta for derivatives
- **Sensitivity Analysis:** DV01, convexity, key rate durations

**Tech Stack:**
- **C++20** for performance-critical pricing kernels (QuantLib integration)
- **Julia 1.10+** for quant models (DifferentialEquations.jl, Optimization.jl)
- **gRPC server** for API exposure
- **GPU acceleration** (CUDA) for Monte Carlo simulations

**Architecture:**
```
Risk Calculation Grid (Distributed)
├── Pricing Workers (C++ w/ QuantLib)
├── Scenario Engine (Julia)
├── Aggregation Service (Java)
└── Cache Layer (Redis/Hazelcast)
```

---

### 3. OMS Service (Java + QuickFIX/J)
**Purpose:** Order Management & Execution

**Capabilities:**
- Order lifecycle: Create → Route → Execute → Allocate → Settle
- Multi-destination routing (smart order routing)
- FIX 4.4/5.0 connectivity to brokers and exchanges
- Execution algos: TWAP, VWAP, POV, Implementation Shortfall
- Post-trade allocation to multiple accounts

**Tech Stack:**
- Java 21, Spring Boot, QuickFIX/J
- PostgreSQL (order book), Kafka (execution events)
- FIX Session Management with automatic reconnect

**FIX Integration:**
```
Broker Connections:
├── Interactive Brokers (FIX 4.4)
├── Bloomberg EMSX (FIX 5.0)
├── Tradeweb (FIX 5.0 SP2)
└── Custom Prime Broker Adapters
```

---

### 4. Market Data Service (Java + Python)
**Purpose:** Unified market data ingestion and normalization

**Capabilities:**
- Real-time streaming (equity, FX, rates, commodities)
- End-of-day batch loading (prices, fundamentals)
- Data normalization (vendor-agnostic format)
- Time-series storage with point-in-time queries
- Tick data replay for backtesting

**Data Providers:**
- Bloomberg (B-PIPE, DLWS)
- Refinitiv (Elektron, Tick History)
- IEX Cloud, Polygon.io (equities)
- Quandl/Nasdaq Data Link (alternative data)

**Tech Stack:**
- **Ingest Layer:** Python 3.11+ (asyncio, aiohttp)
- **Normalization:** Java stream processing (Kafka Streams)
- **Storage:** TimescaleDB (time-series), Parquet files (data lake)

---

### 5. Compliance Service (Java)
**Purpose:** Pre-trade and post-trade compliance checks

**Capabilities:**
- **Pre-trade Rules:**
  - Position limits (% of portfolio, absolute)
  - Restricted securities list
  - Regulatory limits (UCITS 5/10/40 rule)
  - ESG/SRI screening
- **Post-trade Monitoring:**
  - Trade surveillance (market abuse detection)
  - Best execution analysis
  - Regulatory reporting (MiFID II, Dodd-Frank)

**Rule Engine:**
- Drools (business rules engine)
- Custom DSL for compliance rules
- Real-time event processing (Kafka Streams)

---

### 6. AI Copilot Service (Python/LangGraph)
**Purpose:** Natural language assistant for investment workflows

**Capabilities:**
- **Conversational Interface:**
  - "Show me the VaR for portfolio ABC"
  - "What are the top 10 positions in the Tech sector?"
  - "Analyze impact if rates rise 50bps"
- **Tool Calling:** Automatic routing to Portfolio/Risk/OMS APIs
- **Explainability:** Reasoning traces with audit logs

**Tech Stack:**
- Python 3.11+, FastAPI, LangGraph
- LLM: OpenAI GPT-4 / Anthropic Claude / Llama 3
- Vector DB: Pinecone / Weaviate

**Example Interaction:**
```
User: "What's the VaR for my global equity fund?"

Copilot:
1. [Tool: get_portfolio_id] Query: "global equity fund"
   → Result: portfolio_id = "PF-12345"

2. [Tool: calculate_var] Input: {portfolio_id: "PF-12345"}
   → Result: {var_1d: -2.3%, var_10d: -7.1%}

Response: "Your Global Equity Fund has 1-day VaR of -2.3% 
and 10-day VaR of -7.1% at 99% confidence level."
```

---

### 7. Reference Data Service (Java)
**Purpose:** Security master and static data

**Capabilities:**
- Instrument definitions (ISIN, CUSIP, SEDOL)
- Issuer/counterparty data
- Market calendars and holidays
- Currency pairs and FX conversion

**Data Sources:**
- Bloomberg BSYM, Refinitiv RIC
- OpenFIGI, LEI registry

---

### 8. Reporting Service (Python/Scala)
**Purpose:** Client and regulatory reporting

**Capabilities:**
- Client reports (holdings, performance tearsheets)
- Regulatory reports (Form PF, AIFMD Annex IV, MiFID II)

**Tech Stack:**
- Python (Pandas, ReportLab), Apache Spark (Scala)
- Scheduler: Apache Airflow

---

## 🔗 Messaging & Integration

### Event Bus (Apache Kafka)

**Topics:**
```
market-data.quotes       # Real-time price updates
portfolio.positions      # Position changes
oms.executions           # Trade fills
risk.calculations        # Risk metric updates
compliance.breaches      # Rule violations
```

**Message Format (Protobuf):**
```protobuf
message TradeEvent {
  string trade_id = 1;
  string portfolio_id = 2;
  string symbol = 3;
  Side side = 4;  // BUY/SELL
  double quantity = 5;
  double price = 6;
  google.protobuf.Timestamp execution_time = 7;
}
```

---

## 🗄️ Data & Analytics Layer

### Data Lake Architecture

```
Bronze Layer (Raw):
  ├── market_data_raw/
  ├── trade_raw/
  └── reference_data_raw/

Silver Layer (Cleansed):
  ├── positions_cleaned/
  ├── trades_enriched/
  └── prices_normalized/

Gold Layer (Analytics):
  ├── risk_aggregates/
  ├── performance_metrics/
  └── ml_features/
```

**Storage:**
- Time-series: TimescaleDB (hot), Parquet on S3 (cold)
- Analytics: Databricks Delta Lake
- Document: MongoDB

---

## ⚙️ Technology Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | TypeScript, React 18, Redux Toolkit, D3.js |
| **API Gateway** | Kong / Spring Cloud Gateway |
| **Services (Core)** | Java 21, Spring Boot 3.2, Hibernate |
| **Services (Performance)** | C++20, Rust, Julia 1.10 |
| **Services (AI/ML)** | Python 3.11, FastAPI, LangGraph |
| **Messaging** | Apache Kafka 3.6, Protobuf, gRPC |
| **Databases** | PostgreSQL 16, TimescaleDB, MongoDB |
| **Data Lake** | Snowflake / Databricks Delta Lake |
| **Analytics** | Apache Spark 3.5, Apache Flink |
| **Orchestration** | Kubernetes 1.29, Helm 3 |
| **Service Mesh** | Istio 1.20 |
| **Monitoring** | Prometheus, Grafana, Loki, Jaeger |
| **CI/CD** | GitHub Actions, ArgoCD, Terraform |
| **Security** | Keycloak, Vault, OPA |

---

## 📂 Repository Structure

```
aladdin-platform/
├── services/
│   ├── portfolio-service/       # Java - Portfolio & positions
│   ├── risk-engine/             # C++/Julia - Risk calculations
│   ├── oms-service/             # Java - Order management
│   ├── market-data-service/     # Java/Python - Market data
│   ├── compliance-service/      # Java - Compliance rules
│   ├── reference-data-service/  # Java - Security master
│   ├── reporting-service/       # Python/Scala - Reports
│   ├── ai-copilot-service/      # Python - AI assistant
│   └── identity-service/        # Keycloak config
├── frontend/
│   ├── web-portal/              # TypeScript/React
│   └── desktop-client/          # Electron (optional)
├── messaging/
│   ├── protobuf-schemas/        # Message definitions
│   └── sdk/                     # Generated clients
├── infra/
│   ├── kubernetes/              # K8s manifests, Helm charts
│   ├── docker-compose/          # Local development
│   ├── terraform/               # Cloud infrastructure
│   └── kafka/                   # Kafka cluster config
├── libs/
│   ├── common-java/             # Shared Java libraries
│   ├── common-python/           # Shared Python utilities
│   └── quant-models/            # Julia/C++ pricing libraries
├── data/
│   ├── airflow-dags/            # Data pipeline DAGs
│   └── sql-migrations/          # Database schemas (Flyway)
├── docs/
│   ├── architecture/            # ADRs, diagrams
│   ├── api/                     # OpenAPI/gRPC specs
│   └── runbooks/                # Operations guides
└── tests/
    ├── integration/             # End-to-end tests
    ├── performance/             # Load tests (JMeter, K6)
    └── chaos/                   # Chaos engineering
```

---

## 🛣️ Development Roadmap

### Phase 0: Foundation (Months 1-2)
✅ Repository setup, CI/CD pipelines  
✅ Kubernetes cluster provisioning  
✅ Core data models (Protobuf schemas)  
✅ PostgreSQL + TimescaleDB deployment  
✅ Kafka cluster setup  

### Phase 1: IBOR + Market Data (Months 3-5)
🚧 Portfolio Service MVP  
🚧 Market Data Service (1 provider)  
🚧 Reference Data Service  
🚧 Basic web UI (portfolio view)  

### Phase 2: Risk Engine (Months 6-8)
⬜ VaR calculation (Historical + Parametric)  
⬜ Stress testing framework  
⬜ Factor risk attribution  
⬜ Risk UI dashboards  

### Phase 3: OMS & Compliance (Months 9-11)
⬜ OMS Service with FIX connectivity  
⬜ Smart order routing  
⬜ Compliance rule engine  
⬜ Pre-trade checks integration  

### Phase 4: Reporting & AI (Months 12-14)
⬜ Reporting engine  
⬜ AI Copilot with LangGraph  
⬜ NL query interface  
⬜ Advanced analytics (ML models)  

### Phase 5: Production Hardening (Months 15-18)
⬜ Multi-tenant architecture  
⬜ Disaster recovery & HA  
⬜ Security audit  
⬜ Performance optimization  
⬜ Open-source release v1.0  

---

## 🏁 Quick Start

### Prerequisites
- Docker 24+ & Docker Compose
- Kubernetes cluster (minikube/kind for local)
- Java 21, Python 3.11, Node.js 20

### Local Development

```bash
# Clone repository
git clone https://github.com/9j8f7j5kwp-source/aladdin-platform.git
cd aladdin-platform

# Start infrastructure
docker-compose -f infra/docker-compose/infra.yml up -d

# Start core services
docker-compose -f infra/docker-compose/services.yml up -d

# Access UI
open http://localhost:3000
```

### Kubernetes Deployment

```bash
helm repo add aladdin https://aladdin-platform.github.io/charts
helm install aladdin aladdin/aladdin-platform \
  --namespace aladdin-core \
  --create-namespace
```

---

## 🤝 Contributing

We welcome contributions! Areas needing help:
- FIX protocol adapters for additional brokers
- Market data provider integrations
- Quant models (pricing engines, risk models)
- UI/UX improvements
- Documentation

---

## 📄 License

Apache License 2.0 — See [LICENSE](LICENSE)

---

## 🙏 Acknowledgments

Inspired by:
- BlackRock Aladdin architecture
- SimCorp Dimension, Charles River IMS
- Open-source: QuantLib, Apache Kafka, Kubernetes

---

**Built with ❤️ for the investment management community**
