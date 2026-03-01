# appflowy

![Version: 0.0.1](https://img.shields.io/badge/Version-0.0.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

Helm chart for [AppFlowy Cloud](https://github.com/AppFlowy-IO/AppFlowy-Cloud) — a modern, bitnami-free alternative. Uses [Valkey](https://valkey.io/) instead of Redis, and expects PostgreSQL and S3-compatible storage to be provided externally.

## External dependencies

This chart does **not** bundle PostgreSQL or S3 storage. You need to provide them separately:

| Dependency | Recommendation | Notes |
|------------|---------------|-------|
| **PostgreSQL** | [CloudNativePG](https://cloudnative-pg.io/) | Requires `pgvector` and `pgcrypto` extensions |
| **S3 storage** | Any S3-compatible (MinIO, RustFS, AWS S3) | Set `global.s3.*` values accordingly |

## Chart dependencies

| Repository | Name | Version |
|------------|------|---------|
| https://valkey.io/valkey-helm/ | valkey | 0.9.3 |
| _(local)_ | appflowy-cloud | 0.1.0 |
| _(local)_ | appflowy-gotrue | 0.1.0 |
| _(local)_ | appflowy-admin | 0.1.0 |
| _(local)_ | appflowy-worker | 0.1.0 |
| _(local)_ | appflowy-web | 0.1.0 |
| _(local)_ | appflowy-ai | 0.1.0 |

## Quick start

```bash
helm dependency update .
helm install appflowy . -f my-values.yaml -n appflowy --create-namespace
```

## Image versions

| Component | Image | Tag |
|-----------|-------|-----|
| Cloud API | `appflowyinc/appflowy_cloud` | 0.12.3 |
| GoTrue | `appflowyinc/gotrue` | 0.12.3 |
| Admin | `appflowyinc/admin_frontend` | 0.12.6 |
| Worker | `appflowyinc/appflowy_worker` | 0.12.3 |
| Web | `appflowyinc/appflowy_web` | 0.10.8 |
| AI | `appflowyinc/appflowy_ai` | 0.12.3 |
| Valkey | `valkey/valkey` | 9.0.1 |

## Required secrets

When `global.secret.create: true` (default), the chart generates a Kubernetes Secret. The following values **must** be provided — the chart will fail to render without them:

| Key | Description |
|-----|-------------|
| `global.secret.jwt.secret.value` | JWT signing secret (use a random 32+ char string) |
| `global.secret.postgres.adminPassword.value` | PostgreSQL admin password |
| `global.secret.postgres.appflowy.postgresPassword.value` | PostgreSQL password for appflowy user |
| `global.secret.postgres.gotrue.postgresPassword.value` | PostgreSQL password for gotrue user |
| `global.secret.gotrue.adminPassword.value` | GoTrue admin password |
| `global.secret.s3.secret.value` | S3 secret key |

Optional secrets (can be left empty):

| Key | Description |
|-----|-------------|
| `global.secret.smtp.password.value` | SMTP password (required if sending emails) |
| `global.secret.oauth.*.value` | OAuth provider secrets (Google, Discord, GitHub, Apple) |
| `global.secret.ai.openAIAPIKey.value` | OpenAI API key (only if `global.ai.enabled: true`) |

Set `global.secret.create: false` to manage the Secret externally (e.g. via SealedSecrets or SOPS).

## Values

### Global

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| global.scheme | string | `"http"` | `http` or `https` |
| global.externalHost | string | `"appflowy.mydomain.com"` | Hostname for clients to reach the API |
| global.externalPort | int | `80` | Port for clients to reach the API |

### Ingress

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| global.ingress.nginx.enabled | bool | `true` | Create Ingress for NGINX |
| global.ingress.nginx.className | string | `"nginx"` | NGINX ingress class |
| global.ingress.nginx.tls.enabled | bool | `false` | Enable TLS |
| global.ingress.nginx.tls.secretName | string | `""` | TLS secret name |
| global.ingress.traefik.enabled | bool | `false` | Create IngressRoute for Traefik |
| global.ingress.traefik.entryPoint | string | `"web"` | HTTP entry point |
| global.ingress.traefik.secureEntryPoint | string | `"websecure"` | HTTPS entry point |

### Database (external)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| global.database.host | string | `"appflowy-postgres"` | PostgreSQL host |
| global.database.name | string | `"appflowy"` | Database name |
| global.database.port | int | `5432` | Database port |
| global.database.username | string | `"appflowy"` | Database user |
| global.database.adminUsername | string | `"postgres"` | Admin user (for setup job) |

### Redis / Valkey

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| global.redis.uri | string | `"redis://appflowy-valkey:6379"` | Redis-compatible URI (Valkey uses `redis://` protocol) |

### S3 (external)

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| global.s3.createBucket | string | `"true"` | Create bucket on startup |
| global.s3.useMinio | string | `"true"` | Using MinIO or S3-compatible service |
| global.s3.bucket | string | `"appflowy"` | Bucket name |
| global.s3.minioUrl | string | `"http://appflowy-minio:9000"` | S3-compatible endpoint URL |
| global.s3.accessKey | string | `"minioadmin"` | S3 access key |
| global.s3.region | string | `"us-east-1"` | S3 region |

### Valkey

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| valkey.enabled | bool | `true` | Deploy Valkey |
| valkey.auth.enabled | bool | `false` | Enable authentication |
| valkey.replica.enabled | bool | `false` | Enable replicas |
| valkey.fullnameOverride | string | `"appflowy-valkey"` | Service name |

### Setup job

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| setupJob.enabled | bool | `false` | Run pgvector extension setup. Not needed if your PostgreSQL already has the extension. |
| setupJob.postgresConnectTimeout | int | `60` | Connection timeout in seconds |
| setupJob.backoffLimit | int | `5` | Job retry limit |
