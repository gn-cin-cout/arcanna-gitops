# Pre-deployment Secret Setup

ArgoCD cannot generate secrets dynamically. These secrets **must be created manually**
before syncing any ArgoCD Application. Run these commands once against your GKE cluster.

## Step 0 — Target cluster and namespace

```bash
kubectl config use-context <YOUR_GKE_CONTEXT>
kubectl create namespace arcanna --dry-run=client -o yaml | kubectl apply -f -
```

## Step 1 — PostgreSQL credentials

```bash
POSTGRES_PASSWORD=$(openssl rand -base64 32 | tr -d '/+=' | head -c 32)
kubectl create secret generic postgres-credentials \
  -n arcanna \
  --from-literal=user=arcanna \
  --from-literal=password="$POSTGRES_PASSWORD" \
  --from-literal=database=arcanna
echo "Postgres password (save it): $POSTGRES_PASSWORD"
```

## Step 2 — Redis credentials

```bash
REDIS_PASSWORD=$(openssl rand -base64 32 | tr -d '/+=' | head -c 32)
kubectl create secret generic redis-credentials \
  -n arcanna \
  --from-literal=password="$REDIS_PASSWORD"
echo "Redis password (save it): $REDIS_PASSWORD"
```

## Step 3 — Arcanna application tokens

```bash
kubectl create secret generic arcanna-app-credentials \
  -n arcanna \
  --from-literal=seal-token="$(openssl rand -base64 24)" \
  --from-literal=api-token="$(openssl rand -base64 24)" \
  --from-literal=rag-api-key="$(openssl rand -base64 24)" \
  --from-literal=monitoring-api-key="$(openssl rand -base64 24)" \
  --from-literal=monitoring-secret="$(openssl rand -base64 24)"
```

## Step 4 — GCR pull secret (requires key file from Arcanna/Siscale team)

```bash
# Obtain gcr-sa.json from the Arcanna team first.
kubectl create secret docker-registry gcr-pull-secret \
  -n arcanna \
  --docker-server=gcr.io \
  --docker-username=_json_key \
  --docker-password="$(cat /path/to/gcr-sa.json)" \
  --docker-email=sa@arcanna.ai
```

> **Note:** Steps 1–3 unlock Phase 1 (infra). Step 4 is only needed for Phase 2 (app services).

## Step 5 — Verify

```bash
for s in postgres-credentials redis-credentials arcanna-app-credentials gcr-pull-secret; do
  kubectl get secret $s -n arcanna >/dev/null 2>&1 \
    && echo "✅ $s" \
    || echo "❌ $s (missing)"
done
```
