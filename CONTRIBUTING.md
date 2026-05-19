# Contributing to Aladdin Platform

Thank you for your interest in contributing to Aladdin Platform! We welcome contributions from the community.

## Getting Started

1. **Fork the repository**
2. **Clone your fork:**
   ```bash
   git clone https://github.com/YOUR-USERNAME/aladdin-platform.git
   cd aladdin-platform
   ```
3. **Create a branch:**
   ```bash
   git checkout -b feature/your-feature-name
   ```

## Development Setup

### Prerequisites
- Java 21 (for Java services)
- Python 3.11+ (for Python services)
- Node.js 20+ (for frontend)
- Docker & Docker Compose
- Julia 1.10+ (for quant models)
- C++20 compiler (for risk engine)

### Running Locally

```bash
# Start infrastructure
docker-compose -f infra/docker-compose/infra.yml up -d

# Start specific service
cd services/portfolio-service
./gradlew bootRun
```

## Code Style

### Java
- Follow Google Java Style Guide
- Use Spotless for formatting: `./gradlew spotlessApply`

### Python
- Follow PEP 8
- Use Black for formatting: `black .`
- Use ruff for linting: `ruff check .`

### TypeScript
- Follow Airbnb Style Guide
- Use Prettier: `npm run format`
- ESLint: `npm run lint`

## Testing

### Unit Tests
```bash
# Java
./gradlew test

# Python
pytest

# TypeScript
npm test
```

### Integration Tests
```bash
./gradlew integrationTest
```

## Pull Request Process

1. Ensure all tests pass
2. Update documentation if needed
3. Add tests for new features
4. Follow conventional commits format:
   - `feat`: New feature
   - `fix`: Bug fix
   - `docs`: Documentation changes
   - `refactor`: Code refactoring
   - `test`: Adding tests
   - `chore`: Maintenance tasks

## Areas for Contribution

### High Priority
- FIX protocol adapters for additional brokers
- Market data provider integrations
- Additional quant models (pricing, risk)
- UI/UX improvements

### Documentation
- API documentation
- Architecture diagrams
- Deployment guides
- Tutorials and examples

### Infrastructure
- Kubernetes optimizations
- Observability enhancements
- CI/CD improvements

## Code Review Guidelines

- Be respectful and constructive
- Focus on code quality and design
- Suggest improvements, not just problems
- Approve PRs that meet standards

## Questions?

- Open a [Discussion](https://github.com/9j8f7j5kwp-source/aladdin-platform/discussions)
- Join our [Slack community](https://aladdin-platform.slack.com)

## License

By contributing, you agree that your contributions will be licensed under Apache License 2.0.
