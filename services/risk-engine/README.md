# Risk Engine

> Real-time and batch risk analytics engine — VaR, stress testing, greeks

## Overview

Risk Engine is the high-performance calculation core providing:
- Market risk metrics (VaR, Expected Shortfall)
- Stress testing and scenario analysis
- Factor risk attribution (PCA-based)
- Greeks calculation for derivatives
- Sensitivity analysis (DV01, convexity)

## Tech Stack

**Performance Layer (C++20):**
- Pricing kernels with QuantLib integration
- GPU acceleration via CUDA for Monte Carlo
- Parallel computation with OpenMP/TBB

**Quant Models (Julia 1.10+):**
- Risk factor models (DifferentialEquations.jl)
- Portfolio optimization (JuMP.jl, Ipopt)
- Statistical analysis (Distributions.jl)

**Service Layer (Java):**
- gRPC server for API exposure
- Aggregation and caching
- Job scheduling and orchestration

**Storage:**
- Redis/Hazelcast (distributed cache)
- TimescaleDB (calculation results)

## Architecture

```
Risk Engine Architecture

┌─────────────────────────────────────────────┐
│         gRPC API Server (Java)              │
│  - Request routing                          │
│  - Result aggregation                       │
│  - Cache management                         │
└──────────────────┬──────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
┌───────────────┐    ┌───────────────┐
│ Pricing Grid  │    │ Scenario      │
│ (C++ Workers) │    │ Engine (Julia)│
│               │    │               │
│ - QuantLib    │    │ - Factor      │
│ - Black-      │    │   models      │
│   Scholes     │    │ - Stress      │
│ - Monte Carlo │    │   scenarios   │
│ - GPU Accel   │    │ - Copulas     │
└───────────────┘    └───────────────┘
        │                     │
        └──────────┬──────────┘
                   ▼
        ┌────────────────────┐
        │  Cache & Storage   │
        │  (Redis/TimescaleDB)│
        └────────────────────┘
```

## Key Calculations

### Value at Risk (VaR)

**Methods:**
1. **Historical VaR** — Based on historical returns
2. **Parametric VaR** — Assumes normal distribution
3. **Monte Carlo VaR** — Simulation-based

**Implementation (Julia):**
```julia
function calculate_var(returns::Vector{Float64}, 
                       confidence::Float64=0.99,
                       method::Symbol=:historical)
    if method == :historical
        return quantile(returns, 1 - confidence)
    elseif method == :parametric
        μ, σ = mean(returns), std(returns)
        return μ + σ * quantile(Normal(), 1 - confidence)
    elseif method == :monte_carlo
        # Monte Carlo simulation
        simulations = simulate_portfolio(returns, 10000)
        return quantile(simulations, 1 - confidence)
    end
end
```

### Greeks Calculation

**C++ Pricing Kernel (with QuantLib):**
```cpp
#include <ql/quantlib.hpp>

using namespace QuantLib;

struct Greeks {
    Real delta;
    Real gamma;
    Real vega;
    Real theta;
    Real rho;
};

Greeks calculateGreeks(
    Real spot,
    Real strike,
    Real vol,
    Real riskFreeRate,
    Time maturity,
    Option::Type type
) {
    // Black-Scholes pricing
    Handle<Quote> underlyingH(
        boost::shared_ptr<Quote>(new SimpleQuote(spot)));
    
    // ... QuantLib setup ...
    
    Greeks greeks;
    greeks.delta = option->delta();
    greeks.gamma = option->gamma();
    greeks.vega = option->vega();
    greeks.theta = option->theta();
    greeks.rho = option->rho();
    
    return greeks;
}
```

### Stress Testing

**Scenario Types:**
1. **Historical Scenarios:**
   - 2008 Financial Crisis
   - 2020 COVID-19 Crash
   - 2022 Russia-Ukraine

2. **Hypothetical Scenarios:**
   - Rates +/-100bps
   - Equity -20%
   - Credit spread widening

3. **Factor Shocks:**
   - Principal components
   - Risk factors (momentum, value, size)

## gRPC API

**Service Definition (Protobuf):**
```protobuf
service RiskService {
  // Calculate VaR for portfolio
  rpc CalculateVaR(VaRRequest) returns (VaRResponse);
  
  // Run stress test
  rpc RunStressTest(StressTestRequest) returns (StressTestResponse);
  
  // Get portfolio greeks
  rpc GetGreeks(GreeksRequest) returns (GreeksResponse);
  
  // Calculate factor exposures
  rpc GetFactorExposures(FactorRequest) returns (FactorResponse);
}

message VaRRequest {
  string portfolio_id = 1;
  double confidence_level = 2;  // 0.95, 0.99
  int32 horizon_days = 3;        // 1, 10, 30
  VaRMethod method = 4;          // HISTORICAL, PARAMETRIC, MONTE_CARLO
}

message VaRResponse {
  double var_amount = 1;
  double expected_shortfall = 2;
  double portfolio_value = 3;
  double var_percentage = 4;
}
```

## Build & Run

### C++ Pricing Engine

```bash
# Install dependencies
sudo apt-get install libquantlib0-dev cuda-toolkit

# Build
cd cpp/pricing
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
make -j8

# Run worker
./risk_worker --port=50051
```

### Julia Scenario Engine

```bash
# Install Julia packages
julia -e 'using Pkg; Pkg.add(["DifferentialEquations", 
                              "Distributions", 
                              "Optimization"])'

# Run engine
julia --project=. src/scenario_engine.jl
```

### Java Service Layer

```bash
# Build and run
./gradlew bootRun

# Or with Docker
docker build -t risk-engine .
docker run -p 8080:8080 risk-engine
```

## Configuration

**application.yml:**
```yaml
risk:
  workers:
    cpp:
      - host: localhost
        port: 50051
      - host: localhost
        port: 50052
    julia:
      - host: localhost
        port: 50053
  
  cache:
    type: redis
    host: localhost
    port: 6379
    ttl: 3600  # 1 hour
  
  computation:
    parallel-jobs: 8
    gpu-enabled: true
    cuda-devices: [0, 1]
```

## Performance

**Benchmarks (10,000 position portfolio):**

| Calculation | Method | Time |
|------------|--------|------|
| VaR | Historical | 50ms |
| VaR | Monte Carlo (CPU) | 2.5s |
| VaR | Monte Carlo (GPU) | 180ms |
| Greeks | Black-Scholes | 120ms |
| Stress Test | 10 scenarios | 850ms |
| Factor Attribution | PCA | 300ms |

## GPU Acceleration

**CUDA Monte Carlo:**
```cpp
__global__ void monte_carlo_kernel(
    float* prices,
    float* returns,
    int num_simulations,
    int num_steps
) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < num_simulations) {
        // Simulate price path
        curandState state;
        curand_init(1234, idx, 0, &state);
        
        float price = prices[0];
        for (int i = 0; i < num_steps; i++) {
            float z = curand_normal(&state);
            price *= exp(returns[i] + z * 0.15);  // volatility
        }
        prices[idx] = price;
    }
}
```

## Monitoring

**Metrics (Prometheus):**
- `risk.calculations.duration` — Calculation latency
- `risk.cache.hit_rate` — Cache efficiency  
- `risk.gpu.utilization` — GPU usage
- `risk.workers.queue_depth` — Worker backlog

## Roadmap

- [ ] Credit risk (CVA, DVA)
- [ ] Counterparty risk (PFE, EPE)
- [ ] Liquidity risk (LVaR)
- [ ] XVA calculations
- [ ] Machine learning-based risk models
- [ ] Real-time intraday VaR
