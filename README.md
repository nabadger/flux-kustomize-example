# Kustomize + Flux PoC

## Overview

- Defines both a dev and a prod environment
    - We would run these in different clusters, so only 1 flux would point at each environment
- dev and prod can have different workloads (for example prod runs an extra `web-app-3` workload)
- We use a `generic-web-app` base which defines a generic app (deployment/svc)
    - Each web-app inherits from this, and may run a different container-image, be in a different namespace, and in general, perform a different task (the web-app implementations are not related to each other - although in this example we make use of `podinfo` for all 3 workloads).
    - Each web-app uses `namePrefix` to override the generic name of the app
    - Each web-app can have its own set of patches
- Each environment adds a few environment specific annotations

## Usage

```
kustomize build environments/dev | kubectl apply -f -
```

```
kubectl port-forward svc/web-app-1-app 9898
```

Browse to http://localhost:9898


## More info

These workloads deploy a helpful utility container:  https://github.com/stefanprodan/k8s-podinfo

It assumes you have access to a pre-configured k8s cluster, with access to the `default` namespace.
