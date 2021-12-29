<!-- `make toc` to generate https://github.com/jonschlinkert/markdown-toc#cli -->

<!-- toc -->

- [What is Helm?](#what-is-helm)
- [Useful resources/tips from their website](#useful-resourcestips-from-their-website)
- [Basic directory structure of a chart](#basic-directory-structure-of-a-chart)
- [Useful commands/code](#useful-commandscode)
  * [dry run template rendering](#dry-run-template-rendering)
  * [view values](#view-values)
  * [override value files](#override-value-files)
  * [access values with control flow](#access-values-with-control-flow)
  * [predefined values](#predefined-values)
  * [search a chart keyword](#search-a-chart-keyword)
- [Common Helm Commands](#common-helm-commands)

<!-- tocstop -->


### What is Helm?

Helm to Kubernetes is like apt to linux - package management. These packages are called [helm charts](https://artifacthub.io/packages/search?kind=0). Charts use and built on [go template lanaguage](https://helm.sh/docs/howto/charts_tips_and_tricks/#know-your-template-functions). [Chart releaser action can be used with github workflow CI](https://helm.sh/docs/howto/chart_releaser_action/). Chart uses semver and 2 spaces instead of tabs in yaml. `Chart.yaml` needs to have capital `C` as it's case sensitive.

### Useful resources/tips from their website
- [Chart template guide](https://helm.sh/docs/chart_template_guide/)
- [Helm CLI](https://helm.sh/docs/helm/)
- [RBAC in chart](https://helm.sh/docs/chart_best_practices/rbac/)
- [Helm integration with different platforms](https://helm.sh/docs/topics/kubernetes_distros/)
- [Helm architecture](https://helm.sh/docs/topics/architecture/)
- [Tips/tricks when developing chart](https://helm.sh/docs/howto/charts_tips_and_tricks/)

### Basic directory structure of a chart
Check [official doc](https://helm.sh/docs/topics/charts/#the-chart-file-structure) for most up-to-date info. Run `helm create ${package-name}` to generate a chart with default dirs.
```
package-name/
|-- charts/ <== vendor folder for dependencies
|-- crds/ <== custom resources definition
|-- templates/ <== yaml definitions for your services, deployments and other k8s objects
|   |-- NOTES.txt <== a templated txt file printed out after the chart successfully deployed, useful for next step guide (security considerations)
|   |-- _helpers.tpl <== template partials/functions
|   |-- deployment.yaml
|   |-- ingress.yaml
|   |-- service.yaml
|-- Chart.yaml <==contains metadata of your chart, e.g. name, description, version, etc
|-- LICENSE
|-- README.md
|-- requirements.yaml <== dependencies for using this chart
|-- values.yaml <== default values of the chart template
```

### Useful commands/code

#### dry run template rendering
the rendered template will show in output
```
helm install --debug --dry-run ./mychart
```

#### view values
Helm values can come from `values.yaml` of your chart, its sub-charts, any values files (plain yaml files) or values provided directly on the command line.
```
helm inspect values # displays current config
```

#### override value files
Default values are stored under a file called `values.yaml`, override default values 👇
```
helm install -f myvalues.yaml myredis ./redis
helm install -f myvalues.yaml -f override.yaml  myredis ./redis # most right value takes priority
helm install --set name=prod myredis ./redis
helm install --set foo=bar --set foo=newbar  myredis ./redis # most right value takes priority
helm install --set-string long_int=1234567890 myredis ./redis
helm install --set-file my_script=dothings.sh myredis ./redis
```

#### access values with control flow
HashiCorp has a [clear doc](https://learn.hashicorp.com/tutorials/nomad/go-template-syntax) to introduce go template.
- use if/else and range for logic control and less repetition
```
{{ - if .Values.deployment.volumes }}
volumes:
{{ - range .Values.deployment.volumes }}
- name: {{ .name }}
  secret:
    secretName: {{ .secretName }}
{{ - end }}
{{ - end }}
```

- use with to narrow the scope of the value context
similar to java's typesafe config https://github.com/lightbend/config
```
{{ - with .Value.deployment }} # all the values within this block are defined under `deployment` value object
strategy:
  rollingUpdate:
    maxUnavailable: {{ .maxUnavailable }}
    maxSurge: {{ .maxSurge }}
revisionHistoryLimit: {{ .revisionHistoryLimit }}
minReadySeconds: {{ .minReadySeconds }}
{{ - end }}
```
#### predefined values
cannot be overridden
- `Release.Name`
- `Release.Time`
- `Release.Namespace`
#### search a chart keyword
```
helm search ${keyword} # search for the keyword in chart
helm search hub ${keyword} # search or the keyword in chart in Artifact Hub or private hub
helm search repo ${keyword} # search repos for the keyword in charts
```

### Common Helm Commands
- [helm create](https://helm.sh/docs/helm/helm_create/): builds and names a new chart
- [helm pull](https://helm.sh/docs/helm/helm_pull/): called `helm fetch` before v3, downloads and unpacks a chart from a repo into a local dir
- [helm rollback](https://helm.sh/docs/helm/helm_rollback/): allows rollback a release to a previous version, run [helm history](https://helm.sh/docs/helm/helm_history/) for releases and its revision numbers
- [helm status](https://helm.sh/docs/helm/helm_status/): shows status of the requested release
- [helm upgrade](https://helm.sh/docs/helm/helm_upgrade/): upgrades a requested release