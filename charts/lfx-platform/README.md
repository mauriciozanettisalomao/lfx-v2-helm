# LFX platform umbrella Helm chart

This Helm chart deploys infrastructure components, platform services, and key
resource APIs for the LFX platform.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- PV provisioner support in the underlying infrastructure (if persistence is
  enabled)

## Secrets setup

Some subcharts require Kubernetes secrets to exist in the namespace before
installing the chart. These secrets are only needed if the corresponding
subchart is enabled.

To check whether a subchart is enabled, look for its `enabled` field in
`charts/lfx-platform/values.yaml`:

```bash
grep -A1 "lfx-v2-voting-service:" charts/lfx-platform/values.yaml
# enabled: false  ← skip secret creation if false
```

Secret values are stored in the **LFX V2** vault in 1Password under the note
**LFX Platform Chart Values Secrets - Local Development**.

### lfx-v2-voting-service

Requires an Auth0 client ID and RSA private key.

```bash
kubectl create secret generic lfx-v2-voting-service -n lfx \
  --from-literal=ITX_CLIENT_ID="<from-1password>" \
  --from-file=ITX_CLIENT_PRIVATE_KEY=/path/to/private.key
```

### lfx-v2-survey-service

Requires an Auth0 client ID and RSA private key.

```bash
kubectl create secret generic lfx-v2-survey-service -n lfx \
  --from-literal=ITX_CLIENT_ID="<from-1password>" \
  --from-file=ITX_CLIENT_PRIVATE_KEY=/path/to/private.key
```

### lfx-v2-meeting-service

Requires an Auth0 client ID and RSA private key.

```bash
kubectl create secret generic meeting-secrets -n lfx \
  --from-literal=auth0_client_id="<from-1password>" \
  --from-file=auth0_client_private_key=/path/to/private.key
```

### lfx-v2-mailing-list-service

Requires Groups.io credentials and a webhook secret.

```bash
kubectl create secret generic lfx-v2-mailing-list-service -n lfx \
  --from-literal=GROUPSIO_EMAIL="<from-1password>" \
  --from-literal=GROUPSIO_PASSWORD="<from-1password>" \
  --from-literal=GROUPSIO_WEBHOOK_SECRET="<from-1password>"
```

## Installing the chart

First, create the namespace (recommended):

```bash
kubectl create namespace lfx
```

### Installing via the OCI registry

```bash
# Install the latest version of the chart.
helm install -n lfx lfx-platform \
  oci://ghcr.io/linuxfoundation/lfx-v2-helm/chart/lfx-platform
```

For reproducible installs or when debugging a specific release, pin the version
with `--version`:

```bash
helm install -n lfx lfx-platform \
  oci://ghcr.io/linuxfoundation/lfx-v2-helm/chart/lfx-platform \
  --version <version>
```

### Installing from source

Clone the repository before running the following commands from the root of the
working directory.

```bash
# Pull down chart dependencies.
helm dependency update charts/lfx-platform

# Install the chart.
helm install -n lfx lfx-platform \
    ./charts/lfx-platform
```

### Customizing local development values

The default `values.yaml` is configured for local development. To override
specific values for your own environment without committing them, copy the
bundled example file:

```bash
cp charts/lfx-platform/values.local.example.yaml charts/lfx-platform/values.local.yaml
```

`values.local.yaml` is gitignored, so you can freely modify it. Pass it when
installing from OCI or from source:

```bash
# From OCI registry
helm install -n lfx lfx-platform \
  oci://ghcr.io/linuxfoundation/lfx-v2-helm/chart/lfx-platform \
  --values charts/lfx-platform/values.local.yaml

# From source
helm install -n lfx lfx-platform ./charts/lfx-platform \
  --values charts/lfx-platform/values.local.yaml
```

Later `--values` files take precedence over earlier ones, so you can also layer
additional overrides on top:

```bash
helm install -n lfx lfx-platform \
  oci://ghcr.io/linuxfoundation/lfx-v2-helm/chart/lfx-platform \
  --values charts/lfx-platform/values.local.yaml \
  --values my-overrides.yaml
```

