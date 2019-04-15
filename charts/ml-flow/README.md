# MLFlow helm chart

### Getting started

```sh
# Provision cluster on a cloud vendor of your choice. For local development, provision k8s cluster on minikube:
minikube start

# Initialize tiller 
helm init [--service-account tiller] [--history-max 200]
```

### Install mlflow on k8s cluster

```sh
# (Optional) Dry run of helm release
helm install --dry-run --debug .

# Release mlflow
helm install --name mlflow .
```
