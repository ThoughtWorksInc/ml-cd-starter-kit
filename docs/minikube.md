# Using minikube

### Pre-requisites
- [Install minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

### Create k8s cluster using minikube

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

## Install cross-cutting services using helm

The ml-cd-starter-kit helm chart will install some quite memory-intensive applications (e.g. elasticsearch). To try this demo on minikube, you have to first disable elasticsearch, kibana and fluentd by updating the following values in `./values.yaml` from true to false:

```yaml
elasticsearch:
  enabled: false
kibana:
  enabled: false
fluentd:
  enabled: false
fluentd-elasticsearch:
  enabled: false
```

Install cross-cutting services (instructions [here](../README.md#install-cross-cutting-services-using-helm))

Access these services on your browser:

```sh
# get pods and their names
kubectl get pods

# see gocd
kubectl port-forward --namespace default NAME_OF_GOCD_POD 8153:8153

# see mlflow
kubectl port-forward --namespace default NAME_OF_MLFLOW_POD 5000:5000 

# see ml-app-template
kubectl port-forward --namespace default NAME_OF_APP_POD 8080:8080
```

You can now
- Configure GoCD as your CI server. 3 additional steps are required, and you can find these steps in [gocd.md](./gocd.md)
- Go to your project repo (e.g. [`ml-app-template`](https://github.com/ThoughtWorksInc/ml-app-template)) and start developing and pushing your changes.