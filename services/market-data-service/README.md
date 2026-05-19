# Market Data Service

## Overview

The Market Data Service provides real-time and historical market data for all asset classes. Built with C++ for ultra-low latency performance and Python for analytics, it aggregates data from multiple sources and distributes it to all platform services.

## Purpose

Provide unified market data access for the entire platform:
- Real-time Level 1/2/3 market data (quotes, depth, trades)
- Historical tick, bar, and daily data
- Corporate actions and reference data
- Market indices and benchmarks
- Alternative data (sentiment, news, ESG)

## Architecture

### Technology Stack
- **Core Engine**: C++20 (ultra-low latency)
- **Analytics**: Python 3.11 + NumPy/Pandas
- **Messaging**: Kafka + WebSockets
- **Storage**: 
  - TimescaleDB (time-series data)
  - Redis (real-time cache)
  - S3/MinIO (historical archive)
- **API**: gRPC (streaming) + REST

### Key Components

```
market-data-service/
├── cpp/                     # C++ core engine
│   ├── src/
│   │   ├── connectors/     # Data source connectors
│   │   │   ├── IEXConnector
│   │   │   ├── PolygonConnector
│   │   │   ├── AlphaVantageConnector
│   │   │   └── BloombergConnector
│   │   ├── aggregator/     # Data normalization
│   │   ├── publisher/      # Kafka publisher
│   │   ├── cache/          # Redis cache manager
│   │   └── grpc/           # gRPC server
│   └── CMakeLists.txt
├── python/                 # Python analytics
│   ├── analytics/
│   │   ├── technical_indicators.py
│   │   ├── volatility_models.py
│   │   └── correlation_engine.py
│   ├── historical/
│   │   └── data_loader.py
│   └── rest_api/
│       └── app.py          # FastAPI server
└── proto/                  # Protocol buffers
    └── market_data.proto
```

## Key Features

### 1. Real-Time Data Streaming

**Data Types**:
- **Level 1**: Best bid/ask, last trade
- **Level 2**: Market depth (order book)
- **Level 3**: Full order book with order IDs
- **Trades**: Executed trades stream
- **OHLCV**: Aggregated bars (1s, 1m, 5m, 1h, 1d)

**Latency**:
- Market data ingestion: < 100μs
- Cache lookup: < 10μs
- gRPC streaming: < 500μs
- WebSocket delivery: < 1ms

### 2. Data Sources

**Equity Market Data**:
- IEX Cloud (US equities, free tier)
- Polygon.io (US equities, options, forex)
- Alpha Vantage (global equities, FX, crypto)
- Bloomberg Terminal API (enterprise)

**Fixed Income**:
- TRACE (corporate bonds)
- Bloomberg Fixed Income
- ICE Data Services

**Derivatives**:
- CME Market Data
- CBOE LiveVol (options)
- ICE Futures

**Crypto**:
- Coinbase Pro
- Binance
- Kraken

**Alternative Data**:
- Twitter Sentiment (via Twitter API)
- News feeds (Bloomberg, Reuters)
- ESG ratings (MSCI, Sustainalytics)

### 3. Data Normalization

All data normalized to common schema:
```cpp
struct Quote {
    std::string symbol;
    int64_t timestamp_ns;
    double bid_price;
    double ask_price;
    int64_t bid_size;
    int64_t ask_size;
    std::string exchange;
};

struct Trade {
    std::string symbol;
    int64_t timestamp_ns;
    double price;
    int64_t volume;
    std::string exchange;
    char side; // 'B' or 'S'
};
```

### 4. Historical Data

**Storage Tiers**:
- **Hot**: Last 7 days in TimescaleDB
- **Warm**: Last 12 months in compressed TimescaleDB
- **Cold**: >12 months in S3/MinIO (Parquet format)

**Query Performance**:
- Hot data: < 10ms
- Warm data: < 100ms
- Cold data: < 5s

### 5. Analytics Engine (Python)

