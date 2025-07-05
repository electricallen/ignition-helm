# ignition-helm
Simple helm chart for deploying Ignition into Kubernetes

## Installation

### Installing locally

```sh
git clone <THIS REPO URL>
cd ./<THIS REPO NAME>/
cp example-values.yaml values.yaml
# Edit values.yaml
helm install ignition ./
```

## Accessing Ignition

### Port Forward (simplest, temporary)
* `kubectl port-forward svc/ignition 8088:8088`
* Gateway available at `localhost:8088`