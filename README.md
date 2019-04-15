# ml-cd-starter-kit

...

## Prerequisites Details

* Kubernetes 1.8+
* PV dynamic provisioning support on the underlying infrastructure

## Chart Details

...

## Getting started

### Create and configure cluster

```sh
# provision cluster on GCP
gcloud container clusters create ml-cd-starter-kit --region asia-southeast1

# create tiller service account and give tiller access to default namespace
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

# initialize helm on k8s cluster (install tiller into the cluster)
helm init --service-account tiller

# wait for pods and services to be up
kubectl get pods,services 

# mac users can `brew install watch` and run:
# watch kubectl get pods,services
```


In `./values.yaml`:
- If you're using another name of the helm release (instead of `my-release`), replace `my-release` with the name of your release in: `elasticsearch.url: http://my-release-elasticsearch-client:9200`

- TODO: copy over instructions from charts/mlflow/README.md

## Installing the Chart

To install the chart with the release name `my-release`:

```bash
$ helm install --name my-release .
```

## Deleting the Charts

Delete the Helm deployment as normal

```
$ helm delete my-release
```

Deletion of the StatefulSet doesn't cascade to deleting associated PVCs. To delete them:

```
$ kubectl delete pvc -l release=my-release,component=data
```

## Configuration

Each requirement is configured with the options provided by that Chart.
Please consult the relevant charts for their configuration options.
