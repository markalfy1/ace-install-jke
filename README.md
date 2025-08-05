# IBM App Connect Enterprise (ACE) 13 Standalone & Dashboard Deployment on Google Cloud GKE

This repository contains the instructions and Kubernetes manifests to deploy **IBM App Connect Enterprise (ACE) version 13.0** standalone Integration Server and ACE Dashboard on Google Kubernetes Engine (GKE) using the IBM App Connect Operator.

---

## Prerequisites

- A Google Cloud GKE cluster up and running
- `kubectl` configured to connect to your GKE cluster
- `helm` version 3+ installed locally
- IBM Container Registry Entitlement Key ([Get your key]
- (Optional) cert-manager and Ingress controller installed for TLS support

---
## cert-manager

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.17.2/cert-manager.yaml

kubectl get pods --namespace cert-manager
```
then we need to patch cert-manager deployment by applying next

```bash
kubectl patch deployment \
  cert-manager \
  --namespace cert-manager  \
  --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--enable-certificate-owner-ref"}]'
```

## Step 1: Create Namespace and IBM Entitlement Secret

```bash
kubectl create namespace ace-ns

kubectl create secret docker-registry ibm-entitlement-key \
  --docker-server=cp.icr.io \
  --docker-username=cp \
  --docker-password="<YOUR_ENTITLEMENT_KEY>" \
  -n ace-ns

```
---

## Step 2: Install IBM App Connect Operator

```bash

helm repo add ibm-helm https://raw.githubusercontent.com/IBM/charts/master/repo/ibm-helm
helm repo update

helm install ibm-appconnect-operator ibm-helm/ibm-appconnect-operator \
  -n ace-ns \
  --set namespace="ace-ns" \
  --set operator.installMode="AllNamespaces"

```

## Step 3: Deploy Integration Server and Dashboard
Save the following manifests as integrationserver.yaml and dashboard.yaml respectively, then apply:

```bash
kubectl apply -f integrationserver.yaml
kubectl apply -f dashboard.yaml

```
## Step 4: Verify Deployments and Access Dashboard

```bash
kubectl get pods -n ace-ns
```
Get the dashboard URL:

```bash
kubectl get dashboard ace-dashboard -n ace-ns -o jsonpath='{.status.endpoints[0].uri}'

```
Open the URL in your browser.

## Optional: Configure Ingress with TLS
Ensure you have cert-manager and an ingress controller installed.

```bash
kubectl apply -f ace-dashboard-ingress.yaml
```

## Optional: Enable Keycloak Authentication
Modify dashboard.yaml to enable Keycloak:
```bash

spec:
  authentication:
    integrationKeycloak:
      enabled: true
      keycloakRealm: your-realm
      keycloakUrl: https://keycloak.example.com/auth
      keycloakClientId: ace-dashboard-client
  authorization:
    integrationKeycloak:
      enabled: true

```
Apply updated manifest:

```bash
kubectl apply -f dashboard.yaml
```
Configure Keycloak clients and realms accordingly.

## Troubleshooting
Ensure your IBM entitlement key is correct.

Pods may take a few minutes to become ready.

Check logs for pods if issues occur:
```bash
kubectl logs <pod-name> -n ace-ns
```
