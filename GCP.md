# Setup guide for GCP

```sh
# provision cluster on GCP
gcloud container clusters create ml-cd-starter-kit --region asia-southeast1 [--num-nodes=1]

# create tiller service account and give tiller access to default namespace
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
# kubectl --namespace kube-system patch deploy tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 

# make system:default cluster-admin so that when gocd has permission to run kubectl commands as admin
kubectl create clusterrolebinding default-cluster-rule --clusterrole=cluster-admin --serviceaccount=default:default
# TODO: 
# - run above step against ml-cd-starter-kit-gocd service account
# - Make CI run kubectl commands as ml-cd-starter-kit-gocd service account
# - read more here: https://github.com/helm/helm/issues/3130

# initialize helm on k8s cluster (install tiller into the cluster)
helm init --service-account tiller  # TODO: try without the --service-account argument

# create storage class for gocd
kubectl create -f storage/GoCDStorageClass.yaml

# create helm release (you should wait a minute for tiller pod to be ready before running this)
helm install --name ml-cd-starter-kit -f values.yaml .

# wait/watch for all pods to be up
watch kubectl get pods

# once gocd is up, access the gocd pod locally 
# note: run kubectl get pods to see pod name
kubectl port-forward --namespace default ml-cd-starter-kit-gocd-server-fc747f979-dk9m5 8153:8153

# once kibana is up, access the kibana pod locally
kubectl port-forward --namespace default ml-cd-starter-kit-kibana-7dfbf75565-jzcb5 5601:5601

# clean up. to delete the gke cluster, run:
gcloud container clusters delete ml-cd-starter-kit --region asia-southeast1


# Other commands that you might find useful

# to apply new helm configuration
helm upgrade ml-cd-starter-kit -f values.yaml .

# to get logs of a running pod
kubectl logs POD_NAME # get pod name from kubectl get pods
```

### Configure GoCD
- Add artifact store
  - Admin -> Artifact stores -> Add (top right of page) -> 
  - values to put in form:
    - id: dockerhub (as specified in ci-workshop-app/ci.gocd.yaml)
    - Docker Registry Url: (e.g. hub.docker.com)
    - username / password: your username and password
- Add config repo
  - Admin -> Config repositories -> Add (top right of page) ->
  - values to put in form:
    - Config repository id: anything 
    - URL: URL to your git repo containing the gocd pipeline yaml file.
  - Click on Test Connection, and then Save
- Create elastic profile `docker-dind-kubectl`
  - In 'Config Properties' radio button, type: sheroy/gocd-agent-with-kubectl:latest
  - Click on 'Pod Yaml' radio button, and paste the following:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-name-prefix-{{ POD_POSTFIX }}
  labels:
    app: web
spec:
  serviceAccountName: default
  containers:
    - name: gocd-agent-container-{{ CONTAINER_POSTFIX }}
      image: sheroy/gocd-agent-with-kubectl:latest
      securityContext:
        privileged: true
```
```sh


```

TODOs:
- helm/k8s
  - expose GoCD, kibana, MLFlow as public services
  - use helm instead of manual kubectl steps (is this a good idea?)
  - Figure out how to create different namespaces for staging and prod
  - Rename ci-workshop-app-chart to ci-workshop-app
  - Give GoCD service account cluster-admin access (is this a good idea?) and make GoCD run kubectl commands as gocd service account
  - include mlflow in chart
  - figure out why elasticsearch is not up after 20mins
- GoCD
  - Figure out how to skip artifact if job fails (ticket raised. if no solution, can add docker push as the last task in a job)
  - Suggest to tomzo/sheroy about improving local yaml syntax checker (none of the ones listed in tomzo/gocd-yaml-config-plugin works) 
- replace all project-specific values (e.g. ai-sg-workshop, etc.) to environment variables

DONE
- Figure out how to make first `docker build` command faster on GoCD
- install our demo app in this cluster
- push our app to docker hub
- create helm chart for our app