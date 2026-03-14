# n8n Helm Chart

This Helm chart deploys n8n workflow automation platform with PostgreSQL database.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+

## Installation

To install the chart with the release name `n8n`:

```bash
helm repo add n8n https://your-helm-repo
helm install n8n n8n/n8n
```

## Uninstallation

To uninstall/delete the `n8n` deployment:

```bash
helm delete n8n
```

## Configuration

The following table lists the configurable parameters of the n8n chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of n8n replicas | `1` |
| `image.repository` | Container image repository | `n8nio/n8n` |
| `image.tag` | Container image tag | `1.24.0` |
| `image.pullPolicy` | Container image pull policy | `IfNotPresent` |
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `5678` |
| `ingress.enabled` | Enable ingress | `false` |
| `resources` | Resource limits and requests | See values.yaml |
| `persistence.enabled` | Enable persistence | `true` |
| `persistence.size` | Size of persistent volume | `10Gi` |
| `persistence.storageClass` | Storage class for persistent volume | `""` |
| `postgresql.enabled` | Enable PostgreSQL | `true` |
| `postgresql.auth.username` | PostgreSQL username | `n8n` |
| `postgresql.auth.database` | PostgreSQL database name | `n8n` |
| `postgresql.primary.persistence.size` | PostgreSQL PVC size | `10Gi` |

## Values

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
helm install n8n n8n/n8n --set persistence.size=20Gi
```

Alternatively, a YAML file that specifies the values for the parameters can be provided while installing the chart. For example:

```bash
helm install n8n n8n/n8n -f values.yaml
```

## Persistence

n8n requires persistent storage to maintain:
- Workflow data
- User preferences
- Encrypted credentials
- Execution history

The chart supports persistent storage using Kubernetes PersistentVolumeClaim (PVC). By default, persistence is enabled with a size of 10Gi.

### Configuring Persistence

1. Basic configuration (uses default storage class):
```yaml
persistence:
  enabled: true
  size: 10Gi
```

2. Using a specific storage class:
```yaml
persistence:
  enabled: true
  size: 10Gi
  storageClass: "my-storage-class"
```

3. Custom access modes:
```yaml
persistence:
  enabled: true
  size: 10Gi
  accessModes:
    - ReadWriteOnce
    - ReadOnlyMany
```

### Disabling Persistence

To disable persistence and use temporary storage:
```yaml
persistence:
  enabled: false
```

## Database Configuration

The chart supports two database configurations:

### 1. Internal PostgreSQL (Default)

By default, the chart deploys a PostgreSQL instance using the Bitnami PostgreSQL chart. The database credentials are automatically generated and stored in a Kubernetes secret.

To use the default configuration:
```yaml
postgresql:
  enabled: true
  auth:
    username: n8n
    database: n8n
    # Password will be auto-generated
```

To retrieve the auto-generated password:
```bash
kubectl get secret n8n-postgresql -o jsonpath="{.data.postgres-password}" | base64 --decode
```

### 2. External PostgreSQL

To use an external PostgreSQL instance, disable the internal PostgreSQL and configure the external connection:

```yaml
postgresql:
  enabled: false

externalPostgresql:
  host: "your-postgres-host"
  port: 5432
  database: n8n
  username: n8n
  secretName: my-postgres-secret
  secretKey: postgres-password
  ssl:
    enabled: false
    rejectUnauthorized: false
```

#### Setting up External PostgreSQL

1. Create a secret with your database password:
```bash
kubectl create secret generic my-postgres-secret \
  --from-literal=postgres-password='your-secure-password'
```

2. Configure SSL (optional):
```yaml
externalPostgresql:
  ssl:
    enabled: true
    rejectUnauthorized: true  # For strict SSL verification
```

## Security

### Basic Authentication

The chart enables basic authentication by default. The credentials are:
- Username: `admin`
- Password: Auto-generated and stored in a secret

To retrieve the auto-generated basic auth password:
```bash
kubectl get secret n8n-basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode
```

### Database Security

- Internal PostgreSQL: Uses auto-generated secure passwords
- External PostgreSQL: Requires manual secret creation
- SSL support for external PostgreSQL connections

## Troubleshooting

### Database Connection Issues

1. Check if the PostgreSQL pod is running:
```bash
kubectl get pods -l app.kubernetes.io/name=postgresql
```

2. Check PostgreSQL logs:
```bash
kubectl logs -l app.kubernetes.io/name=postgresql
```

3. Verify database credentials:
```bash
kubectl get secret n8n-postgresql -o yaml
```

### Access Issues

1. Check if the service is running:
```bash
kubectl get svc n8n
```

2. Check n8n logs:
```bash
kubectl logs -l app.kubernetes.io/name=n8n
```

3. Verify basic auth credentials:
```bash
kubectl get secret n8n-basic-auth -o yaml
```

### Storage Issues

1. Check PVC status:
```bash
kubectl get pvc -l app.kubernetes.io/name=n8n
```

2. Check PV status:
```bash
kubectl get pv
```

3. Check storage class:
```bash
kubectl get storageclass
``` 