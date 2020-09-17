# Using Docker's K8s Labs

Docker provides a [lab environment][1] for Kubernetes with nodes that persist for four hours. You only need to log in with your Docker Hub ID.

## Bootstrapping

### Launching an Instance

Each system that you launch needs some basic commands installed. When you launch a new instance, execute the following on it before proceeding to instance-specific installations.

1. Click `Add New Instance` to launch the first instance.
2. Execute these common commands:
    ```
    # install curl and which
    yum install -y curl which

    # install K3d
    curl -s https://raw.githubusercontent.com/rancher/k3d/main/install.sh | bash

    # install Kustomize
    curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
    mv kustomize /usr/local/bin

    # install Helm 3
    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | VERIFY_CHECKSUM=false bash
    ```

### Launching a K3d Cluster

Each instance will host its own 1-node K3d cluster.

1. Create a new K3d cluster (replace `{name}` with the name of your cluster)
    ```
    k3d cluster create {name} -p 80:80@loadbalancer -p 443:443@loadbalancer --api-port=6443
    ```

## Launch the Working Cluster

Create a new instance and then create a K3d cluster on it by following the commands above.

## Launch the Rancher Cluster

Create a new instance and then create a K3d cluster on it by following the commands above. Continue with the Rancher installation steps:

1. Add Helm Repos
    ```
    helm repo add jetstack https://charts.jetstack.io
    helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
    helm repo update
    ```
2. Install cert-manager and wait for it to complete
    ```
    kubectl create namespace cert-manager
    helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.1 --set installCRDs=true
    kubectl rollout status deploy/cert-manager -n cert-manager
    kubectl get pods -n cert-manager
    ```
3. Set the `EXT_HOST` variable to the external name of your host, visible in the `URL` field above the terminal
4. Install Rancher and wait for it to complete
    ```
    kubectl create namespace cattle-system
    helm install rancher-stable/rancher --name rancher --namespace cattle-system --set hostname=$EXT_HOST
    ```
5. On the working cluster, edit `/etc/hosts`.
6. Enter the IP address of the Rancher cluster and the hostname you set for `EXT_HOST`. This will enable the working cluster to communicate directly with the Rancher cluster on its internal IP.
7. Visit `$EXT_HOST` in a browser


[1]: https://labs.play-with-k8s.com
