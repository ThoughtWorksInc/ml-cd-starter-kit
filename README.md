# ml-cd-starter-kit

A helm chart that contains subcharts commonly used in ML projects (e.g. MLFlow, GoCD, EFK).

## Prerequisites Details

- Kubernetes 1.8+
- PV dynamic provisioning support on the underlying infrastructure
- Create project on GCP: https://cloud.google.com/resource-manager/docs/creating-managing-projects
- Install `gcloud`:

```bash
export CLOUDSDK_CORE_DISABLE_PROMPTS=1
curl https://sdk.cloud.google.com | bash
export PATH=$HOME/google-cloud-sdk/bin
```

- Configure gcloud cli (authenticate, set default project, etc.): https://cloud.google.com/sdk/docs/quickstart-macos (scroll down to "Initialize the SDK")
- Install `helm`:
  - mac: `brew install kubernetes-helm`
  - windows: `choco install kubernetes-helm`
  - Other OS: see https://github.com/helm/helm#install

### Create and configure k8s cluster (using Google Cloud Platform)

Note: For instructions on how to run this locally on minikube instead, see [minikube.md](./docs/minikube.md)

```sh
# provision cluster on GCP
gcloud container clusters create my-cluster --region asia-southeast1
# note: 
# - you may have to enable Kubernetes Engine API for your project in the GCP console. If you have not done so, running the command above will provide a link for you to do so.
# - if you are new to Google Cloud Platform, you might need to upgrade your account when prompted.

# create tiller service account and give tiller access to default namespace
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

# initialize helm on k8s cluster (install tiller into the cluster)
helm init --service-account tiller

# give gocd service account access to run kubectl commands to deploy to staging and prod
kubectl create clusterrolebinding default-cluster-rule --clusterrole=cluster-admin --serviceaccount=default:default

# wait for tiller-deploy pod to be ready
kubectl get pods,services --all-namespaces
# mac users can `brew install watch` and run:
# watch kubectl get pods,services --all-namespaces (Hit Ctrl+C to exit) (Hit Ctrl+C to exit)
```

## Install cross-cutting services using helm

Install the following services using the helm chart:
- GoCD for continuous integration
- ElasticSearch, Fluentd, Kibana and Grafana for monitoring
- MLFlow for tracking metrics and hyperparameters
- [`ml-app-template`](https://github.com/ThoughtWorksInc/ml-app-template)

```bash
helm install --name ml-cd-starter-kit .
# note: you can replace ml-cd-starter-kit with the name of your release if you want.
# if you do that, you have to replace `ml-cd-starter-kit` with the name of your release in ./values.yaml: elasticsearch.url: http://YOUR_RELEASE_NAME-elasticsearch-client:9200`
# also, you'll need to replace `ml-cd-starter-kit` with your release name in ml-app-template/ci.gocd.yaml

# wait for pods and services to be up
kubectl get pods,services
# mac users can `brew install watch` and run:
# watch kubectl get pods,services (Hit Ctrl+C to exit)

# create a new release for our 'production' app (ml-app-template)
# the first helm install command installed the app as our 'staging' app
cd charts/ml-app-template
helm install --name ml-cd-starter-kit-prod -f values.yaml .
# note: if you use another release name, remember to update it in the deploy_prod stage in ml-app-template/ci.gocd.yaml
```

That's it! You now have a kubernetes cluster running cross-cutting services (GoCD, MLFlow, EFK, Grafana) and an instance of [ml-app-template](https://github.com/ThoughtWorksInc/ml-app-template)

You can now:

- Access these services (e.g. GoCD, MLFlow). To get the public IP of these services, run `kubectl get services` and look the `EXTERNAL-IP` column. Don't forget to specify the port after the public IP (e.g. MLFlow runs on port 5000)
- Configure GoCD as your CI server. 3 additional steps are required, and you can find these steps in [gocd.md](./gocd.md)
- Go to your project repo (e.g. [`ml-app-template`](https://github.com/ThoughtWorksInc/ml-app-template)) and start developing and pushing your changes.
- Update the following in `ml-app-template/src/settings.py`:
  - `MLFLOW_IP`
  - `FLUENTD_IP`

### Configuring the helm charts

You can configure the chart and subcharts by updating `./values.yaml`. When you're done, you can run the following commands:

```sh
# do a dry run of your helm installation. this will send the chart to the Tiller server, which will render the templates. But instead of installing the chart, it will return the rendered template to you so you can see the output yaml files
helm install . --dry-run --debug 

# apply your changes to your helm charts
helm upgrade ml-cd-starter-kit -f values.yaml .
```

You can read more about how to configure subcharts [here](https://github.com/helm/helm/blob/master/docs/chart_template_guide/subcharts_and_globals.md)

### Other useful commands

Here are some other useful commands that you might find useful:
```sh
# to access pods on localhost
# note: run kubectl get pods to see pod name
kubectl port-forward --namespace default NAME_OF_POD 8153:8153 # replace 8153 with the port of the service you want to access

# to get logs of a running pod
kubectl logs POD_NAME # get pod name from kubectl get pods
```

## Clean up

### Deleting the Charts

Delete the Helm deployment as normal

```
helm delete ml-cd-starter-kit
```

Deletion of the StatefulSet doesn't cascade to deleting associated PVCs. To delete them:

```
kubectl delete pvc -l release=ml-cd-starter-kit,component=data
```

### Destroy k8s cluster

**Important**

```diff
!  Running k8s clusters will get expensive, so definitely remember to destroy your
!  cluster when you're done exploring this repo:
```

`gcloud container clusters delete my-cluster --region asia-southeast1`

## Configuration

Each requirement is configured with the options provided by that Chart.
Please consult the relevant charts for their configuration options.