**Technical Indicators**:
- Moving Averages (SMA, EMA, WMA)
- RSI, MACD, Bollinger Bands
- ATR, ADX, Stochastic

**Volatility Models**:
- Historical volatility
- GARCH models
- Implied volatility surface

**Correlation Analysis**:
- Pearson/Spearman correlation
- Rolling correlation windows
- Principal Component Analysis (PCA)

## API Endpoints

### gRPC Streaming API

```protobuf
service MarketDataService {
  // Real-time quote stream
  rpc StreamQuotes(QuoteRequest) returns (stream Quote);
  
  // Real-time trades stream
  rpc StreamTrades(TradeRequest) returns (stream Trade);
  
  // Market depth stream
  rpc StreamDepth(DepthRequest) returns (stream MarketDepth);
  
  // Historical data
  rpc GetHistoricalBars(HistoricalRequest) returns (BarsResponse);
}

message QuoteRequest {
  repeated string symbols = 1;
  bool level2 = 2;
}
```

### REST API (Python/FastAPI)

#### Get Latest Quote
```http
GET /api/v1/quotes/{symbol}

Response:
{
  "symbol": "AAPL",
  "bid": 175.45,
  "ask": 175.47,
  "bidSize": 100,
  "askSize": 200,
  "timestamp": "2026-05-19T20:00:00Z"
}
```

#### Get Historical Bars
```http
GET /api/v1/bars/{symbol}?from=2026-01-01&to=2026-05-19&interval=1d

Response:
{
  "symbol": "AAPL",
  "bars": [
    {
      "timestamp": "2026-01-01T00:00:00Z",
      "open": 172.50,
      "high": 174.20,
      "low": 171.80,
      "close": 173.90,
      "volume": 65432100
    }
  ]
}
```

#### Calculate Indicators
```http
POST /api/v1/indicators
Content-Type: application/json

{
  "symbol": "AAPL",
  "indicators": ["SMA_20", "RSI_14", "BBANDS"],
  "from": "2026-01-01",
  "to": "2026-05-19"
}
```

### WebSocket API

```javascript
ws://market-data-service:8084/ws

// Subscribe to quotes
{
  "action": "subscribe",
  "channel": "quotes",
  "symbols": ["AAPL", "MSFT", "GOOGL"]
}

// Receive updates
{
  "channel": "quotes",
  "data": {
    "symbol": "AAPL",
    "bid": 175.45,
    "ask": 175.47,
    "timestamp": 1716148800000
  }
}
```

## Event-Driven Architecture

### Kafka Topics

#### Published Events:
- `market-data.quotes.{symbol}` - Real-time quotes
- `market-data.trades.{symbol}` - Trade executions
- `market-data.depth.{symbol}` - Market depth updates
- `market-data.bars.1m` - 1-minute OHLCV bars
- `market-data.corporate-actions` - Dividends, splits, etc.
- `market-data.reference` - Symbol reference data

#### Consumed Events:
- `portfolio.subscriptions` - Subscribe to symbols
- `oms.orders` - Order flow for execution analytics

## Database Schema

### TimescaleDB (Time-Series)

**quotes** (hypertable)
```sql
CREATE TABLE quotes (
  time TIMESTAMPTZ NOT NULL,
  symbol TEXT NOT NULL,
  bid DOUBLE PRECISION,
  ask DOUBLE PRECISION,
  bid_size INTEGER,
  ask_size INTEGER,
  exchange TEXT
);

SELECT create_hypertable('quotes', 'time');
```

**trades** (hypertable)
```sql
CREATE TABLE trades (
  time TIMESTAMPTZ NOT NULL,
  symbol TEXT NOT NULL,
  price DOUBLE PRECISION,
  volume INTEGER,
  exchange TEXT,
  side CHAR(1)
);

SELECT create_hypertable('trades', 'time');
```

### PostgreSQL (Reference Data)

**symbols**
- `symbol` (PK)
- `name`
- `exchange`
- `asset_type`
- `currency`
- `country`
- `sector`
- `industry`

