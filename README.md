# ml-cd-starter-kit

A helm chart that contains subcharts commonly used in ML projects (e.g. MLFlow, GoCD, EFK).

## Prerequisites Details

- Kubernetes 1.8+
- PV dynamic provisioning support on the underlying infrastructure

## Getting started

### Pre-requisites

1. Create project on GCP: https://cloud.google.com/resource-manager/docs/creating-managing-projects
2. Install `gcloud`

```bash
export CLOUDSDK_CORE_DISABLE_PROMPTS=1
curl https://sdk.cloud.google.com | bash
export PATH=$HOME/google-cloud-sdk/bin
```

3. Configure gcloud cli (authenticate, set default project, etc.): https://cloud.google.com/sdk/docs/quickstart-macos (scroll down to "Initialize the SDK")
4. Install helm

- mac: `brew install kubernetes-helm`
- windows: `choco install kubernetes-helm`
- Other OS: see https://github.com/helm/helm#install

### Create and configure k8s cluster

#### Option 1: GCP

```sh
# provision cluster on GCP
gcloud container clusters create my-cluster --region asia-southeast1 --num-nodes 2
```

Common issues:

- Enabling Kubernetes Engine API for your project in Google Cloud Console. If you have not done so, running the command above will provide a link for you to do so.
- If you are new to Google Cloud Platform, you might need to upgrade your account.
- There might be an issue of not having enough master nodes which leads to the pods being stuck in the pending state. To circumvent this, we can increase the num-nodes in the command above. In order to increase num-nodes we might also need to request for an increase in the quota in Google Cloud Platform (an error message will pop out with a link to guide you to the page to request for this increase).

#### Option 2: Minikube

```sh
# Optionally delete minikube
minikube delete

# Ensure that your minikube is at least 0.34
minikube version
# if not, upgrade minikube
brew cask upgrade minikube

# start kubernetes cluster on minikube
minikube start --vm-driver=virtualbox --cpus 6 --memory 8192 --bootstrapper=kubeadm --extra-config=apiserver.authorization-mode=RBAC
```

## Install helm

```sh
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

## Installing the Chart

1. Decide on a name for your helm release (e.g. `ml-cd-starter-kit`)
2. If you're using another name for your helm release (instead of `ml-cd-starter-kit`), go to `./values.yaml` and replace `ml-cd-starter-kit` with the name of your release in: `elasticsearch.url: http://ml-cd-starter-kit-elasticsearch-client:9200`
3. Install the following using the helm chart:

- GoCD for continuous integration
- ElasticSearch, Fluentd, Kibana and Grafana for monitoring
- MLFlow for tracking metrics and hyperparameters
- ml-app-template

```bash
helm install --name ml-cd-starter-kit .
# note: you can replace ml-cd-starter-kit with the name of your release if you want

# wait for pods and services to be up
kubectl get pods,services
# mac users can `brew install watch` and run:
# watch kubectl get pods,services (Hit Ctrl+C to exit)

# create a new release for our production app (ml-app-template)
# the first helm install command installed the app in the staging environment only
cd charts/ml-app-template
helm install --name ml-cd-starter-kit-prod -f values.yaml .
# note: if you use another release name, remember to update it in the deploy_prod stage in ml-app-template/ci.gocd.yaml
```

That's it! You now have a kubernetes cluster running cross-cutting services (GoCD, MLFlow, EFK, Grafana) and an instance of [ml-app-template](https://github.com/ThoughtWorksInc/ml-app-template)

You can now:

- Access these services (e.g. GoCD, MLFlow). To get the public IP of these services, run `kubectl get services` and look the `EXTERNAL-IP` column. Don't forget to specify the port after the public IP (e.g. MLFlow runs on port 5000)
- Go to your project repo (e.g. [`ml-app-template`](https://github.com/ThoughtWorksInc/ml-app-template)) and start developing and pushing your changes.
- Configure GoCD as your CI server. 3 additional steps are required, and you can find these steps in [gocd.md](./gocd.md)
- Update the following in `ml-app-template/src/env.py`:
  - `MLFLOW_IP`
  - `FLUENTD_IP`

### Other useful commands

Here are some other useful commands that you might want to use

```sh
# to apply any changes to your helm charts
helm upgrade ml-cd-starter-kit -f values.yaml .

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
