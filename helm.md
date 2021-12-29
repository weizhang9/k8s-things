### What is Helm?

Helm to Kubernetes is like apt to linux - package management. These packages are called [helm charts](https://artifacthub.io/packages/search?kind=0). Charts use [go template lanaguage](https://pkg.go.dev/text/template). [Chart releaser action can be used with github workflow CI](https://helm.sh/docs/howto/chart_releaser_action/). Chart uses semver and 2 spaces instead of tabs in yaml. `Chart.yaml` needs to have capital `C` as it's case sensitive.

### Useful resources/tips from their website
- [Chart template guide](https://helm.sh/docs/chart_template_guide/)
- [Helm CLI](https://helm.sh/docs/helm/)
- [RBAC in chart](https://helm.sh/docs/chart_best_practices/rbac/)
- [Helm integration with different platforms](https://helm.sh/docs/topics/kubernetes_distros/)
- [Helm architecture](https://helm.sh/docs/topics/architecture/)
- [Tips/tricks when developing chart](https://helm.sh/docs/howto/charts_tips_and_tricks/)
- 