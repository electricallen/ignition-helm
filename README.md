# ![Ignition Helm Icon](assets/icon.svg) ignition-helm

Simple helm chart for deploying Ignition into Kubernetes

## Installation

### Add helm repo and install (preferred method)

> [!IMPORTANT]  
> You must edit `values.yaml` to accept the EULA

Either copy/paste the contents or run:

```sh
wget -O values.yaml https://raw.githubusercontent.com/electricallen/ignition-helm/main/charts/ignition/values.yaml
```

Then run:

```sh
helm repo add ignition-helm https://electricallen.github.io/ignition-helm
helm upgrade --install ignition ignition-helm/ignition -f values.yaml
```

### Installing locally

```sh
git clone https://github.com/electricallen/ignition-helm.git
```

Edit `./ignition-helm/charts/ignition/values.yaml` then run:

```
helm upgrade --install ignition ./ignition-helm/charts/ignition
```

## Accessing Ignition

### Port Forward (simplest, temporary)
* `kubectl port-forward svc/ignition 8088:8088`
* Gateway available at `localhost:8088`