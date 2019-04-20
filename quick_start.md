# Getting started

```sh
# create k8s cluster
gcloud container clusters create my-cluster region asia-southeast1

# create tiller service account and give tiller access to default namespace
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

# initialize helm on k8s cluster (install tiller into the cluster)
helm init --service-account tiller

# give gocd service account access to run kubectl commands to deploy to staging and prod
kubectl create clusterrolebinding default-cluster-rule --clusterrole=cluster-admin --serviceaccount=default:default

# wait for tiller-deploy pod to be ready
kubectl get pods,services

helm install --name ml-cd-starter-kit .
# note: you can replace ml-cd-starter-kit with the name of your release if you want

# wait for pods and services to be up
kubectl get pods,services
```