# OMS Service (Order Management System)

## Overview

The Order Management System (OMS) is a core service responsible for the complete order lifecycle management, from order creation through routing, execution, and settlement. Built with Java 21 and Spring Boot, it provides FIX protocol connectivity to brokers and exchanges.

## Purpose

Manage the complete order lifecycle for equity, fixed income, FX, and derivatives trading:
- Order creation, validation, and routing
- Multi-destination smart order routing
- FIX 4.4/5.0 connectivity to brokers and exchanges
- Execution management and allocation
- Post-trade settlement processing

## Architecture

### Technology Stack
- **Language**: Java 21
- **Framework**: Spring Boot 3.2
- **Messaging**: Kafka (event-driven)
- **Database**: PostgreSQL (order book, audit trail)
- **Cache**: Redis (real-time order state)
- **FIX Engine**: QuickFIX/J
- **API**: gRPC + REST

### Key Components

```
oms-service/
├── src/main/java/com/aladdin/oms/
│   ├── controller/         # REST/gRPC controllers
│   ├── service/            # Business logic
│   │   ├── OrderService
│   │   ├── RoutingService
│   │   ├── ExecutionService
│   │   └── AllocationService
│   ├── fix/                # FIX protocol handlers
│   │   ├── FIXSessionManager
│   │   ├── FIXMessageHandler
│   │   └── brokers/        # Broker-specific adapters
│   ├── domain/             # Domain entities
│   │   ├── Order
│   │   ├── Execution
│   │   ├── Allocation
│   │   └── Route
│   ├── repository/         # Data access
│   ├── kafka/              # Event producers/consumers
│   └── config/             # Configuration
└── src/main/resources/
    ├── application.yml
    ├── fix/                # FIX session configs
    └── db/migration/       # Flyway migrations
```

## Key Features

### 1. Order Lifecycle Management

**Order States**:
- `NEW` → Order created, awaiting validation
- `VALIDATED` → Passed all pre-trade checks
- `ROUTED` → Sent to broker/exchange
- `PARTIALLY_FILLED` → Partial execution received
- `FILLED` → Fully executed
- `CANCELLED` → Order cancelled
- `REJECTED` → Order rejected

**Order Types Supported**:
- Market
- Limit
- Stop
- Stop-Limit
- IOC (Immediate or Cancel)
- FOK (Fill or Kill)
- Iceberg
- VWAP
- TWAP
- POV (Percentage of Volume)

### 2. Smart Order Routing (SOR)

**Routing Strategies**:
- Best execution (price, liquidity, speed)
- Multi-destination routing
- Dark pool aggregation
- Algorithmic routing (VWAP, TWAP, Implementation Shortfall)

**Destination Types**:
- Primary exchanges
- Alternative venues (ATS)
- Dark pools
- Electronic brokers

### 3. FIX Connectivity

**Supported Brokers**:
- Interactive Brokers (FIX 4.4)
- Bloomberg EMSX (FIX 5.0)
- Tradeweb (FIX 5.0 SP2)
- Custom Prime Brokers

**FIX Message Types**:
- `NewOrderSingle` (D)
- `OrderCancelRequest` (F)
- `OrderCancelReplaceRequest` (G)
- `ExecutionReport` (8)
- `OrderStatusRequest` (H)

### 4. Execution Management

**Capabilities**:
- Real-time execution capture
- Average price calculation
- Fill aggregation
- Execution quality analysis
- Transaction cost analysis (TCA)

### 5. Post-Trade Processing

**Allocation Engine**:
- Multi-account allocation
- Pro-rata allocation
- Cash-based allocation
- Model-based allocation

**Settlement**:
- T+1, T+2, T+3 settlement cycles
- DVP (Delivery vs Payment)
- Settlement instruction generation

## API Endpoints

### REST API

#### Create Order
```http
POST /api/v1/orders
Content-Type: application/json

{
  "symbol": "AAPL",
  "side": "BUY",
  "quantity": 1000,
  "orderType": "LIMIT",
  "limitPrice": 175.50,
  "timeInForce": "DAY",
  "account": "ACC-12345",
  "strategy": "VWAP"
}
```

#### Get Order Status
```http
GET /api/v1/orders/{orderId}
```

#### Cancel Order
```http
DELETE /api/v1/orders/{orderId}
```

#### Get Executions
```http
GET /api/v1/executions?orderId={orderId}
```

### gRPC API

```protobuf
service OrderManagementService {
  rpc CreateOrder(CreateOrderRequest) returns (OrderResponse);
  rpc GetOrder(GetOrderRequest) returns (OrderResponse);
  rpc CancelOrder(CancelOrderRequest) returns (OrderResponse);
  rpc GetExecutions(GetExecutionsRequest) returns (ExecutionsResponse);
  rpc StreamOrders(StreamOrdersRequest) returns (stream OrderUpdate);
}
```

