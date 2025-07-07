# LFX v2 helm charts

This repository contains Helm charts for deploying the LFX v2 platform on Kubernetes.

## Repository structure

```text
lfx-v2-helm/
├── charts/
│   └── lfx-platform/       # Main LFX Platform chart
│       ├── charts/         # Subcharts
│       ├── templates/      # Kubernetes templates
│       ├── Chart.yaml      # Chart metadata
│       ├── values.yaml     # Default values
│       └── README.md       # Documentation
```

## Installation

To install the LFX Platform chart:

```bash
# Add required Helm repositories
helm repo add traefik https://traefik.github.io/charts
helm repo add dadrus https://dadrus.github.io/heimdall/charts
helm repo add jouve https://jouve.github.io/charts/
helm repo add nats https://nats-io.github.io/k8s/helm/charts/
helm repo add openfga https://openfga.github.io/helm-charts
helm repo add opensearch https://opensearch-project.github.io/helm-charts/
helm repo update

# Create namespace
kubectl create namespace lfx

# Install the chart
helm install lfx-platform ./charts/lfx-platform -n lfx
```

## Components

The LFX v2 Platform includes the following core components:

- **Traefik**: API Gateway and Ingress Controller
- **OpenFGA**: Fine-Grained Authorization with Relationship-Based Access Control (ReBAC)
- **Heimdall**: Identity and Access Proxy
- **NATS**: Messaging and event streaming system

## Configuration

See the [lfx-platform chart README](./charts/lfx-platform/README.md) for configuration options and examples.

## Development

To contribute to this repository:

1. Fork the repository
2. Make your changes
3. Submit a pull request

## License

Copyright The Linux Foundation and each contributor to LFX.

This project’s source code is licensed under the MIT License. A copy of the
license is available in `LICENSE`.

This project’s documentation is licensed under the Creative Commons Attribution
4.0 International License \(CC-BY-4.0\). A copy of the license is available in
`LICENSE-docs`.