**corporate_actions**
- `action_id` (PK)
- `symbol`
- `action_type` (dividend, split, merger)
- `ex_date`
- `record_date`
- `payment_date`
- `value`

## Configuration

### C++ Configuration (config.yaml)
```yaml
data_sources:
  - name: iex
    url: https://cloud.iexapis.com/stable
    api_key: ${IEX_API_KEY}
    enabled: true
  
  - name: polygon
    url: wss://socket.polygon.io
    api_key: ${POLYGON_API_KEY}
    enabled: true

kafka:
  brokers:
    - kafka-service:9092
  topics:
    quotes: market-data.quotes
    trades: market-data.trades

redis:
  host: redis-service
  port: 6379
  ttl_seconds: 300

grpc:
  port: 50051
  max_connections: 10000
```

### Python Configuration (application.yml)
```yaml
api:
  host: 0.0.0.0
  port: 8084

database:
  timescale:
    host: timescaledb-service
    port: 5432
    database: market_data
  
analytics:
  cache_indicators: true
  parallel_processing: true
  num_workers: 8
```

## Deployment

### Docker (C++ Service)

**Dockerfile**
```dockerfile
FROM ubuntu:22.04 as builder
RUN apt-get update && apt-get install -y \
    build-essential cmake libboost-all-dev \
    librdkafka-dev libhiredis-dev grpc++

WORKDIR /app
COPY cpp/ .
RUN cmake -B build && cmake --build build --config Release

FROM ubuntu:22.04
COPY --from=builder /app/build/market-data-service /usr/local/bin/
EXPOSE 50051
CMD ["market-data-service"]
```

### Docker (Python Service)

**Dockerfile**
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY python/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY python/ .
EXPOSE 8084
CMD ["uvicorn", "rest_api.app:app", "--host", "0.0.0.0", "--port", "8084"]
```

### Kubernetes

**deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: market-data-cpp
  namespace: aladdin-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: market-data-cpp
  template:
    spec:
      containers:
      - name: market-data-cpp
        image: ghcr.io/aladdin-platform/market-data-cpp:latest
        ports:
        - containerPort: 50051
        resources:
          requests:
            memory: "4Gi"
            cpu: "2000m"
          limits:
            memory: "8Gi"
            cpu: "4000m"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: market-data-python
  namespace: aladdin-platform
spec:
  replicas: 2
  template:
    spec:
      containers:
      - name: market-data-python
        image: ghcr.io/aladdin-platform/market-data-python:latest
        ports:
        - containerPort: 8084
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
```

## Performance

### Benchmarks
- **C++ Engine**:
  - Market data ingestion: 100μs
  - Cache writes: 10μs
  - gRPC streaming: 500μs
  - Throughput: 1M messages/second

- **Python Analytics**:
  - Indicator calculation: 50ms (100K bars)
  - Historical query: 10ms (hot), 100ms (warm)

## Monitoring

### Metrics
- Message ingestion rate (per source)
- Cache hit ratio
- gRPC connection count
- WebSocket subscriber count
- Data latency (exchange → platform)
- API response times

### Health Checks
```http
GET /health
GET /metrics
GET /prometheus
```

## Data Quality

### Validation
- Price range checks (outlier detection)
- Volume spike detection
- Missing data detection
- Cross-source validation

### Monitoring
- Data freshness alerts
- Source outage detection
- Quality score tracking

## Security

### API Key Management
- External API keys stored in Kubernetes secrets
- Rate limit enforcement per API
- Circuit breakers for failing sources

### Data Access
- gRPC mTLS authentication
- API key authentication for REST
- Per-symbol entitlements

## Dependencies

- oms-service (consumes market data)
- risk-engine (consumes market data)
- portfolio-service (consumes market data)

## Next Steps

1. Add machine learning for price prediction
2. Implement options volatility surface
3. Add more alternative data sources
4. Build market microstructure analytics

## Contributing

See [CONTRIBUTING.md](../../CONTRIBUTING.md)

## License

See [LICENSE](../../LICENSE)
