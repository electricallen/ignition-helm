# ignition-helm
![Ignition Helm Icon](assets/icon.svg)

Simple helm chart for deploying Ignition into Kubernetes. This was primarily made for a learning exercise and to accelerate some testing while IA works on their official Helm release.

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

The example shown in the comments provisions a 1 GiB [`local-path`](https://github.com/rancher/local-path-provisioner) volume. This storage class comes pre-installed in k3s and Rancher desktop clusters, but can be [installed to other Kubernetes clusters manually](https://github.com/rancher/local-path-provisioner?tab=readme-ov-file#deployment). 

> [!CAUTION]
> Volumes using the `local-path`, `local`, or `hostPath` storage classes all come with limitations and are not suited for production Ignition deployments
>   `hostPath` has [many security risks](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath) and can cause data loss if pods are rescheduling to a new node. `local` prevents pods from being scheduled to a new node, and does not support dynamic provisioning. `local-path` creates either `local` or `hostPath` volumes, and is subject to the limitations of both.
>
> Use dedicated storage for production use. I've been told using block storage eliminates some issues seen with file storage classes like NFS.

## HTTPS

There are many ways to use HTTPS for the gateway web page. Among them:

1. Connect to the pod using the HTTPS port (default `:8043`)
    * Create an ingress to reach the pod on this port
    * Self sign or generate a CA-signed certificate and upload through the web portal ("the old fashioned way")
1. Connect to the pod using the HTTP port and have Kubernetes resources manage TLS
    * Install an ingress controller such as `traefik`, and optionally a certificate manager such as `cert-manager`
    * These services can generate, store, and serve certificates in front of the gateway pod

The second approach centralizes certificate management, which can be reused for other applications in the cluster. 

### Traefik, cert-manager, Let's Encrypt, and ACME

At the risk of veering off topic, the second approach is demonstrated here using one potential collection of tools. There are many different tools you can use to accomplish TLS outside the pod, and all should work with this helm chart provided only one ingress is used. 

For this approach, the following are required:

1. A domain 
1. The ability to issue [ACME challenges](https://letsencrypt.org/docs/challenge-types/)

#### Steps

1. Install `traefik` on the cluster. This comes pre-installed on k3s, but can be [installed using helm](https://doc.traefik.io/traefik/getting-started/install-traefik/#use-the-helm-chart)
1. [Install `cert-manager`](https://cert-manager.io/docs/installation/helm/#2-install-cert-manager)
1. Create a `ClusterIssuer` resource and any necessary secrets. 
    * Here is an example using the ACME staging URL and a cloudflare DNS-01 challenge with a zoned API token. See the `cert-manager` docs for [more issuer configuration examples](https://cert-manager.io/docs/configuration/issuers/).
    * Create this file as `manifest.yaml`:
    ```yaml
    apiVersion: cert-manager.io/v1
    kind: ClusterIssuer
    metadata:
    name: letsencrypt-staging
    spec:
    acme:
        email: yourEmailHere@domain.tld
        profile: tlsserver
        server: https://acme-staging-v02.api.letsencrypt.org/directory
        privateKeySecretRef:
        name: cluster-issuer-account-key
        solvers:
        - dns01:
            cloudflare:
            apiTokenSecretRef:
                name: cloudflare-api-token-secret
                key: api-token
    ---
    apiVersion: v1
    kind: Secret
    metadata:
    name: cloudflare-api-token-secret
    type: Opaque
    stringData:
    api-token: Add secret here!
    ```
    * Create these resources with `kubectl apply -f manifest.yaml -n cert-manager`

> [!NOTE]
> The `secret` resources must be in the `cert-manager` default namespace, which is typically `cert-manager`
> See [the docs here](https://cert-manager.io/docs/configuration/#cluster-resource-namespace)

4. Configure the ingress in `values.yaml` to request a certificate using the new `ClusterIssuer`:
    ```yaml
    ingress:
      enabled: true
      className: traefik
      annotations:
        cert-manager.io/cluster-issuer: letsencrypt-staging
        traefik.ingress.kubernetes.io/router.entrypoints: websecure
      portName: http
      hosts:
        - host: ignition.yourdomain.tld
          paths:
            - path: /
              pathType: Prefix
      tls:
        - hosts:
            - ignition.yourdomain.tld
          secretName: ignition-tls
    ```
5. Apply with `helm upgrade --install ignition electricallen/ignition -f values.yaml` 

This will request a certificate for `ignition.yourdomain.tld`, complete the challenge, and then use that certificate for HTTP requests. Traefik will terminate TLS, and the pod will only see HTTP traffic. Subsequent ingresses can create their own certificates by adding a `tls` section.