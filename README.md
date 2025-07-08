# ignition-helm
![Ignition Helm Icon](assets/icon.svg)

Simple helm chart for deploying Ignition into Kubernetes

## Installation

> [!IMPORTANT]  
> You must accept Inductive Automation's [Ignition EULA](https://inductiveautomation.com/ignition/license) by setting the `eula.accepted` value true

### Quickstart

Add the repo and install. Passing `--set eula.accepted=true` constitutes acceptance of the [Ignition EULA](https://inductiveautomation.com/ignition/license).

```sh
helm repo add electricallen https://electricallen.github.io/ignition-helm
helm upgrade --install ignition electricallen/ignition --set eula.accepted=true
```

### Editing values

To modify helm values, download and edit `charts/ignition/values.yaml`, EG:

```sh
curl -O https://raw.githubusercontent.com/electricallen/ignition-helm/main/charts/ignition/values.yaml
```

Then create a new release with:

```sh
helm upgrade --install ignition electricallen/ignition -f values.yaml
```

### Updating the chart

To update to the latest version of the chart run:

```sh
helm repo update electricallen
```

## Networking

See the [Kubernetes docs on Service types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) for more details. 

### `kubectl port-forward`

By default the gateway is only accessible from inside the cluster. Remote hosts can reach this service using the `kubectl port-forward`, for example:

```
kubectl port-forward svc/ignition 8088:8088
```

The gateway is then available in browser or through designer at http://localhost:8088. 

### Ingress

The default approach is only available on machines that can run `kubectl` and the connection terminates once the command exits. A more practical approach is to create an [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to expose the service to hosts outside the cluster. The following is required to use an ingress:

* `ingress.enabled` = `true` in `values.yaml`
* DNS and/or routing to the hostnames or IPs defined in `ingress.hosts`
* An [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) resource installed on the cluster (**not** included in this repo)
    * Any ingress controller can be used, the default is configured for Traefik (which is pre-installed in k3s and can be [installed using helm](https://doc.traefik.io/traefik/getting-started/install-traefik/#use-the-helm-chart) elsewhere)

