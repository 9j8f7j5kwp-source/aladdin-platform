# Portfolio Service

> Investment Book of Records (IBOR) — Single source of truth for positions

## Overview

Portfolio Service is the core accounting engine managing:
- Portfolio hierarchy (Firm → Fund → Strategy → Account)
- Position tracking (real-time T+0/T+1/T+N)
- Cash management and FX exposure
- Corporate actions processing
- Performance attribution

## Tech Stack

- **Language:** Java 21
- **Framework:** Spring Boot 3.2.x
- **Database:** PostgreSQL 16 (transactional data)
- **Cache:** Redis 7.x
- **API:** REST + gRPC

## Architecture

```
PortfolioService
├── API Layer (REST Controllers)
├── Domain Layer (Portfolio, Position, Transaction)
├── Application Layer (Use Cases)
├── Infrastructure Layer (PostgreSQL, Redis, Kafka)
└── gRPC Server (Inter-service communication)
```

## Key Entities

### Portfolio
```java
@Entity
public class Portfolio {
    private UUID id;
    private String name;
    private PortfolioType type;  // FUND, ACCOUNT, etc.
    private UUID parentId;
    private String baseCurrency;
    private LocalDate inceptionDate;
}
```

### Position
```java
@Entity
public class Position {
    private UUID id;
    private UUID portfolioId;
    private String symbol;
    private BigDecimal quantity;
    private BigDecimal averageCost;
    private LocalDate settlementDate;
}
```

## APIs

### REST Endpoints

```
POST   /api/v1/portfolios
GET    /api/v1/portfolios/{id}
GET    /api/v1/portfolios/{id}/positions
GET    /api/v1/portfolios/{id}/performance
POST   /api/v1/transactions
GET    /api/v1/portfolios/{id}/cash-balances
```

### gRPC Services

```protobuf
service PortfolioService {
  rpc GetPortfolio(PortfolioRequest) returns (PortfolioResponse);
  rpc GetPositions(PositionsRequest) returns (PositionsResponse);
  rpc RecordTransaction(TransactionRequest) returns (TransactionResponse);
}
```

## Event Publishing

Publishes to Kafka topics:
- `portfolio.positions` — Position updates
- `portfolio.transactions` — New transactions
- `portfolio.cash` — Cash movements

## Setup

### Prerequisites
- Java 21
- Docker & Docker Compose
- PostgreSQL 16
- Kafka

### Run Locally

```bash
# Start dependencies
docker-compose up -d postgres redis kafka

# Run service
./gradlew bootRun

# Or with Maven
./mvnw spring-boot:run
```

### Configuration

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/portfolio
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: validate
  kafka:
    bootstrap-servers: localhost:9092
```

## Testing

```bash
# Unit tests
./gradlew test

# Integration tests
./gradlew integrationTest

# Load tests
k6 run tests/load/portfolio-load.js
```

## Metrics

- `portfolio.api.requests` — API request count
- `portfolio.positions.count` — Total positions tracked
- `portfolio.transactions.rate` — Transactions/sec
- `portfolio.db.latency` — Database query latency

## Roadmap

- [ ] Multi-currency support
- [ ] Real-time P&L calculation
- [ ] Advanced attribution models
- [ ] GraphQL API
- [ ] Event sourcing migration
