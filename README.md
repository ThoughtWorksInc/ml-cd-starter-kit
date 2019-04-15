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
- If you're using another name of the helm release (instead of `ml-cd-starter-kit`), replace `ml-cd-starter-kit` with the name of your release in: `elasticsearch.url: http://ml-cd-starter-kit-elasticsearch-client:9200`

## Installing the Chart

To install the chart with the release name `ml-cd-starter-kit`:

```bash
$ helm install --name ml-cd-starter-kit .
```

TODO: Add instructions on GoCD config (elastic agent profile and dockerhub artifact repository)in `GCP.md`

That's it! You now have a kubernetes cluster running:
- GoCD for continuous integration
- EFKG for monitoring
- MLFlow for tracking metrics and hyperparameters

You can now go to your project repo (e.g. `ci-workshop-app`) and start developing and pushing your changes.

## Deleting the Charts

Delete the Helm deployment as normal

```
$ helm delete ml-cd-starter-kit
```

Deletion of the StatefulSet doesn't cascade to deleting associated PVCs. To delete them:

```
$ kubectl delete pvc -l release=ml-cd-starter-kit,component=data
```

## Configuration

Each requirement is configured with the options provided by that Chart.
Please consult the relevant charts for their configuration options.