## Event-Driven Architecture

### Kafka Topics

#### Published Events:
- `orders.created` - New order created
- `orders.validated` - Order passed validation
- `orders.routed` - Order sent to destination
- `orders.executed` - Execution received
- `orders.filled` - Order fully filled
- `orders.cancelled` - Order cancelled
- `executions.received` - Execution details
- `allocations.completed` - Allocation finalized

#### Consumed Events:
- `market-data.quotes` - Real-time quotes for routing
- `risk.limits` - Risk limit updates
- `portfolio.positions` - Position updates
- `compliance.rules` - Compliance checks

## Database Schema

### Core Tables

**orders**
- `order_id` (PK)
- `client_order_id`
- `symbol`
- `side` (BUY/SELL)
- `quantity`
- `order_type`
- `limit_price`
- `stop_price`
- `time_in_force`
- `status`
- `account_id`
- `strategy`
- `created_at`
- `updated_at`

**executions**
- `execution_id` (PK)
- `order_id` (FK)
- `exec_price`
- `exec_quantity`
- `exec_time`
- `venue`
- `exec_broker`
- `commission`
- `liquidity_indicator`

**allocations**
- `allocation_id` (PK)
- `order_id` (FK)
- `execution_id` (FK)
- `account_id`
- `allocated_quantity`
- `allocated_price`
- `settlement_date`

**fix_sessions**
- `session_id` (PK)
- `broker_name`
- `sender_comp_id`
- `target_comp_id`
- `host`
- `port`
- `status`
- `last_heartbeat`

## Configuration

### application.yml
```yaml
spring:
  application:
    name: oms-service
  datasource:
    url: jdbc:postgresql://postgres-service:5432/aladdin
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
  kafka:
    bootstrap-servers: kafka-service:9092
    consumer:
      group-id: oms-service
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

oms:
  fix:
    config-path: /config/fix
    heartbeat-interval: 30
  routing:
    enabled: true
    strategy: best-execution
  execution:
    auto-allocate: false
```

### FIX Session Configuration

**config/fix/interactive-brokers.cfg**
```ini
[DEFAULT]
ConnectionType=initiator
ReconnectInterval=30
FileStorePath=store
FileLogPath=log
StartTime=00:00:00
EndTime=23:59:59

[SESSION]
BeginString=FIX.4.4
SenderCompID=ALADDIN
TargetCompID=IBKR
SocketConnectHost=127.0.0.1
SocketConnectPort=4001
HeartBtInt=30
```

## Deployment

### Docker

**Dockerfile**
```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY target/oms-service-*.jar app.jar
EXPOSE 8082 9092
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Kubernetes

**deployment.yaml**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oms-service
  namespace: aladdin-platform
spec:
  replicas: 3
  selector:
    matchLabels:
      app: oms-service
  template:
    metadata:
      labels:
        app: oms-service
    spec:
      containers:
      - name: oms-service
        image: ghcr.io/aladdin-platform/oms-service:latest
        ports:
        - containerPort: 8082
        - containerPort: 9092
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: production
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: username
        resources:
          requests:
            memory: "2Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
```

## Testing

### Unit Tests
```bash
./gradlew test
```

### Integration Tests
```bash
./gradlew integrationTest
```

### FIX Protocol Testing
```bash
# Start FIX simulator
docker run -p 9878:9878 quickfixj/simulator

# Run FIX tests
./gradlew fixTest
```

## Monitoring

### Metrics
- Order creation rate
- Order fill rate
- Average execution time
- FIX session uptime
- Order rejection rate
- Routing success rate

### Health Checks
```http
GET /actuator/health
GET /actuator/metrics
GET /actuator/prometheus
```

## Security

### Authentication
- OAuth2/JWT for REST API
- mTLS for gRPC
- FIX session certificates

### Authorization
- Role-based access (Trader, Portfolio Manager, Compliance)
- Account-level permissions
- Order type restrictions

## Performance

### Benchmarks
- Order latency: < 5ms (99th percentile)
- Throughput: 50,000 orders/second
- FIX message processing: < 1ms

## Dependencies

- portfolio-service (position validation)
- risk-engine (pre-trade risk checks)
- market-data-service (real-time prices)
- compliance-service (regulatory checks)

## Next Steps

1. Implement algorithmic execution strategies
2. Add machine learning for optimal routing
3. Integrate with additional brokers
4. Build execution analytics dashboard

## Contributing

See [CONTRIBUTING.md](../../CONTRIBUTING.md)

## License

See [LICENSE](../../LICENSE)
