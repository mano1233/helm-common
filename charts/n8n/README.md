# n8n Helm Chart

This Helm chart deploys [n8n](https://n8n.io/) on a Kubernetes cluster.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- PV provisioner support in the underlying infrastructure (if persistence is enabled)

## Installing the Chart

To install the chart with the release name `my-n8n`:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm dependency update
helm install my-n8n .
```

## Configuration

The following table lists the configurable parameters of the n8n chart and their default values.

### Global Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of n8n replicas | `1` |
| `image.repository` | n8n image repository | `n8nio/n8n` |
| `image.tag` | n8n image tag | `1.28.0` |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full chart name | `""` |

### Service Account Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.name` | Service account name | `""` |

### Service Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Kubernetes service type | `ClusterIP` |
| `service.port` | Service port | `5678` |

### Ingress Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.className` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress hosts configuration | `[{host: "chart-example.local", paths: [{path: "/", pathType: "ImplementationSpecific"}]}]` |
| `ingress.tls` | Ingress TLS configuration | `[]` |

### Resource Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | `1000m` |
| `resources.limits.memory` | Memory limit | `1Gi` |
| `resources.requests.cpu` | CPU request | `500m` |
| `resources.requests.memory` | Memory request | `512Mi` |

### Autoscaling Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable autoscaling | `false` |
| `autoscaling.minReplicas` | Minimum replicas | `1` |
| `autoscaling.maxReplicas` | Maximum replicas | `100` |
| `autoscaling.targetCPUUtilizationPercentage` | Target CPU utilization | `80` |
| `autoscaling.targetMemoryUtilizationPercentage` | Target memory utilization | `80` |

### Persistence Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `persistence.enabled` | Enable persistence | `true` |
| `persistence.storageClass` | Storage class | `""` |
| `persistence.size` | Storage size | `10Gi` |
| `persistence.accessModes` | Access modes | `["ReadWriteOnce"]` |

### n8n Configuration

#### Database Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.database.type` | Database type | `postgresql` |
| `n8n.database.port` | Database port | `5432` |
| `n8n.database.schema` | Database schema | `public` |
| `n8n.database.ssl.enabled` | Enable SSL | `false` |
| `n8n.database.ssl.rejectUnauthorized` | Reject unauthorized SSL | `false` |

Note: When using the built-in PostgreSQL (enabled by default), the following database parameters are managed by the PostgreSQL subchart:
- `n8n.database.host` is automatically set to the PostgreSQL service name
- `n8n.database.database` is set to `postgresql.auth.database`
- `n8n.database.user` is set to `postgresql.auth.username`
- `n8n.database.password` is set to `postgresql.auth.password`

#### Server Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.server.host` | Server host | `"0.0.0.0"` |
| `n8n.server.port` | Server port | `5678` |
| `n8n.server.protocol` | Server protocol | `http` |
| `n8n.server.baseUrl` | Base URL | `""` |

#### Security Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.security.basicAuth.active` | Enable basic auth | `false` |
| `n8n.security.basicAuth.user` | Basic auth username | `admin` |
| `n8n.security.basicAuth.password` | Basic auth password | `admin` |
| `n8n.security.jwt.secret` | JWT secret | `your-jwt-secret` |
| `n8n.security.jwt.expiresIn` | JWT expiration | `7d` |

#### Email Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.email.mode` | Email mode | `smtp` |
| `n8n.email.smtp.host` | SMTP host | `smtp.example.com` |
| `n8n.email.smtp.port` | SMTP port | `587` |
| `n8n.email.smtp.user` | SMTP user | `your-email@example.com` |
| `n8n.email.smtp.pass` | SMTP password | `your-password` |
| `n8n.email.smtp.sender` | SMTP sender | `your-email@example.com` |

#### Features Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.features.externalHooks` | Enable external hooks | `false` |
| `n8n.features.externalSecrets` | Enable external secrets | `false` |
| `n8n.features.sourceControl` | Enable source control | `false` |
| `n8n.features.queueMode` | Enable queue mode | `false` |

#### Logging Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.logs.level` | Log level | `info` |
| `n8n.logs.output` | Log output | `console` |
| `n8n.logs.file.path` | Log file path | `/home/node/.n8n/logs` |
| `n8n.logs.file.maxSize` | Max log file size | `10m` |
| `n8n.logs.file.maxFiles` | Max log files | `5` |

#### User Management Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.userManagement.disabled` | Disable user management | `false` |
| `n8n.userManagement.smtp.enabled` | Enable SMTP for user management | `false` |
| `n8n.userManagement.ldap.enabled` | Enable LDAP | `false` |
| `n8n.userManagement.saml.enabled` | Enable SAML | `false` |
| `n8n.userManagement.twoFactor.enabled` | Enable 2FA | `false` |
| `n8n.userManagement.twoFactor.issuer` | 2FA issuer | `n8n` |

#### Workflow Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.workflows.defaultName` | Default workflow name | `"My Workflow"` |
| `n8n.workflows.callPolicy` | Workflow call policy | `"workflowFromSameOwner"` |
| `n8n.workflows.tags.enabled` | Enable workflow tags | `true` |
| `n8n.workflows.history.pruneData` | Prune workflow history | `true` |
| `n8n.workflows.history.maxAge` | Max history age | `24h` |

### PostgreSQL Configuration

The chart includes [Bitnami's PostgreSQL chart](https://github.com/bitnami/charts/tree/main/bitnami/postgresql) as a dependency. You can configure it using the following parameters:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `postgresql.enabled` | Enable PostgreSQL | `true` |
| `postgresql.auth.username` | PostgreSQL username | `n8n` |
| `postgresql.auth.password` | PostgreSQL password | `n8n` |
| `postgresql.auth.database` | PostgreSQL database | `n8n` |
| `postgresql.primary.persistence.size` | PostgreSQL storage size | `10Gi` |
| `postgresql.primary.persistence.storageClass` | PostgreSQL storage class | `""` |

For more configuration options, please refer to the [Bitnami PostgreSQL chart documentation](https://github.com/bitnami/charts/tree/main/bitnami/postgresql#parameters).

## Upgrading

### To 1.0.0

This is the first major release of the n8n Helm chart. If you're upgrading from a previous version, please review the configuration changes and test the upgrade in a non-production environment first.

## Uninstalling the Chart

To uninstall/delete the `my-n8n` deployment:

```bash
helm delete my-n8n
```

This command removes all the Kubernetes components associated with the chart and deletes the release.

## Contributing

Feel free to open issues and pull requests with improvements. 