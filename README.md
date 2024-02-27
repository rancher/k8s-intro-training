# Training Guide

## Kubernetes Options

Rancher gives you several options for running Kubernetes. You can use these or any other CNCF-certified Kubernetes distribution.

- [K3s](https://k3s.io) - Lightweight Kubernetes that works both in the datacenter and resource-constrained environments like IoT and the Edge.
- [RKE](https://rancher.com/products/rke/) - A complete installation of Kubernetes that runs inside of Docker containers.
- [RKE2](https://docs.rke2.io/) - The next generation of RKE, using ContainerD and FIPS compliant.
- [K3d](https://k3d.io) - Runs a multi-node Kubernetes cluster within Docker. Designed for local development operations.

## Deploy K3s

### Local Deployment

For this you'll need to be logged into a Linux system.

``` bash
curl -sfL https://get.k3s.io | sh -
```

### Remote Deployment over SSH

For this you'll need a Linux system you can SSH into. We'll use the `k3sup` command available from [k3sup.dev](https://k3sup.dev). The defaults assume that you're using `~/.ssh/id_rsa` as your key and connecting as the `root` user. If you need to change these or other defaults, see the output from `k3sup install --help`.

All `k3sup` needs to install K3s over SSH is the IP of the destination server. We're adding a release channel to make sure that we're running the `stable` release, and not the `latest`.

```bash
# replace this with the IP of your host

# Bash
export IP=10.68.0.143

# Fish
set -x IP 10.68.0.143

k3sup install --ip=$IP --k3s-channel=stable
```

Alternatively, to install a specific version:

``` bash
export VERSION=v1.19.4+k3s1
k3sup install --ip=$IP --k3s-version=$VERSION
```

You can add additional configuration. See `k3sup --help` and `k3s help server`, as well as the [Rancher Documentation](https://rancher.com/docs/k3s/)

``` bash
k3sup install --ip=$IP --k3s-version=$VERSION --local-path=config.a.yaml --context a
```

## Working With Kubernetes

### Using `kubectl`

If you're using a local installation of K3s, the `kubectl` binary already points to the configuration file, located at `/etc/rancher/k3s/k3s.yaml`.

If you're using a remote system installed via `k3sup`, the config was locally saved in the current working directory. Point the `KUBECONFIG` environment variable at it.

``` bash
# Bash
export KUBECONFIG=$(pwd)/kubeconfig

# Fish
set -x KUBECONFIG (pwd)/kubeconfig
```

Verify your installation and that you're pointed to the right cluster.

``` bash
kubectl get nodes
```

`kubectl` supports plugins that can help make things easier. I'll be using `kubectl neat` when showing manifests. You can install plugins with [krew](https://krew.sigs.k8s.io/).

You'll also want to start using aliases. See [this repository](https://github.com/ahmetb/kubectl-aliases) for a system of aliases that will make your life easier.

[Kui](https://kui.tools) is a wonderful utility for interacting with Kubernetes that gives you rich information for exploring the cluster and its workloads.

[Kubie](https://github.com/sbstp/kubie) is a utility for switching between Kubernetes clusters and contexts (namespaces) with isolation between terminal windows. It works great with Kui to keep it pointed at the right cluster.

``` bash
kubie ctx a  # will search KUBECONFIG for the a context
kubie ctx -f config.a.yaml  # will use the contents of this config
kubie ns training # will switch to the "training" namespace
```

I have these aliased:

```
alias kcc="kubie ctx"
alias ns="kubie ns"
```

### Pods

Smallest unit you can deploy in Kubernetes. The actual workload.

```bash
kubectl apply -f 01-pod/pod.yaml
kubectl logs myapp-pod
kubectl get po -w
kubectl delete po myapp-pod
```

Watch how Kubernetes tries to restart the Pod and then backs off the restart timer.

```
kubectl get po -w
```

### Deployment

Creates a ReplicaSet that in turn creates and manages one or more Pods

``` bash
kubectl create deploy nginx --image=nginx:1.19-alpine
kubectl get deploy
kubectl get replicaset
kubectl get po
```

Describe the pod, look at the image

``` bash
kubectl describe po/<pod name>
kubectl get po/<pod name> -o yaml | less
```

Scale the Deployment manually

``` bash
kubectl scale deploy/nginx --replicas=3
kubectl rollout status deploy/nginx
kubectl get deploy
kubectl get po
```

What happens if we upgrade with a bad image?

``` bash
kubectl set image deploy/nginx nginx=nginx:1.20-alpne
kubectl rollout status deploy/nginx
kubectl get po
kubectl rollout undo deploy/nginx
```

We can redo the upgrade from the manifest.

``` bash
kubectl edit deploy/nginx
```

### ConfigMaps

ConfigMaps allow us to override data within the container.

``` bash
kubectl exec -it <pod-name> -- ash
cat /usr/share/html/index.html
```

We can see that the HTML file is the default. We can override this directory with a ConfigMap

``` bash
cd 02-deployment/base
cat configs/index.html
cat deployment.yaml  # how is the ConfigMap created?
```

A tool called Kustomize, built into `kubectl` can assemble all of this from templates.

``` bash
cat kustomize.yaml
kustomize build . | less
```

We can see the ConfigMap was generated with a unique name. This forces a rolling update when the ConfigMap changes, which keeps us in line with GitOps and the repository being the source of truth.

Kustomize allows us to create and template content, reducing human error present when copying things between manifests.

We'll use this in a later section.

### Services

Let's deploy something that will show us interesting traffic.

``` bash
cd 03-rancher-demo
cat kustomization.yaml
kubectl apply -k .

# Bash
export PORT=$(kubectl get service/demo -o jsonpath='{.spec.ports[0].nodePort}')

# Fish
set -x PORT (kubectl get service/demo -o jsonpath='{.spec.ports[0].nodePort}')

# Use $IP from the top of this README
curl -I $IP:$PORT

# Open $IP:$PORT in a browser
```

We can demonstrate the benefits of Kustomize and a ConfigMap by editing `kustomization.yaml` and changing the value of `COW_COLOR`.

```bash
vi kustomization.yaml
# change COW_COLOR to pink and save

kubectl apply -k .
kubectl get po -w
```

If we look at the Deployment YAML now, we'll see that the `configMapRef` was updated to use the new ConfigMap that Kustomize created. This gives us a clear path for rolling back if we want. If we had updated in place, the Deployment wouldn't have changed, and we wouldn't be able to roll back the change to the configuration.

```bash
kubectl rollout undo deploy demo
```

The cows are yellow again.

### Ingress

``` bash
# still in 03-rancher-demo
kubectl apply -f ingress.yaml
kubectl get ingress
curl -I -H 'Host: rancher-demo.home.monach.us' http://$IP/
```

### Cleanup

Delete it all.

---

## Rancher

### Server Deploy

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
helm repo update

helm install cert-manager jetstack/cert-manager --namespace cert-manager --set installCRDs=true --create-namespace

kubectl get po -n cert-manager -w

# Bash
export HOSTNAME=rancher-training.home.monach.us

# Fish
set -x $HOSTNAME rancher-training.home.monach.us

helm install rancher rancher-stable/rancher --namespace cattle-system --set hostname=$HOSTNAME --create-namespace
```

### Walkthrough

- Cluster Explorer
- Apps
- Continuous Delivery

Remember Kustomize? We'll use it now.

- Show the kustomization files and how they work together
- Deploy with Fleet: `https://github.com/rancher/k8s-intro-training` with the `02-deployment` folder
