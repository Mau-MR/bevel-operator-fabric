# Local Development & Image Testing Guide

This guide describes how to build, push, and deploy local images of the HLF Operator for testing.

## Prerequisites

*   A local Kubernetes cluster (e.g., [Kind](https://kind.sigs.k8s.io/) or [Minikube](https://minikube.sigs.k8s.io/)).
*   A local Docker registry running at `localhost:5001`.
*   `kubectl` and `helm` installed.

## 1. Build and Push the Operator Image

Use the following command to build the operator binary and Docker image, then push it to your local registry.

```bash
# Build the binary with CGO disabled (matching the Alpine-based Dockerfile)
CGO_ENABLED=0 go build -o hlf-operator main.go

# Build and tag the image
IMAGE_TAG=localhost:5001/hlf-operator:v1.9.0-fix-crds
docker build -t $IMAGE_TAG .

# Push to local registry
docker push $IMAGE_TAG

# Clean up local binary
rm hlf-operator
```

## 2. Deploy with Helm

The Helm chart in `chart/hlf-operator` has been updated to use the local image by default in `values.yaml`. 

To install or upgrade the operator using this local image:

```bash
# Update local dependencies if needed
helm dependency update chart/hlf-operator

# Install/Upgrade the operator
helm upgrade --install hlf-operator ./chart/hlf-operator \
  --namespace hlf-operator --create-namespace \
  --set image.repository=localhost:5001/hlf-operator \
  --set image.tag=v1.9.0-fix-crds
```

## 3. Verify the Deployment

Check if the pod is running and using the correct image:

```bash
kubectl get pods -n hlf-operator
kubectl get pod -n hlf-operator -l app.kubernetes.io/name=hlf-operator -o jsonpath='{.items[0].spec.containers[0].image}'
```

## 4. Testing the Fix

The fix specifically addresses a race condition when processing certificates. You can test this by creating a `FabricIdentity` using a `secretRef` for the CA TLS certificate:

```yaml
apiVersion: hlf.kungfusoftware.es/v1alpha1
kind: FabricIdentity
metadata:
  name: my-identity
spec:
  # ... other specs ...
  catls:
    cacert: ""
    secretRef:
      name: my-ca-tls-secret
      key: tls.crt
```

Monitor the operator logs to ensure the identity transitions to `READY` without certificate processing errors.
