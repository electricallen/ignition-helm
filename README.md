# ![Ignition Helm Icon](assets/icon.svg) ignition-helm

Simple helm chart for deploying Ignition into Kubernetes

## Installation

> [!IMPORTANT]  
> You must edit `values.yaml` to accept the EULA

### Add helm repo and install (preferred method)

Either copy/paste the contents or run:

```sh
wget -O values.yaml https://raw.githubusercontent.com/electricallen/ignition-helm/main/charts/ignition/values.yaml
```

Edit Then run:

```sh
helm repo add ignition-helm https://electricallen.github.io/ignition-helm
helm upgrade --install ignition ignition-helm/ignition -f values.yaml
```

### Cloning Git and installing from local chart

```sh
git clone https://github.com/electricallen/ignition-helm.git
```

Edit `./ignition-helm/charts/ignition/values.yaml` then run:

```
helm upgrade --install ignition ./ignition-helm/charts/ignition
```

## Networking

See the [Kubernetes docs on Service types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types) for more details. 

### Insecure ClusterIP (default)

By default, `values.yaml` is configured to create a `ClusterIP` service, and is only reachable from inside the cluster. Remote hosts can reach this service using the `kubectl port-forward`:

```
kubectl port-forward svc/ignition 8088:8088
```

Then you can access the gateway in browser or through designer at http://localhost:8088. 

### Ingress

The default approach is only available on machines that can run `kubectl` and the connection terminates once the command exits. 

A more durable approach is to create an [ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to expose the service to hosts outside the cluster. The following is required to use an ingress:

* `ingress.enabled` = `true` in `values.yaml`
* DNS and/or routing to the hostnames or IPs defined in `ingress.hosts`
* An [ingress controller](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) resource installed on the cluster (**not** included in this repo)
    * Any ingress controller can be used, the default is configured for Traefik
    * Traefik can be [installed using helm](https://doc.traefik.io/traefik/getting-started/install-traefik/#use-the-helm-chart); it is also preinstalled in k3s

Here is one example of `values.yaml` configured to make the gateway available at `http://ignition.mydomain.tld`:

```yaml
service:
  type: NodePort
  port: 8088
  targetPort: 8088 # Will also set GATEWAY_HTTP_PORT env var
  nodePort: 30080 

ingress:
  enabled: true
  className: traefik
  annotations: {}
  hosts:
    - host: ignition.mydomain.tld
      paths:
        - path: /
          pathType: Prefix
  tls: []
```

