# Overview

blurb

<!-- @test @makeTempWorkDir -->
```
WORKDIR=$(mktemp -d)
```

<!-- @test @build -->
```
kustomize build app-config/apps/sandbox/echoheaders > $WORKDIR/manifests.yaml
```

<!-- @test @show -->
```
cat $WORKDIR/manifests.yaml
```

<!-- @test @validate -->
```
cat $WORKDIR/manifests.yaml | kubeval
```
