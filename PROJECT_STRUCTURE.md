# Aladdin Platform — Complete Project Structure

> Detailed breakdown of all components, files, and configurations

## Complete Directory Tree

```
aladdin-platform/
├── .github/
│   ├── workflows/
│   │   ├── ci-services.yml
│   │   ├── ci-frontend.yml
│   │   ├── ci-infra.yml
│   │   ├── release.yml
│   │   └── security-scan.yml
│   ├── ISSUE_TEMPLATE/
│   │   ├── bug_report.md
│   │   ├── feature_request.md
│   │   └── service_template.md
│   └── dependabot.yml
│
├── services/
│   ├── portfolio-service/          # ✅ Created
│   │   ├── src/
│   │   │   ├── main/
│   │   │   │   ├── java/com/aladdin/portfolio/
│   │   │   │   │   ├── api/
│   │   │   │   │   │   ├── PortfolioController.java
│   │   │   │   │   │   ├── PositionController.java
│   │   │   │   │   │   └── TransactionController.java
│   │   │   │   │   ├── domain/
│   │   │   │   │   │   ├── Portfolio.java
│   │   │   │   │   │   ├── Position.java
│   │   │   │   │   │   ├── Transaction.java
│   │   │   │   │   │   └── CashBalance.java
│   │   │   │   │   ├── application/
│   │   │   │   │   │   ├── PortfolioService.java
│   │   │   │   │   │   ├── PositionService.java
│   │   │   │   │   │   └── PerformanceService.java
│   │   │   │   │   ├── infrastructure/
│   │   │   │   │   │   ├── jpa/
│   │   │   │   │   │   │   ├── PortfolioRepository.java
│   │   │   │   │   │   │   └── PositionRepository.java
│   │   │   │   │   │   ├── kafka/
│   │   │   │   │   │   │   ├── PositionEventPublisher.java
│   │   │   │   │   │   │   └── TransactionEventPublisher.java
│   │   │   │   │   │   └── cache/
│   │   │   │   │   │       └── RedisCacheConfig.java
│   │   │   │   │   └── grpc/
│   │   │   │   │       └── PortfolioGrpcService.java
│   │   │   │   └── resources/
│   │   │   │       ├── application.yml
│   │   │   │       ├── application-dev.yml
│   │   │   │       ├── application-prod.yml
│   │   │   │       └── db/migration/
│   │   │   │           ├── V1__init_portfolio_schema.sql
│   │   │   │           └── V2__add_performance_tables.sql
│   │   │   └── test/
│   │   │       ├── java/
│   │   │       │   └── integration/
│   │   │       │       ├── PortfolioApiTest.java
│   │   │       │       └── PositionServiceTest.java
│   │   │       └── resources/
│   │   │           └── test-data.sql
│   │   ├── build.gradle
│   │   ├── Dockerfile
│   │   ├── docker-compose.yml
│   │   └── README.md
│   │
│   ├── risk-engine/                # ✅ Created
│   │   ├── cpp/
│   │   │   ├── src/
│   │   │   │   ├── pricing/
│   │   │   │   │   ├── BlackScholes.cpp
│   │   │   │   │   ├── MonteCarlo.cpp
│   │   │   │   │   ├── Greeks.cpp
│   │   │   │   │   └── QuantLibWrapper.cpp
│   │   │   │   ├── cuda/
│   │   │   │   │   ├── monte_carlo_kernel.cu
│   │   │   │   │   └── parallel_pricing.cu
│   │   │   │   └── grpc/
│   │   │   │       └── pricing_service.cpp
│   │   │   ├── include/
│   │   │   │   └── aladdin/pricing/
│   │   │   │       ├── types.hpp
│   │   │   │       └── calculator.hpp
│   │   │   ├── CMakeLists.txt
│   │   │   └── Dockerfile.cuda
│   │   ├── julia/
│   │   │   ├── src/
│   │   │   │   ├── RiskEngine.jl
│   │   │   │   ├── var_calculator.jl
│   │   │   │   ├── stress_testing.jl
│   │   │   │   ├── factor_models.jl
│   │   │   │   └── scenario_engine.jl
│   │   │   ├── test/
│   │   │   │   └── runtests.jl
│   │   │   ├── Project.toml
│   │   │   └── Manifest.toml
│   │   ├── java/
│   │   │   ├── src/main/java/
│   │   │   │   └── com/aladdin/risk/
│   │   │   │       ├── RiskAggregationService.java
│   │   │   │       ├── CacheManager.java
│   │   │   │       └── grpc/RiskGrpcServer.java
│   │   │   ├── build.gradle
│   │   │   └── application.yml
│   │   └── README.md
│   │
│   ├── oms-service/
│   │   ├── src/main/java/com/aladdin/oms/
│   │   │   ├── api/
│   │   │   │   ├── OrderController.java
│   │   │   │   ├── ExecutionController.java
│   │   │   │   └── AllocationController.java
│   │   │   ├── domain/
│   │   │   │   ├── Order.java
│   │   │   │   ├── Execution.java
│   │   │   │   ├── OrderState.java
│   │   │   │   └── Allocation.java
│   │   │   ├── fix/
│   │   │   │   ├── FIXSessionManager.java
│   │   │   │   ├── FIXMessageHandler.java
│   │   │   │   ├── adapters/
│   │   │   │   │   ├── InteractiveBrokersAdapter.java
│   │   │   │   │   ├── BloombergEMSXAdapter.java
│   │   │   │   │   └── TradingTechAdapter.java
│   │   │   │   └── config/
│   │   │   │       └── fix-session-config.properties
│   │   │   ├── routing/
│   │   │   │   ├── SmartOrderRouter.java
│   │   │   │   └── algo/
│   │   │   │       ├── TWAPAlgorithm.java
│   │   │   │       ├── VWAPAlgorithm.java
│   │   │   │       └── POVAlgorithm.java
│   │   │   └── allocation/
│   │   │       └── AllocationEngine.java
│   │   ├── build.gradle
│   │   ├── quickfixj.cfg
│   │   ├── Dockerfile
│   │   └── README.md
│   │
│   ├── market-data-service/
│   │   ├── ingest/
│   │   │   ├── python/
│   │   │   │   ├── src/
│   │   │   │   │   ├── bloomberg_connector.py
│   │   │   │   │   ├── refinitiv_connector.py
│   │   │   │   │   ├── iex_cloud_connector.py
│   │   │   │   │   └── websocket_client.py
│   │   │   │   ├── requirements.txt
│   │   │   │   └── Dockerfile
│   │   │   └── java/
│   │   │       └── src/main/java/
│   │   │           └── normalizer/
│   │   │               ├── QuoteNormalizer.java
│   │   │               └── StreamProcessor.java
│   │   ├── storage/
│   │   │   └── sql/
│   │   │       ├── schema.sql
│   │   │       └── timescale-config.sql
│   │   ├── api/
│   │   │   └── src/main/java/
│   │   │       └── MarketDataController.java
│   │   ├── docker-compose.yml
│   │   └── README.md
│   │
│   ├── compliance-service/
│   │   ├── src/main/java/com/aladdin/compliance/
│   │   │   ├── rules/
│   │   │   │   ├── RuleEngine.java
│   │   │   │   ├── PreTradeChecker.java
│   │   │   │   ├── PostTradeMonitor.java
│   │   │   │   └── drl/
│   │   │   │       ├── position_limits.drl
│   │   │   │       ├── ucits_rules.drl
│   │   │   │       └── restricted_securities.drl
│   │   │   ├── reporting/
│   │   │   │   ├── MiFIDReporter.java
│   │   │   │   └── DoddFrankReporter.java
│   │   │   └── api/
│   │   │       └── ComplianceController.java
│   │   ├── build.gradle
│   │   └── README.md
│   │
│   ├── ai-copilot-service/
│   │   ├── src/
│   │   │   ├── agent/
│   │   │   │   ├── copilot_agent.py
│   │   │   │   ├── intent_detector.py
│   │   │   │   ├── tool_router.py
│   │   │   │   └── response_generator.py
│   │   │   ├── tools/
│   │   │   │   ├── portfolio_tool.py
│   │   │   │   ├── risk_tool.py
│   │   │   │   ├── oms_tool.py
│   │   │   │   └── market_data_tool.py
│   │   │   ├── llm/
│   │   │   │   ├── openai_client.py
│   │   │   │   ├── claude_client.py
│   │   │   │   └── prompt_templates.py
│   │   │   └── vector_store/
│   │   │       ├── pinecone_client.py
│   │   │       └── embeddings.py
│   │   ├── requirements.txt
│   │   ├── pyproject.toml
│   │   ├── Dockerfile
│   │   └── README.md
│   │
│   ├── reference-data-service/
│   │   ├── src/main/java/
│   │   │   └── com/aladdin/refdata/
│   │   │       ├── SecurityMaster.java
│   │   │       ├── CalendarService.java
│   │   │       ├── FXRateService.java
│   │   │       └── LEIRegistry.java
│   │   ├── data/
│   │   │   ├── securities.csv
│   │   │   ├── calendars.json
│   │   │   └── fx_pairs.csv
│   │   └── README.md
│   │
│   ├── reporting-service/
│   │   ├── python/
│   │   │   ├── src/
│   │   │   │   ├── report_generator.py
│   │   │   │   ├── templates/
│   │   │   │   │   ├── performance_tearsheet.html
│   │   │   │   │   └── risk_report.html
│   │   │   │   └── pdf_exporter.py
│   │   │   └── requirements.txt
│   │   ├── scala/
│   │   │   └── src/main/scala/
│   │   │       └── com/aladdin/reporting/
│   │   │           └── BatchReportJob.scala
│   │   └── README.md
│   │
│   └── identity-service/
│       ├── keycloak/
│       │   ├── realm-config.json
│       │   ├── themes/
│       │   │   └── aladdin/
│       │   │       └── login/
│       │   └── docker-compose.yml
│       └── README.md
│
├── frontend/
│   ├── web-portal/
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── (dashboard)/
│   │   │   │   │   ├── page.tsx
│   │   │   │   │   └── layout.tsx
│   │   │   │   ├── portfolio/
│   │   │   │   │   ├── [id]/
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   ├── positions/
│   │   │   │   │   │   ├── performance/
│   │   │   │   │   │   └── risk/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── trading/
│   │   │   │   │   ├── order-entry/
│   │   │   │   │   ├── blotter/
│   │   │   │   │   └── execution-analytics/
│   │   │   │   ├── risk/
│   │   │   │   │   ├── var-dashboard/
│   │   │   │   │   └── stress-test/
│   │   │   │   ├── compliance/
│   │   │   │   │   └── breaches/
│   │   │   │   └── reports/
│   │   │   ├── components/
│   │   │   │   ├── ui/
│   │   │   │   │   ├── Button.tsx
│   │   │   │   │   ├── Table.tsx
│   │   │   │   │   ├── Chart.tsx
│   │   │   │   │   └── DataGrid.tsx
│   │   │   │   ├── portfolio/
│   │   │   │   │   ├── PositionTable.tsx
│   │   │   │   │   └── PerformanceChart.tsx
│   │   │   │   ├── risk/
│   │   │   │   │   ├── VaRGauge.tsx
│   │   │   │   │   └── StressTestResults.tsx
│   │   │   │   └── trading/
│   │   │   │       ├── OrderTicket.tsx
│   │   │   │       └── Blotter.tsx
│   │   │   ├── lib/
│   │   │   │   ├── api/
│   │   │   │   │   ├── portfolio-client.ts
│   │   │   │   │   ├── risk-client.ts
│   │   │   │   │   └── oms-client.ts
│   │   │   │   ├── hooks/
│   │   │   │   │   ├── usePortfolio.ts
│   │   │   │   │   └── useWebSocket.ts
│   │   │   │   └── utils/
│   │   │   │       └── formatters.ts
│   │   │   └── styles/
│   │   │       └── globals.css
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── tailwind.config.ts
│   │   ├── next.config.js
│   │   └── README.md
│   └── desktop-client/
│       ├── src/
│       │   ├── main.ts
│       │   └── renderer/
│       ├── package.json
│       └── README.md
│
├── messaging/
│   ├── protobuf-schemas/
│   │   ├── portfolio.proto
│   │   ├── risk.proto
│   │   ├── oms.proto
│   │   ├── market_data.proto
│   │   ├── compliance.proto
│   │   └── common.proto
│   ├── sdk/
│   │   ├── java/
│   │   │   └── build.gradle
│   │   ├── python/
│   │   │   ├── setup.py
│   │   │   └── generated/
│   │   ├── cpp/
│   │   │   └── CMakeLists.txt
│   │   ├── julia/
│   │   │   └── Project.toml
│   │   └── typescript/
│   │       └── package.json
│   ├── buf.yaml
│   ├── buf.gen.yaml
│   └── README.md
│
├── infra/
│   ├── kubernetes/
│   │   ├── base/
│   │   │   ├── namespace.yaml
│   │   │   ├── service-accounts.yaml
│   │   │   └── network-policies.yaml
│   │   ├── services/
│   │   │   ├── portfolio-service/
│   │   │   │   ├── deployment.yaml
│   │   │   │   ├── service.yaml
│   │   │   │   ├── configmap.yaml
│   │   │   │   └── hpa.yaml
│   │   │   ├── risk-engine/
│   │   │   │   ├── deployment.yaml
│   │   │   │   ├── service.yaml
│   │   │   │   └── gpu-pod.yaml
│   │   │   ├── oms-service/
│   │   │   ├── market-data-service/
│   │   │   ├── compliance-service/
│   │   │   └── ai-copilot-service/
│   │   ├── data-layer/
│   │   │   ├── postgresql/
│   │   │   │   ├── statefulset.yaml
│   │   │   │   ├── pvc.yaml
│   │   │   │   └── backup-cronjob.yaml
│   │   │   ├── redis/
│   │   │   │   └── deployment.yaml
│   │   │   ├── kafka/
│   │   │   │   ├── zookeeper.yaml
│   │   │   │   └── kafka-cluster.yaml
│   │   │   └── timescaledb/
│   │   │       └── statefulset.yaml
│   │   ├── observability/
│   │   │   ├── prometheus/
│   │   │   │   ├── prometheus.yaml
│   │   │   │   └── service-monitor.yaml
│   │   │   ├── grafana/
│   │   │   │   ├── deployment.yaml
│   │   │   │   └── dashboards/
│   │   │   │       ├── portfolio-dashboard.json
│   │   │   │       ├── risk-dashboard.json
│   │   │   │       └── oms-dashboard.json
│   │   │   ├── loki/
│   │   │   │   └── deployment.yaml
│   │   │   └── jaeger/
│   │   │       └── deployment.yaml
│   │   ├── ingress/
│   │   │   ├── nginx-ingress.yaml
│   │   │   └── cert-manager.yaml
│   │   └── kustomization.yaml
│   │
│   ├── helm/
│   │   ├── aladdin-platform/
│   │   │   ├── Chart.yaml
│   │   │   ├── values.yaml
│   │   │   ├── values-dev.yaml
│   │   │   ├── values-prod.yaml
│   │   │   └── templates/
│   │   │       ├── _helpers.tpl
│   │   │       ├── service.yaml
│   │   │       └── deployment.yaml
│   │   └── charts/
│   │       ├── portfolio-service/
│   │       ├── risk-engine/
│   │       └── oms-service/
│   │
│   ├── docker-compose/
│   │   ├── infra.yml               # PostgreSQL, Redis, Kafka
│   │   ├── services.yml            # All services
│   │   ├── observability.yml       # Prometheus, Grafana
│   │   └── full-stack.yml          # Complete local stack
│   │
│   ├── terraform/
│   │   ├── aws/
│   │   │   ├── main.tf
│   │   │   ├── eks-cluster.tf
│   │   │   ├── rds.tf
│   │   │   ├── elasticache.tf
│   │   │   ├── msk.tf              # Managed Kafka
│   │   │   └── variables.tf
│   │   ├── azure/
│   │   │   ├── main.tf
│   │   │   ├── aks-cluster.tf
│   │   │   └── cosmos-db.tf
│   │   └── gcp/
│   │       ├── main.tf
│   │       └── gke-cluster.tf
│   │
│   └── kafka/
│       ├── topics.yaml
│       ├── schemas.yaml
│       └── connect-config.yaml
│
├── libs/
│   ├── common-java/
│   │   ├── src/main/java/com/aladdin/common/
│   │   │   ├── domain/
│   │   │   │   ├── AggregateRoot.java
│   │   │   │   └── DomainEvent.java
│   │   │   ├── exception/
│   │   │   │   └── BusinessException.java
│   │   │   └── util/
│   │   │       ├── MoneyUtil.java
│   │   │       └── DateUtil.java
│   │   └── build.gradle
│   │
│   ├── common-python/
│   │   ├── aladdin_common/
│   │   │   ├── logging.py
│   │   │   ├── metrics.py
│   │   │   └── config.py
│   │   ├── setup.py
│   │   └── requirements.txt
│   │
│   └── quant-models/
│       ├── julia/
│   │       ├── src/
│   │       │   ├── QuantModels.jl
│   │       │   ├── black_scholes.jl
│   │       │   └── factor_models.jl
│   │       └── Project.toml
│       └── cpp/
│           ├── src/
│           │   └── pricing/
│           └── CMakeLists.txt
│
├── data/
│   ├── airflow-dags/
│   │   ├── eod_portfolio_reconciliation.py
│   │   ├── risk_calculation_batch.py
│   │   ├── market_data_ingestion.py
│   │   └── regulatory_reporting.py
│   ├── sql-migrations/
│   │   ├── portfolio/
│   │   │   ├── V1__init_schema.sql
│   │   │   └── V2__add_indexes.sql
│   │   ├── risk/
│   │   │   └── V1__init_risk_tables.sql
│   │   └── market-data/
│   │       └── V1__timeseries_setup.sql
│   └── seeds/
│       ├── test_portfolios.sql
│       └── sample_securities.csv
│
├── docs/
│   ├── architecture/
│   │   ├── adr/
│   │   │   ├── 001-microservices-architecture.md
│   │   │   ├── 002-event-driven-messaging.md
│   │   │   ├── 003-multi-language-strategy.md
│   │   │   └── 004-data-lake-design.md
│   │   ├── diagrams/
│   │   │   ├── system-architecture.puml
│   │   │   ├── data-flow.puml
│   │   │   ├── deployment-architecture.puml
│   │   │   └── c4/
│   │   │       ├── system-context.puml
│   │   │       ├── container-diagram.puml
│   │   │       └── component-diagram.puml
│   │   └── tech-radar.md
│   │
│   ├── api/
│   │   ├── openapi/
│   │   │   ├── portfolio-service.yaml
│   │   │   ├── risk-engine.yaml
│   │   │   └── oms-service.yaml
│   │   └── grpc/
│   │       └── service-catalog.md
│   │
│   ├── guides/
│   │   ├── getting-started.md
│   │   ├── local-development.md
│   │   ├── deployment-guide.md
│   │   └── troubleshooting.md
│   │
│   └── runbooks/
│       ├── incident-response.md
│       ├── backup-recovery.md
│       └── scaling-guide.md
│
├── tests/
│   ├── integration/
│   │   ├── portfolio-service/
│   │   │   └── PortfolioAPITest.java
│   │   ├── oms-service/
│   │   │   └── OrderFlowTest.java
│   │   └── end-to-end/
│   │       └── TradeLifecycleTest.java
│   │
│   ├── performance/
│   │   ├── k6/
│   │   │   ├── portfolio-load-test.js
│   │   │   ├── risk-calc-load-test.js
│   │   │   └── oms-stress-test.js
│   │   └── jmeter/
│   │       └── trading-simulation.jmx
│   │
│   └── chaos/
│       ├── chaos-mesh/
│       │   ├── network-delay.yaml
│       │   ├── pod-failure.yaml
│       │   └── stress-scenarios.yaml
│       └── litmus/
│           └── experiments/
│
├── scripts/
│   ├── setup/
│   │   ├── init-dev-env.sh
│   │   ├── setup-k8s-cluster.sh
│   │   └── install-dependencies.sh
│   ├── deployment/
│   │   ├── deploy-services.sh
│   │   └── rollback.sh
│   └── maintenance/
│       ├── backup-db.sh
│       └── clean-cache.sh
│
├── .gitignore
├── .editorconfig
├── LICENSE
├── README.md
├── CONTRIBUTING.md
└── PROJECT_STRUCTURE.md
```

