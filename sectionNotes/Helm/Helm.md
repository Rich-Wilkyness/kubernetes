# Helm
- is a package manager for Kubernetes.
- helm is like a game installer, it installs the game and all the dependencies needed to run the game.
    - if you define a package called wordpress, helm will install wordpress and all the dependencies needed to run wordpress.
        - the deployment, pods, services, pv, pvc, secrets, etc. (the yaml files needed to run wordpress)

- able to define many resources in a single file (passwords, usernames, etc.)
- when we make a change to the wordpress package we can run `helm upgrade wordpress` to update the package
- can rollback to a previous version of the package with `helm rollback wordpress 1`
- we can uninstall the package with `helm uninstall wordpress`

# Installing Helm
- first, must have a functioning Kubernetes cluster
- kubectl must be installed and configured to use the cluster (correct credentials in the kubeconfig file)
- works on Windows, Mac, and Linux
- always check documentation for the most up-to-date installation instructions
## Linux
- you can use `sudo snap install helm --classic` to install helm
    - --classic is a relaxed sandboxing mode that allows access to the host system
        - this way helm can access the kubectl configuration file in the home directory, so it can connect to the Kubernetes cluster
    - snap is a package manager for Linux
- for Ubuntu and Debian (need to get the key and sources list for the helm repository)
    - `curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -`
    - `sudo apt-get install apt-transport-https --yes`
    - `echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list`
    - `sudo apt-get update`
    - `sudo apt-get install helm`
- for package based systems
    - `pkg install helm`

# Helm Concepts
- how to turn yaml files into a package 
    - first turn the yaml files into templates
    - determine variables (things that change) and put them in a values.yaml file
        - double curly braces `{{ .Values.variableName }}` to reference the variable in the template

```yaml templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.name }}-deployment
spec:
    replicas: {{ .Values.replicaCount }}
    selector:
        matchLabels:
            app: {{ .Values.appLabel }}
    template:
        metadata:
            labels:
                app: {{ .Values.appLabel }}
        spec:
        containers:
        - name: {{ .Values.containerName }}
            image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
            ports:
            - containerPort: {{ .Values.containerPort }}
```
```yaml values.yaml
name: myapp
replicaCount: 3
appLabel: frontend
containerName: myapp
image:
    repository: nginx
    tag: latest
containerPort: 80
```

- the values.yaml file + the template files = a chart!!!
    - this will create a chart.yaml file that describes the chart

- add other public charts
    - `helm repo add <repo-name> <repo-url>`
- search for charts
    - `helm search repo <chart-name>`
    - `helm repo list` to see the repos you have added
- pull down a chart
    - `helm pull --untar <repo-name>/<chart-name>`
        - `--untar` will untar the chart (extract the files)
- install a chart
    - `helm install <release-name> <chart-name>`
        - `helm install release-1 bitnami/wordpress`
            - each time you run install, it creates a new release that is independent of the other releases
                - if you install release-1 and then install release-2, release-1 will not be affected by release-2 and vice versa
    - `helm install <release-name> <repo-name>/<chart-name>`
- uninstall a chart
    - `helm uninstall <release-name>`
- change the values of the chart 
    - in the directory of the chart you can run ls to see the files
    - edit the values.yaml file to change the values
- once the changes are made, you can run `helm install <release-name> ./<chart-path>` 
    - the path is the path to the chart directory


# Helm Commands
Which command is used to search for a wordpress helm chart package from the Artifact Hub?
- use `helm --help` to see the commands
    - `helm search hub <chart-name>`
        - hub is the Artifact Hub 

Add a bitnami helm chart repository in the controlplane node.
- `helm repo add bitnami https://charts.bitnami.com/bitnami`

How many helm repositories are added in the controlplane node?
- `helm repo list`

How to get the release name of a chart package we installed?
- `helm list` - will show name version 