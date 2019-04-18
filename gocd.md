# Configure GoCD

### Add artifact store
If you are using an external artifact store (e.g. dockerhub), you have to configure GoCD so that it knows how to push to the external artifact store.

Steps:
- Visit GoCD url on port 8153 (you can get the url by running `kubectl get svc`)
- On the top of the UI, click on Admin -> Artifact stores -> Add (top right of page)
- Place the following in the form:
  - id: dockerhub (you can name this anything you want, but remember to update it in `ml-app-template/ci.gocd.yaml`)
  - Docker Registry Url: (e.g. hub.docker.com)
  - username / password: your username and password

### Add config repo
Adding a config repo tells GoCD where to look for the your ci pipeline yaml file.

Steps:
- On the GoCD UI, click on Admin -> Config repositories -> Add (top right of page)
- Place the following in the form:
  - Config repository id: anything-you-want 
  - URL: URL to your git repo containing the gocd pipeline yaml file (e.g. https://github.com/ThoughtWorksInc/ml-app-template)
- Click on Test Connection, and then Save

### Create elastic profile 
- On the GoCD UI, click on Admin -> Elastic Profile -> Add
- In the `id` field, fill in `docker-dind-kubectl` (this is what we specified in `ml-app-template/ci.gocd.yaml`)
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
      image: godemo/gocd-agent-with-kubectl:latest  # use this image to run our tasks on CI
      securityContext:
        privileged: true
```

You can also create your own elastic agent profiles. Just remember update the `elastic_profile_id` according in your `app/ci.gocd.yaml`