# Most efficient Kube-Native & Multicloud CI-CD Tool!

* Delete your scary **Jenkins** away!
* You can also not rely on any **SaaS platforms** that contain your sensitive configuration in **their vault**.
* Install AgnOps in any cloud that contains K8s cluster and deploy with **no kubeconfig at all!**
* It's a **100% code-based** pipeline, so you're on **100% IaC** vision.
* AgnOps's components are mostly based on Golang, consume about **100Mb of RAM** in total, and manage K8s Secrets as a DB.
* The whole system is **scalable** - AgnOps main components (K8s Deployments) and pipeline workers (K8s Job objects).

## General installation guide for Minikube/On-premise/Cloud
## [Multicloud installation guide](Multicloud.md)

![AgnOps](https://raw.githubusercontent.com/agnops/AgnOps/master/Architecture.svg)

### Prerequisites

* Kubernetes cluster.
* Helm v3
* [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/) configured with SSL certificate (security-wise recommended for webhooks).
* If you are using minikube, execute the following line to enable Ingress: `minikube addons enable ingress`

### Installing the Chart

```bash
# Add the AgnOps Helm repository
helm repo add agnops https://charts.agnops.com/
```

* It is advisable to check the [chart configuration](http://charts.agnops.com/) to tailor the installation for your needs, below are the links to AgnOps components: 

    * [cloud-wrapper](https://github.com/agnops/cloud-wrapper)
    * [oauth2-proxy](https://github.com/agnops/oauth2-proxy)
    * [job-generator](https://github.com/agnops/job-generator)
    * [webhook-manager](https://github.com/agnops/webhook-manager)
    * [notification](https://github.com/agnops/notification)

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
export scmProvider=github # gitlab , bitbucket
export ingressClass='--set global.ingress.annotations."kubernetes\.io/ingress\.class"=nginx'
export slackUrl="https://hooks.slack.com/services/bla/bla/bla"
export oauth2Proxy='--set oauth2-proxy.clientId="",oauth2-proxy.clientSecret="",oauth2-proxy.cookieSecret=""'
export namespace="default" # ci-cd-tools

helm upgrade --install agnops agnops/agnops --set cloud-wrapper.enabled=false $oauth2Proxy --set notification.slack.webhookUrl=$slackUrl --set global.scmProvider=$scmProvider $ingressClass --namespace=$namespace
```

Alternatively, a YAML file that specifies the values for the above parameters can be provided while installing the chart. For example,

```console
$ helm upgrade --install agnops -f values.yaml agnops/agnops --namespace=ci-cd-tools
```

> **Tip**: You can use the default [values.yaml](https://github.com/agnops/helm-umbrella-chart/blob/master/agnops/values.yaml)

### Uninstalling the Chart
To uninstall/delete the `agnops` deployment:
The command removes all the Kubernetes components associated with the chart and deletes the release.

```bash
export namespace="default" # ci-cd-tools

helm delete agnops --namespace=$namespace
```

If you want to uninstall AgnOps persistent data completely, you will also need to execute the following `kubectl` commands:

```bash
# SecretsDB Cleanup
for p in $(kubectl get secrets -n $namespace -l AgnOps=OrgUserAuthDetails --no-headers | awk '{print $1}'); do kubectl delete secrets -n $namespace $p;done
for p in $(kubectl get secrets -n $namespace -l AgnOps=WebhookSecret --no-headers | awk '{print $1}'); do kubectl delete secrets -n $namespace $p;done
```