---

## File Count by Category

| Category | Files | Languages |
|----------|-------|-----------|
| **Services (Backend)** | ~200 | Java, C++, Julia, Python |
| **Frontend** | ~50 | TypeScript/React |
| **Infrastructure** | ~80 | YAML, HCL (Terraform) |
| **Protobuf/Messaging** | ~15 | Protobuf |
| **Documentation** | ~30 | Markdown, PlantUML |
| **Tests** | ~40 | Java, TypeScript, JS |
| **Configuration** | ~50 | YAML, Properties |
| **Total** | **~465 files** | Multi-language |

---

## Next Steps for Implementation

### Phase 1: Core Infrastructure (Week 1-2)
1. Set up Kubernetes cluster (local minikube or cloud EKS/AKS)
2. Deploy PostgreSQL, Redis, Kafka via Helm
3. Configure observability stack (Prometheus, Grafana, Jaeger)
4. Set up CI/CD pipelines

### Phase 2: Portfolio & Market Data (Week 3-5)
1. Implement Portfolio Service (Java)
2. Build Market Data ingestion (Python)
3. Create Reference Data Service
4. Implement basic web UI for portfolio view

### Phase 3: Risk Engine (Week 6-8)
1. Build C++ pricing workers with QuantLib
2. Implement Julia scenario engine
3. Create Java aggregation service
4. Build risk dashboards in frontend

### Phase 4: OMS & Compliance (Week 9-11)
1. Implement OMS Service with QuickFIX/J
2. Integrate with test FIX broker
3. Build compliance rule engine
4. Create trading UI

### Phase 5: AI & Advanced Features (Week 12-14)
1. Implement AI Copilot with LangGraph
2. Build reporting engine
3. Add advanced analytics
4. Performance optimization

---

## Technology Breakdown by Service

### Java Services (Spring Boot)
- Portfolio Service
- OMS Service  
- Compliance Service
- Reference Data Service
- Risk Engine (aggregation layer)

### C++ Components
- Pricing kernels (QuantLib)
- CUDA kernels for GPU acceleration
- High-frequency calculation workers

### Python Services
- Market Data ingestion
- AI Copilot Service
- Reporting engine
- Data processing scripts

### Julia Components
- Quant models (VaR, stress testing)
- Portfolio optimization
- Factor analysis

### TypeScript/React
- Web portal (Next.js)
- Component library
- API clients

---

This structure provides a complete blueprint for building an enterprise-grade investment management platform comparable to BlackRock Aladdin.
