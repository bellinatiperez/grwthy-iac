# Grwthy IAC

Kustomize manifests for deploying **grwthy-web** and **grwthy-api** to K3s.

## Prerequisites

- K3s cluster with Traefik ingress controller
- kubectl with kustomize support
- Container images pushed to `ghcr.io/bellinatiperez/`

## Structure

```
base/
  web/          # Next.js frontend (port 3000)
  api/          # Node.js API (port 3001)
overlays/
  dev/          # namespace: grwthy-dev, host: dev.grwthy.com
  production/   # namespace: grwthy-prod, host: grwthy.com
```

## Usage

### Preview manifests

```bash
kubectl kustomize overlays/dev
kubectl kustomize overlays/production
```

### Deploy

```bash
# Create namespace
kubectl create namespace grwthy-dev

# Create secrets (replace with actual values)
kubectl -n grwthy-dev create secret generic grwthy-web-secret \
  --from-literal=NEXT_PUBLIC_SUPABASE_URL=your_url \
  --from-literal=NEXT_PUBLIC_SUPABASE_ANON_KEY=your_key

kubectl -n grwthy-dev create secret generic grwthy-api-secret \
  --from-literal=SUPABASE_URL=your_url \
  --from-literal=SUPABASE_SERVICE_KEY=your_key \
  --from-literal=DATABASE_URL=your_database_url

# Create configmaps
kubectl -n grwthy-dev create configmap grwthy-web-config \
  --from-literal=NEXT_PUBLIC_API_URL=https://dev.grwthy.com/api \
  --from-literal=NEXT_PUBLIC_CNPJ=00.000.000/0001-00

kubectl -n grwthy-dev create configmap grwthy-api-config \
  --from-literal=PORT=3001 \
  --from-literal=NODE_ENV=development

# Apply manifests
kubectl apply -k overlays/dev
```

### Production

```bash
kubectl create namespace grwthy-prod

# Create secrets and configmaps (same pattern as dev with production values)
# ...

kubectl apply -k overlays/production
```

## Environments

| | Dev | Production |
|---|---|---|
| Namespace | `grwthy-dev` | `grwthy-prod` |
| Host | `dev.grwthy.com` | `grwthy.com` |
| Image tag | `:dev` | `:latest` |
| Web replicas | 1 | 2 |
| API replicas | 1 | 2 |
| Requests | 64Mi / 100m | 128Mi / 200m |
| Limits | 128Mi / 200m | 256Mi / 500m |

## Services

| Service | Port | Health check |
|---|---|---|
| grwthy-web | 3000 | `GET /` |
| grwthy-api | 3001 | `GET /api/health` |
