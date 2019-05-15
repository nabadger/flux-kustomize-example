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


## Flux Example

Note, this example works against the latest PR and built image here: https://github.com/weaveworks/flux/pull/1848

If you are using remote bases, you ensure flux has the deploy-keys required to access all the required repos.

### Handling multiple environments

Example flux deployment config showing a dev cluster running flux to deploy dev workloads.
```
        # replace or remove the following URL
        - --git-url=git@github.com:nabadger/flux-kustomize-example.git
        - --git-branch=master
        - --git-path=environments/dev
```

Example flux deployment config showing a prod cluster running flux to deploy prod workloads.

```
        # replace the following url
        - --git-url=git@github.com:nabadger/flux-kustomize-example.git
        - --git-branch=master
        - --git-path=environments/prod
```

### Image Releases

When flux performs an auto-release, or you force a release via `fluxctl release`, flux ends up creating (and committing back) a patch-file
which contains the diff.

Flux handles this patch file (and applies it against the rendered output of the `kustomize build` command).

Example `flux-patch.yaml` created under `./environments/dev/flux-patch.yaml`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name:  web-app-1-app
spec:
  template:
    spec:
      $setElementOrder/containers:
      - name: container-name
      containers:
      - image: stefanprodan/podinfo:v1.0
        name: container-name
---
...
---
```



## More info

These workloads deploy a helpful utility container:  https://github.com/stefanprodan/k8s-podinfo

It assumes you have access to a pre-configured k8s cluster, with access to the `default` namespace.