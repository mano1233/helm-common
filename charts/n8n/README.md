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

## Secret Management

This chart uses Kubernetes secrets for managing sensitive data. All sensitive values are expected to be stored in external secrets that are not managed by this chart. Here's how to create the required secrets:

### Basic Authentication and JWT Secret

```bash
kubectl create secret generic n8n-auth \
  --from-literal=basic-auth-password='your-secure-password' \
  --from-literal=jwt-secret='your-jwt-secret'
```

### SMTP Password

```bash
kubectl create secret generic n8n-email \
  --from-literal=smtp-password='your-smtp-password'
```

### LDAP Bind Password (if LDAP is enabled)

```bash
kubectl create secret generic n8n-ldap \
  --from-literal=bind-password='your-ldap-password'
```

### SAML Certificate and Key (if SAML is enabled)

```bash
kubectl create secret generic n8n-saml \
  --from-file=saml-cert=path/to/cert.pem \
  --from-file=saml-key=path/to/key.pem
```

### External Secrets Provider Credentials (if external secrets are enabled)

```bash
kubectl create secret generic n8n-external-secrets \
  --from-literal=aws-access-key-id='your-access-key' \
  --from-literal=aws-secret-access-key='your-secret-key'
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

Note: Database credentials are managed by the PostgreSQL subchart.

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
| `n8n.security.basicAuth.active` | Enable basic authentication | `false` |
| `n8n.security.basicAuth.user` | Basic auth username | `admin` |
| `n8n.security.basicAuth.password` | Basic auth password (leave empty to use secret) | `""` |
| `n8n.security.basicAuth.secretName` | Secret name for basic auth password | `{{ include "n8n.fullname" . }}-basic-auth` |
| `n8n.security.basicAuth.secretKey` | Secret key for basic auth password | `basic-auth-password` |
| `n8n.security.jwt.secret` | JWT secret (leave empty to use secret) | `""` |
| `n8n.security.jwt.secretName` | Secret name for JWT secret | `{{ include "n8n.fullname" . }}-jwt` |
| `n8n.security.jwt.secretKey` | Secret key for JWT secret | `jwt-secret` |
| `n8n.security.jwt.expiresIn` | JWT token expiration time | `7d` |

#### Email Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.email.mode` | Email mode (smtp or ses) | `smtp` |
| `n8n.email.smtp.host` | SMTP host | `smtp.gmail.com` |
| `n8n.email.smtp.port` | SMTP port | `587` |
| `n8n.email.smtp.user` | SMTP username | `""` |
| `n8n.email.smtp.password` | SMTP password (leave empty to use secret) | `""` |
| `n8n.email.smtp.secretName` | Secret name for SMTP password | `{{ include "n8n.fullname" . }}-smtp` |
| `n8n.email.smtp.secretKey` | Secret key for SMTP password | `smtp-password` |
| `n8n.email.smtp.sender` | SMTP sender email | `""` |

#### Features Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.features.externalHooks` | Enable external hooks | `false` |
| `n8n.features.externalSecrets` | Enable external secrets | `false` |
| `n8n.features.sourceControl` | Enable source control | `false` |
| `n8n.features.queueMode` | Enable queue mode | `false` |

#### External Secrets Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `n8n.externalSecrets.provider` | External secrets provider | `aws` |
| `n8n.externalSecrets.region` | AWS region | `us-east-1` |
| `n8n.externalSecrets.accessKeyId` | AWS access key ID (leave empty to use secret) | `""` |
| `n8n.externalSecrets.secretAccessKey` | AWS secret access key (leave empty to use secret) | `""` |
| `n8n.externalSecrets.secretName` | Secret name for AWS credentials | `{{ include "n8n.fullname" . }}-aws` |
| `n8n.externalSecrets.accessKeyIdKey` | Secret key for AWS access key ID | `aws-access-key-id` |
| `n8n.externalSecrets.secretAccessKeyKey` | Secret key for AWS secret access key | `aws-secret-access-key` |

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
| `n8n.userManagement.ldap.bindDn` | LDAP bind DN | `""` |
| `n8n.userManagement.ldap.password` | LDAP bind password (leave empty to use secret) | `""` |
| `n8n.userManagement.ldap.secretName` | Secret name for LDAP password | `{{ include "n8n.fullname" . }}-ldap` |
| `n8n.userManagement.ldap.secretKey` | Secret key for LDAP password | `ldap-password` |
| `n8n.userManagement.saml.enabled` | Enable SAML | `false` |
| `n8n.userManagement.saml.secretName` | Secret name for SAML credentials | `n8n-saml` |
| `n8n.userManagement.saml.certKey` | Secret key for SAML certificate | `saml-cert` |
| `n8n.userManagement.saml.keyKey` | Secret key for SAML key | `saml-key` |
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
| `postgresql.enabled` | Enable PostgreSQL deployment | `true` |
| `postgresql.auth.username` | PostgreSQL username | `n8n` |
| `postgresql.auth.password` | PostgreSQL password (leave empty for auto-generated) | `""` |
| `postgresql.auth.database` | PostgreSQL database name | `n8n` |
| `postgresql.auth.secretName` | Secret name for PostgreSQL password (leave empty for default) | `""` |
| `postgresql.auth.secretKey` | Secret key for PostgreSQL password | `postgres-password` |
| `postgresql.primary.persistence.size` | PostgreSQL storage size | `10Gi` |
| `postgresql.primary.persistence.storageClass` | PostgreSQL storage class | `""` |

Note: By default, the PostgreSQL password is automatically generated by the Bitnami chart and stored in a secret named `{release-name}-postgresql` with the key `postgres-password`. You can override this by setting a custom password or specifying a different secret name.

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

## Notes

- For all secret configurations, you have three options:
  1. Leave the password/secret field empty to use the default secret (auto-generated or specified)
  2. Set a direct password/secret value in the values file
  3. Use a custom secret by specifying `secretName` and `secretKey`

- PostgreSQL password is auto-generated by default when using Bitnami's PostgreSQL chart. You can:
  1. Leave `postgresql.auth.password` empty to use the auto-generated password
  2. Set a custom password in `postgresql.auth.password`
  3. Use a custom secret by setting `postgresql.auth.secretName` and `postgresql.auth.secretKey`

- When using custom secrets, make sure they exist in the cluster before deploying the chart. 