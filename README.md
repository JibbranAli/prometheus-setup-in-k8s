# Kubernetes Prometheus Monitoring Stack

A complete Prometheus monitoring solution for Kubernetes clusters using k3d, featuring automated service discovery and node metrics collection.

## Overview

This project provides a production-ready Prometheus monitoring stack that can be deployed on any Kubernetes cluster. It includes automated service discovery for monitoring pods and collects essential node metrics through Node Exporter.

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Prometheus    │───▶│  Node Exporter   │───▶│   Node Metrics  │
│   (Port 9090)   │    │   (Port 9100)    │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
         │
         ▼
┌─────────────────┐
│  Web Interface  │
│ (NodePort 30090)│
└─────────────────┘
```

## Features

- **Automated Service Discovery**: Kubernetes-native pod discovery using labels
- **Node Metrics Collection**: System-level metrics via Node Exporter
- **RBAC Security**: Proper role-based access control configuration
- **External Access**: NodePort service for easy browser access
- **Fast Scraping**: 5-second scrape interval for real-time monitoring
- **Lightweight**: Optimized for k3d and resource-constrained environments

## Prerequisites

- Docker installed and running
- k3d CLI tool
- kubectl configured
- Basic understanding of Kubernetes concepts

## Quick Start

### 1. Create k3d Cluster

```bash
k3d cluster create observability \
  --agents 1 \
  -p "30090:30090@loadbalancer"
```

Verify cluster creation:
```bash
kubectl get nodes
```

### 2. Deploy Monitoring Stack

```bash
# Create monitoring namespace
kubectl create namespace monitoring

# Apply all configurations
kubectl apply -f prometheus-rbac.yaml
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-service.yaml
kubectl apply -f node-exporter-pod.yaml
```

### 3. Verify Deployment

```bash
# Check all pods are running
kubectl get pods -n monitoring

# Verify Node Exporter metrics
kubectl exec -it node-exporter-pod -n monitoring -- wget -qO- http://localhost:9100/metrics
```

### 4. Access Prometheus

Open your browser and navigate to:
```
http://localhost:30090
```

## Configuration Files

| File | Purpose |
|------|---------|
| `prometheus-rbac.yaml` | Service account and RBAC permissions |
| `prometheus-config.yaml` | Prometheus configuration and scrape targets |
| `prometheus-deployment.yaml` | Prometheus server deployment |
| `prometheus-service.yaml` | NodePort service for external access |
| `node-exporter-pod.yaml` | Node Exporter pod for system metrics |

## Monitoring Capabilities

### Available Metrics

- **System Metrics**: CPU, memory, disk, network usage
- **Pod Discovery**: Automatic detection of labeled pods
- **Kubernetes Metrics**: Cluster and node-level statistics

### Sample PromQL Queries

```promql
# Check service availability
up

# CPU usage over time
node_cpu_seconds_total

# Available memory
node_memory_MemAvailable_bytes

# Pod count by namespace
count by (namespace) (kube_pod_info)
```

## Customization

### Modify Scrape Interval

Edit `prometheus-config.yaml`:
```yaml
global:
  scrape_interval: 15s  # Change from 5s to 15s
```

### Add New Scrape Targets

Add to the `scrape_configs` section in `prometheus-config.yaml`:
```yaml
- job_name: "my-application"
  static_configs:
    - targets: ["app:8080"]
```

### Scale Prometheus

Modify replicas in `prometheus-deployment.yaml`:
```yaml
spec:
  replicas: 2  # Scale to 2 replicas
```

## Troubleshooting

### Common Issues

**Pods not starting:**
```bash
kubectl describe pod <pod-name> -n monitoring
kubectl logs <pod-name> -n monitoring
```

**Metrics not appearing:**
```bash
# Check service discovery
kubectl get endpoints -n monitoring

# Verify pod labels
kubectl get pods -n monitoring --show-labels
```

**Access issues:**
```bash
# Check service status
kubectl get svc -n monitoring

# Verify NodePort
kubectl get svc prometheus-service -n monitoring -o yaml
```

## Security Considerations

- RBAC is configured with minimal required permissions
- Service account follows principle of least privilege
- No persistent storage configured (metrics are ephemeral)
- Consider adding network policies for production use

## Production Recommendations

- Add persistent storage for long-term metric retention
- Configure alerting rules and Alertmanager
- Implement proper backup strategies
- Add resource limits and requests
- Consider using Helm charts for easier management
- Set up TLS/SSL for secure access

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly
5. Submit a pull request

## License

This project is open source and available under the MIT License.

## Support

For issues and questions:
- Check the troubleshooting section
- Review Kubernetes and Prometheus documentation
- Open an issue in the repository

---

**Built with ❤️ for the Kubernetes community**