Refer to the [Configuration](#configuration) section and the inline comments
in `values.yaml` for all available parameters.

## Uninstalling the chart

To uninstall/delete the `lfx-platform` deployment:

```bash
helm uninstall lfx-platform -n lfx
# Optional: delete the namespace to delete any persistent resources.
kubectl delete namespace lfx
```

## Configuration

The following table lists the configurable parameters of the LFX Platform chart
and their default values. You can override these values in your `values.local.yaml`
or by using the `--set` flag when installing the chart.

### Global parameters

| Parameter              | Description                     | Default           |
|------------------------|---------------------------------|-------------------|
| `lfx.domain`           | Domain for services             | `k8s.orb.local`   |
| `lfx.image.registry`   | Global Docker image registry    | `linuxfoundation` |
| `lfx.image.pullPolicy` | Global Docker image pull policy | `IfNotPresent`    |

### Subchart configuration

#### Traefik

| Parameter                                | Description              | Default |
|------------------------------------------|--------------------------|---------|
| `traefik.enabled`                        | Enable Traefik           | `true`  |
| `traefik.ingressRoute.dashboard.enabled` | Enable Traefik dashboard | `true`  |

For more Traefik configuration options, see the [Traefik Helm Chart documentation](https://github.com/traefik/traefik-helm-chart).

#### OpenFGA

| Parameter                          | Description                                      | Default |
|------------------------------------|--------------------------------------------------|---------|
| `openfga.enabled`                  | Enable OpenFGA                                   | `true`  |
| `openfga.postgres.enabled`         | Enable built-in PostgreSQL                       | `true`  |
| `openfga.datastore.existingSecret` | Secret for external PostgreSQL connection string | `nil`   |

For more OpenFGA configuration options, see the [OpenFGA Helm Chart documentation](https://github.com/openfga/helm-charts).

For information on managing OpenFGA see the [OpenFGA Documentation](../../docs/openfga.md).

#### Heimdall

| Parameter                              | Description                             | Default            |
|----------------------------------------|-----------------------------------------|--------------------|
| `heimdall.enabled`                     | Enable Heimdall                         | `true`             |
| `heimdall.autheliaIntegration.enabled` | Enable Authelia integration             | `true`             |
| `heimdall.secretsRef`                  | Heimdall secrets reference              | `heimdall-secrets` |
| `heimdall.certsSecretRef`              | Heimdall certificates secrets reference | `heimdall-certs`   |

For more Heimdall configuration options, see the [Heimdall Helm Chart documentation](https://github.com/dadrus/heimdall/tree/main/charts/heimdall).

#### NATS

| Parameter                | Description             | Default |
|--------------------------|-------------------------|---------|
| `nats.enabled`           | Enable NATS             | `true`  |
| `nats.cluster.enabled`   | Enable NATS clustering  | `true`  |
| `nats.cluster.replicas`  | Number of NATS replicas | `3`     |
| `nats.jetstream.enabled` | Enable JetStream        | `true`  |

For more NATS configuration options, see the [NATS Helm Chart documentation](https://github.com/nats-io/k8s/tree/main/helm/charts/nats).

#### Mailpit

| Parameter         | Description    | Default |
|-------------------|----------------|---------|
| `mailpit.enabled` | Enable Mailpit | `true`  |

For more mailpit configuration options, see the [Mailpit documentation](https://github.com/jouve/charts/tree/main/charts/mailpit).

#### Authelia

| Parameter          | Description     | Default |
|--------------------|-----------------|---------|
| `authelia.enabled` | Enable Authelia | `true`  |

For more authelia configuration options, see the [Authelia documentation](https://github.com/authelia/chartrepo/tree/master/charts/authelia).

#### NACK

| Parameter      | Description | Default |
|----------------|-------------|---------|
| `nack.enabled` | Enable Nack | `true`  |

For more NACK configuration options, see the [NACK documentation](https://github.com/nats-io/k8s/tree/main/helm/charts/nack).

## Using external PostgreSQL with OpenFGA

To use an external PostgreSQL database with OpenFGA:

1. Create a secret with the PostgreSQL connection string:

```bash
kubectl create secret generic openfga-postgresql-client \
  --from-literal="uri=postgres://username:password@postgres-host:5432/dbname?sslmode=disable" \
  -n lfx
```

2. Configure OpenFGA in your values file:

```yaml
openfga:
  postgres:
    enabled: false
  datastore:
    existingSecret: openfga-postgresql-client
```

## Jaeger

Jaeger provides distributed tracing capabilities for the LFX platform.
It should be installed in a separate `observability` namespace.

### Jaeger Prerequisites

Add the Jaeger Helm repository:

```bash
helm repo add jaegertracing https://jaegertracing.github.io/helm-charts
helm repo update
```

### Installing Jaeger

Install Jaeger using the all-in-one chart (suitable for development/testing):

```bash
helm install jaeger jaegertracing/jaeger \
  -n observability \
  --create-namespace \
  --set allInOne.enabled=true \
  --set agent.enabled=false \
  --set collector.enabled=false \
  --set query.enabled=false \
  --set storage.type=memory \
  --set provisionDataStore.cassandra=false
```

### Set Helm Values

Either update `charts/lfx-platform/values.yaml` directly or create a new
`tracing-values.yaml` file with the following values to enable traces to
be sent to Jaeger.

#### Traefik Values

```yaml
traefik:
  tracing:
    otlp:
      enabled: true
```

#### OpenFGA Values

```yaml
openfga:
  telemetry:
    trace:
      enabled: true
```

#### Heimdall Values

```yaml
heimdall:
  env:
    HEIMDALLCFG_TRACING_ENABLED: "true"
```

### Upgrade Helm Deployment

Then upgrade the helm deployment.

```bash
helm upgrade lfx-platform charts/lfx-platform
```

If using a values file, pass it to the command:

```bash
helm upgrade -f tracing-values.yaml lfx-platform charts/lfx-platform
```

### Accessing Jaeger UI

To access the Jaeger UI locally:

```bash
kubectl port-forward -n observability svc/jaeger-query 16686:16686
```

Then open `http://localhost:16686` in your browser.
