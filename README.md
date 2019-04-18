# ml-cd-starter-kit

A helm chart that contains subcharts commonly used in ML projects (e.g. MLFlow, GoCD, EFK).

## Prerequisites Details

* Kubernetes 1.8+
* PV dynamic provisioning support on the underlying infrastructure

## Chart Details

...

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

### Create and configure cluster

#### Option 1: GCP
```sh
# provision cluster on GCP
gcloud container clusters create my-cluster region asia-southeast1

# create tiller service account and give tiller access to default namespace
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

# initialize helm on k8s cluster (install tiller into the cluster)
helm init --service-account tiller

# wait for pods and services to be up
kubectl get pods,services --all-namespaces
# mac users can `brew install watch` and run:
# watch kubectl get pods,services --all-namespaces
```

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

## Installing the Chart

1. Decide on a name for your helm release (e.g. `ml-cd-starter-kit`)
2. If you're using another name for your helm release (instead of `ml-cd-starter-kit`), go to `./values.yaml` and replace `ml-cd-starter-kit` with the name of your release in: `elasticsearch.url: http://ml-cd-starter-kit-elasticsearch-client:9200`
3. To install the chart with the release name `ml-cd-starter-kit`:

```bash
helm install --name ml-cd-starter-kit .  
# note: you can replace ml-cd-starter-kit with the name of your release

# wait for pods and services to be up
kubectl get pods,services
# mac users can `brew install watch` and run:
# watch kubectl get pods,services
```

That's it! You now have a kubernetes cluster running:
- GoCD for continuous integration
- EFKG for monitoring
- MLFlow for tracking metrics and hyperparameters

You can now go to your project repo (e.g. [`ml-app-template`](https://github.com/ThoughtWorksInc/ml-app-template)) and start developing and pushing your changes.

3 additional steps is required to configure GoCD as your CI server. You can find these steps [here](./gocd.md)

### Other useful commands

Here are some other useful commands that you might want to use
```sh
# to apply new helm configuration
helm upgrade ml-cd-starter-kit -f values.yaml .

# access pods locally
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
**Important**: Running k8s clusters can get expensive, so definitely remember to destroy your cluster when you're done exploring this repo:

`gcloud container clusters delete my-cluster --region asia-southeast1`

## Configuration

Each requirement is configured with the options provided by that Chart.
Please consult the relevant charts for their configuration options.

TODO: 
- Add instructions on GoCD config (elastic agent profile and dockerhub artifact repository)in `GCP.md`
- run above step against ml-cd-starter-kit-gocd service account
- Make CI run kubectl commands as ml-cd-starter-kit-gocd service account
- read more here: https://github.com/helm/helm/issues/3130
- find and replace 'davified'?