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