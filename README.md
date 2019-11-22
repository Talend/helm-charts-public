## Welcome to the Talend public helm charts registry

This repository host the public [helm charts](https://helm.sh/) that are created and maintained by Talend and provide them as a Helm registry.


### How to add the Talend public registry to helm

In order for the helm command line to be able to install the charts you need to register this helm registry using the following command

```bash
helm repo add talend-public https://talend.github.io/helm-charts-public
```

You may list all charts available to installation with this command.
```bash
helm search talend
```

and then use the classic `helm install` command in order to install the chart into your favorite kubernetes cluster.
