### maintaining the helm registry
the registry is built statically using github pages
see this [helm document](https://github.com/helm/helm/blob/master/docs/chart_repository.md#github-pages-example) for more details.
The registry can be found here : https://talend.github.io/helm-charts-public/

In order to add a new chart you need to :
```
$ cd all-my-charts-root-folder
$ helm package mychart
$ cd root-of-this-git-repo
$ cp all-my-charts-root-folder/mychart-0.1.0.tgz .
$ helm repo index . --url https://talend.github.io/helm-charts-public/
$ git add -i
$ git commit -m "adding mychart v0.1.0 to the registry"
$ git push origin master
```
