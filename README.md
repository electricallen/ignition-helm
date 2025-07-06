# ignition-helm
Simple helm chart for deploying Ignition into Kubernetes

## Installation

### Installing locally

    ```sh
    git clone https://github.com/electricallen/ignition-helm.git
    cd ./ignition-helm
    ```
> [!IMPORTANT]  
> You must change `values` to accept the EULA

Edit `./charts/ignition/values.yaml` then run:

    ```
    helm install ignition ./charts/ignition
    ```

## Accessing Ignition

### Port Forward (simplest, temporary)
* `kubectl port-forward svc/ignition 8088:8088`
* Gateway available at `localhost:8088`