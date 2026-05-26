# Operator Pre-requisites

Two Kubernetes operators must be installed before deploying Arcanna infra.
These are cluster-scoped and typically installed once by a platform team.

## ECK Operator (manages Elasticsearch + Kibana)

```bash
kubectl create -f https://download.elastic.co/downloads/eck/2.16.1/crds.yaml
kubectl apply -f https://download.elastic.co/downloads/eck/2.16.1/operator.yaml
# Wait for operator pod
kubectl rollout status deployment elastic-operator -n elastic-system
```

## CFK Operator (manages Kafka in KRaft mode)

```bash
# Install via Helm (Confluent's recommended method)
helm repo add confluentinc https://packages.confluent.io/helm
helm repo update
helm upgrade --install confluent-operator confluentinc/confluent-for-kubernetes \
  -n confluent --create-namespace
# Wait for operator pod
kubectl rollout status deployment confluent-operator -n confluent
```

## Verify CRDs exist

```bash
# ECK
kubectl get crd elasticsearches.elasticsearch.k8s.elastic.co
kubectl get crd kibanas.kibana.k8s.elastic.co

# CFK
kubectl get crd kafkas.platform.confluent.io
kubectl get crd kraftcontrollers.platform.confluent.io
```

Both should return resources before you sync the infra ArgoCD apps.
