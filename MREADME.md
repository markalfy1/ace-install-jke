## Here's a step-by-step guide to manually deploy IBM ACE 13.x Integration Server on Google Kubernetes Engine (GKE), including deploying a BAR file via REST API.
## Installing cert-manager in your Kubernetes cluster
run this commands:
```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.17.2/cert-manager.yaml
```
- Verify cert-manger NS.
```
kubectl get pods --namespace cert-manager
```
- Patch Cert-Manger deployment
```bash
kubectl patch deployment \
  cert-manager \
  --namespace cert-manager  \
  --type='json' \
  -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--enable-certificate-owner-ref"}]'

```
## Step 1: Set up your GKE environment
Make sure you have:

- A running GKE cluster
- `kubectl` configured to access the cluster
- `Helm` installed (optional for Operator, not required here)

## Step 2: Create a Kubernetes namespace
We'll create a namespace called `ace` to keep resources organized:
```bash
kubectl create namespace ace
```
## Step 3: Create IBM Entitlement Key secret

Create a secret with your IBM container registry entitlement key to pull IBM ACE images:
```bash
kubectl create secret docker-registry ibm-entitlement-key \
  --docker-server=cp.icr.io \
  --docker-username=cp \
  --docker-password=<YOUR_ENTITLEMENT_KEY> \
  --namespace ace
```
Replace <`YOUR_ENTITLEMENT_KEY`> with your actual IBM entitlement key.

## Step 4: Create the Deployment manifest file
Create a file named `ace-integration-server.yaml`

## Step 5: Create the Service manifest file
Create a file named `ace-integration-server-service.yaml`

## Step 6: Apply the Deployment and Service manifests
Run these commands to create the Deployment and Service:
```bash
kubectl apply -f ace-integration-server.yaml
kubectl apply -f ace-integration-server-service.yaml
```
## Step 7: Verify Pods and Service are running
Check the status of your pods:
```bash
kubectl get pods -n ace
```
Check the service and note the EXTERNAL-IP (may take a few minutes to be assigned):
```bash
kubectl get svc -n ace
```
## Step 8: Access ACE Integration Server
Once the `EXTERNAL-IP` for the `ace-integration-server-service` is ready, you can connect to it on port `7600`.

## Step 9: Deploy BAR files to the ACE server via REST API

Create a s`cript deploy-bar.sh` to upload BAR files using the ACE REST API:
```bash
#!/bin/bash

# Replace with your external IP or hostname
ACE_HOST=<EXTERNAL-IP-OR-HOSTNAME>

# Replace with path to your BAR file
BAR_FILE_PATH=<path-to-your-bar-file>.bar

curl -X POST "http://$ACE_HOST:7600/mgmt/action/deploy" \
  -H "Content-Type: multipart/form-data" \
  -F "file=@${BAR_FILE_PATH}" \
  -F "name=ace-server" \
  -F "async=false"
```
Make it executable:
```bash
chmod +x deploy-bar.sh
```
Run the script:
```bash
./deploy-bar.sh

```
## Step 10: Monitor the ACE pod logs (optional)
To troubleshoot or monitor:
```bash
kubectl logs -f <ace-integration-server-pod-name> -n ace
```
You can find the pod name via:
```bash
kubectl get pods -n ace
```
---
## Summary
You have now deployed IBM ACE 13 Integration Server on GKE manually and know how to deploy BAR files using the REST API.