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

    # install Kustomize
    curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
    mv kustomize /usr/local/bin

    # install Helm 3
    curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | VERIFY_CHECKSUM=false bash
    ```

## Install K3s

We'll start the class with Kubernetes commands pointed at the local cluster. For that we need to install [K3s][2].

```
# install K3s (ignore the error that appears)
curl -sfL https://get.k3s.io | INSTALL_K3S_SELINUX_WARN=true sh -s - --docker

# start K3s
k3s server --docker > /var/log/k3s.log 2>&1 &

# export KUBECONFIG and test connection
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
kubectl get nodes
```

## Install Rancher

1. Add Helm Repos
    ```
    helm repo add jetstack https://charts.jetstack.io
    helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
    helm repo update
    ```
2. Install cert-manager and wait for it to complete
    ```
    kubectl create namespace cert-manager
    helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.3 --set installCRDs=true
    kubectl rollout status deploy/cert-manager -n cert-manager
    kubectl get pods -n cert-manager
    ```
3. Set the `EXT_HOST` variable to the external name of your host, visible in the `URL` field above the terminal
    ```
    # example - your hostname will be different
    export EXT_HOST=ip172-18-0-10-bu460puj2b7000arid60.direct.labs.play-with-k8s.com
    ```
4. Install Rancher and wait for it to complete
    ```
    kubectl create namespace cattle-system
    helm install rancher rancher-latest/rancher --namespace cattle-system --set hostname=$EXT_HOST
    kubectl rollout status deploy/rancher -n cattle-system
    kubectl get pods -n cattle-system
    ```
7. Visit `$EXT_HOST` in a browser


[1]: https://labs.play-with-k8s.com
[2]: https://k3s.io


<!--
```
yum install -y curl which
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
mv kustomize /usr/local/bin
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | VERIFY_CHECKSUM=false bash
# install k3s
curl -sfL https://get.k3s.io | INSTALL_K3S_SELINUX_WARN=true sh -s - --docker
k3s server --docker > /var/log/k3s.log 2>&1 &
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
sleep 10
kubectl get nodes
# install cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
helm repo update
kubectl create namespace cert-manager
helm install cert-manager jetstack/cert-manager --namespace cert-manager --version v1.0.3 --set installCRDs=true
kubectl rollout status deploy/cert-manager -n cert-manager
kubectl get pods -n cert-manager
# install rancher
export EXT_HOST="ip172-18-0-13-bu4794mj2b7000arie3g.direct.labs.play-with-k8s.com"
kubectl create namespace cattle-system
helm install rancher rancher-latest/rancher --namespace cattle-system --set hostname=$EXT_HOST
kubectl rollout status deploy/rancher -n cattle-system
kubectl get pods -n cattle-system
```
-->