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

To update to the latest version of the chart run:

```sh
helm repo update electricallen
```

To uninstall the release run:

```sh
helm uninstall ignition
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

## Persistent storage

By default, no volumes are configured and gateway data is lost when a pod is restarted. [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) can be created to persist gateway data between pod restarts. The following is required in order to use persistent volumes:

* A [Storage Class](https://kubernetes.io/docs/concepts/storage/storage-classes/) installed on the cluster (check with `kubectl get storageclass`)
* The `volumeClaimTemplates` section of `values.yaml` configured, specifically:
    * `volumeClaimTemplates[0].spec.resources.requests.storage` set to the amount of storage needed for each pod
    * `volumeClaimTemplates[0].spec.storageClassName` set to the storage class installed on the cluster

This example provisions a 1 GiB [`local-path`](https://github.com/rancher/local-path-provisioner) volume. The `local-path` storage class comes pre-installed in k3s and Rancher desktop clusters, but can be[ installed to other Kubernetes clusters manually](https://github.com/rancher/local-path-provisioner?tab=readme-ov-file#deployment). 

```yaml
volumeClaimTemplates: 
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOncePod"]
      resources:
        requests:
          storage: 1Gi
      storageClassName: local-path
```

> [!CAUTION]
> Volumes using the `local-path`, `local`, or `hostPath` storage classes all come with limitations and are not suited for production Ignition deployments
>   `hostPath` has [many security risks](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) and can cause data loss if pods are rescheduling to a new node. `local` prevents pods from being scheduled to a new node, and does not support dynamic provisioning. `local-path` creates either `local` or `hostPath` volumes, and is subject to the limitations of both.
>
> Use a dedicated off-cluster storage class in production, such as [`nfs`](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath), or the storage classes provided by your cloud provider (EG [AWS](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volume-types.html) or [Azure](https://learn.microsoft.com/en-us/azure/aks/concepts-storage#storage-classes))