# Setup / Deploy

## Before webinar
- 2 hosts - one server and one node
- install server before webinar starts
- connect to server, set auth
- pull images down to host/server
  - look at current running env to get latest image names/tags

## During webinar
- add host via `docker run`
- install k8s
- copy kubectl locally

# Pods

- explain pods (slide)

## Shell Pod
- show pod YAML (shell)
- demonstrate `kubectl apply -f`
- demonstrate `kubectl get|describe`
- demonstrate `kubectl exec|logs`

## nginx pod
- run it
- describe it
- curl it from shell pod
- demonstrate `kubectl delete`
- talk about how pods don't restart automatically

# Deployments

- explain deployments (slide)

## nginx deployment
- make sure no nginx pods are running
- show deployment yaml
- run it
- show pods
- demonstrate `kubectl edit`
  - change image from 1.13 to 1.12
  - show rolling update
  - demonstrate `kubectl rollout status deploy/nginx`
  - demonstrate `kubectl rollout undo`
  - talk about how deployments make it easy to manage the underlying components
  - show making deployment from `kubectl run`

  # Replication Controllers and Replica Sets

  - explain how RCs used to define RS and then Pods
  - explain that deployment covers it all now, with deployment creating RS to manage pods

  # Services

  - so you have a pod or group of pods...now what?
  - show service yaml
  - run it
  - explain service gives stable endpoint to pods and acts like load balancer
  - show DNS endpoint from shell pod and curl it
    - nginx.default.svc.cluster.local
    - delete service
    - create service via 'kubectl expose' with NodePort
    - connect to it
    - edit it and change type to LoadBalancer

    # Config Maps

    - show slide
    - explain that they remove the need for data containers that hold config files
    - can be environment variables, volumes, or files
    - show test-cm with key/value
    - create it
    - demonstrate `kubectl get configmap/test-cm`
    - show cm-as-env
    - create it
    - shell into it and echo $PRIMARY_KEY and $SECONDARY_KEY
    - show cm-as-volume
    - create it
    - shell into it and look at contents of /etc/config
    - what about a practical example?
    - show string-cm
    - create it
    - show cm-as-file and explain the mountPath and subPath
    - create it
    - shell into it and look at kibana.yml in /usr/share/kibana/config/kibana.yml
    - what happens when we need to change it?
    - edit test-cm and add key3/value3
    - shell into volume-cm-pod and show that key3 appears in /etc/config
    - edit kibana-config-v1
    - show that it does _not_ update the file
    - wait.
    - treat configmaps as versioned entities
    - edit kibana-config-v1 and change to v2
    - create it
    - edit kibana deploy and change v1 to v2
    - shell into it and see that the config has updated
    - how can we combine these?
    - show test-cm-string.yaml
    - apply it
    - shell into volume-cm-pod
    - ls /etc/config
    - cat /etc/config/httpd.conf
    - exit
    - edit configmap/test-cm
    - shell back into the pod
    - note that it didn't change
    - wait 
    - note that it does change
    - explain that this is an excellent way to update configurations for applications that automatically refresh their config on change, or applications that can process a signal to reload their config

    # Ingress

    - show slide
    - explain that ingress acts as a router, routing by host or path or port
    - multiple ingresses on multiple ports or single ingress with multiple routes
    - show/explain ingress.yaml
    - apply it
    - show in GUI that it's creating ingress-lbs listening on designated ports
    - connect to ports (one works, one doesn't)
    - create new service named 'proxy'


