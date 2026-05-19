# Aladdin Platform Infrastructure

This directory contains all infrastructure-as-code configurations for the Aladdin Platform.

## Overview

The infrastructure is built on Kubernetes and includes:
- **Kubernetes base configurations**: Namespaces, RBAC, service accounts
- **PostgreSQL cluster**: Highly available database with StatefulSets
- **Kafka cluster**: Event streaming with ZooKeeper coordination
- **CI/CD pipelines**: GitHub Actions workflows for automated deployments

## Directory Structure

```
infra/
├── k8s/
│   ├── base/              # Base Kubernetes resources
│   │   ├── namespace.yaml # Namespace definitions
│   │   └── rbac.yaml      # RBAC policies
│   ├── database/
│   │   └── postgres/      # PostgreSQL StatefulSet
│   │       ├── statefulset.yaml
│   │       └── secret.yaml
│   └── messaging/
│       └── kafka/         # Kafka cluster
│           └── cluster.yaml
└── README.md             # This file
```

## Prerequisites

- Kubernetes cluster (v1.27+)
- kubectl CLI tool
- Helm (optional, for advanced deployments)
- Storage class configured for persistent volumes

## Quick Start

### 1. Deploy Base Infrastructure

```bash
# Create namespaces
kubectl apply -f k8s/base/namespace.yaml

# Apply RBAC policies
kubectl apply -f k8s/base/rbac.yaml
```

### 2. Deploy PostgreSQL

```bash
# Create database secrets (IMPORTANT: Change password in production!)
kubectl apply -f k8s/database/postgres/secret.yaml

# Deploy PostgreSQL cluster
kubectl apply -f k8s/database/postgres/statefulset.yaml

# Verify deployment
kubectl get statefulsets -n aladdin-platform
kubectl get pods -n aladdin-platform -l app=postgres
```

### 3. Deploy Kafka

```bash
# Deploy Kafka cluster with ZooKeeper
kubectl apply -f k8s/messaging/kafka/cluster.yaml

# Verify deployment
kubectl get statefulsets -n aladdin-platform
kubectl get pods -n aladdin-platform -l app=kafka
kubectl get pods -n aladdin-platform -l app=zookeeper
```

## Component Details

### PostgreSQL Cluster

- **Replicas**: 3 (high availability)
- **Storage**: 50Gi per instance
- **Resources**: 2-4Gi memory, 1-2 CPU cores
- **Version**: PostgreSQL 15 Alpine
- **Features**: 
  - Persistent storage with PVCs
  - Health checks (liveness/readiness probes)
  - Secrets-based credential management

**Connection String**:
```
postgresql://postgres-0.postgres-service:5432/aladdin
```

### Kafka Cluster

- **Kafka Replicas**: 3
- **ZooKeeper Replicas**: 3
- **Storage**: 50Gi per Kafka broker, 10Gi per ZooKeeper
- **Resources**: 
  - Kafka: 2-4Gi memory, 1-2 CPU cores
  - ZooKeeper: 1-2Gi memory, 0.5-1 CPU
- **Version**: Kafka 7.5.0, ZooKeeper 3.8
- **Configuration**:
  - Replication factor: 3
  - Min in-sync replicas: 2

**Bootstrap Servers**:
```
kafka-0.kafka-service:9092,kafka-1.kafka-service:9092,kafka-2.kafka-service:9092
```

## Security Considerations

### Secrets Management

⚠️ **IMPORTANT**: The default PostgreSQL password in `k8s/database/postgres/secret.yaml` is a placeholder.

**For production deployments**:
1. Use external secret management (HashiCorp Vault, AWS Secrets Manager)
2. Rotate credentials regularly
3. Never commit real secrets to version control

### Network Policies

To be implemented in Phase 2:
- Network isolation between namespaces
- Egress/ingress policies
- Pod-to-pod communication restrictions

### RBAC

The current RBAC configuration provides:
- Service account for platform services
- Read-only access to Kubernetes resources
- Job creation permissions for batch processing

## Monitoring & Observability

To be implemented in Phase 2:
- Prometheus for metrics collection
- Grafana dashboards
- Jaeger for distributed tracing
- ELK stack for log aggregation

## Scaling

### PostgreSQL

```bash
# Scale replicas (not recommended without proper setup)
kubectl scale statefulset postgres --replicas=5 -n aladdin-platform
```

### Kafka

```bash
# Scale Kafka brokers
kubectl scale statefulset kafka --replicas=5 -n aladdin-platform

# Scale ZooKeeper (always use odd numbers)
kubectl scale statefulset zookeeper --replicas=5 -n aladdin-platform
```

## Troubleshooting

### PostgreSQL Issues

```bash
# Check pod status
kubectl get pods -n aladdin-platform -l app=postgres

# View logs
kubectl logs -n aladdin-platform postgres-0

# Connect to database
kubectl exec -it postgres-0 -n aladdin-platform -- psql -U postgres -d aladdin
```

### Kafka Issues

```bash
# Check pod status
kubectl get pods -n aladdin-platform -l app=kafka
kubectl get pods -n aladdin-platform -l app=zookeeper

# View Kafka logs
kubectl logs -n aladdin-platform kafka-0

# View ZooKeeper logs
kubectl logs -n aladdin-platform zookeeper-0
```

## Backup & Recovery

### PostgreSQL Backup

```bash
# Manual backup
kubectl exec -n aladdin-platform postgres-0 -- pg_dump -U postgres aladdin > backup.sql

# Restore
cat backup.sql | kubectl exec -i -n aladdin-platform postgres-0 -- psql -U postgres aladdin
```

### Kafka Backup

Kafka data is replicated across brokers. For disaster recovery:
1. Use MirrorMaker for cross-cluster replication
2. Regular snapshots of persistent volumes

## CI/CD Integration

GitHub Actions workflow (`.github/workflows/ci.yml`) automatically:
1. Builds and tests services on PR
2. Creates Docker images on main branch merge
3. Deploys to staging environment

See `.github/workflows/` for details.

## Next Steps

**Phase 2 - Service Deployments**:
- Deploy microservices (Portfolio, Risk, OMS, Market Data)
- Configure service meshes (Istio/Linkerd)
- Set up ingress controllers
- Implement API gateways

**Phase 3 - Observability**:
- Prometheus/Grafana setup
- Distributed tracing
- Log aggregation
- Alerting rules

## Contributing

See [CONTRIBUTING.md](../CONTRIBUTING.md) for guidelines.

## License

See [LICENSE](../LICENSE) for details.
