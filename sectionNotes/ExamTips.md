# Important Commands (Imperative Commands)
- `kubectl create pod <pod-name> --image=<image-name> -n <namespace> --restart=Never --dry-run=client --port=80 --labels=app=myapp --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi' -o yaml > pod.yaml`
- `--dry-run=client` - does not create the pod, just outputs the yaml
- remember to use `create <resource>` with `dry-run` and `-o yaml > file.yaml` to generate the yaml
- `alias k=kubectl`
- Root privileges can be obtained by running `sudo -i`.

- `k pod --help` - get help for the pod command
- `-o wide` - to get IP addresses and nodes
- `--rm` for temporary containers
- `--expose` add this to a pod command to create a service for the pod automatically (it creates a ClusterIP service)
  - `kubectl expose deployment <deploy-name> --name=<name-of-service> --port=6379 --target-port=6379`
- `kubectl exec ubuntu-sleeper -- whoami` used to see who is running the command in the container
- `kubectl get pods -l 'team in (shiny, legacy)',env=prod --show-labels`
- `kubectl get pods -o yaml | grep -C 3 'annotations:'`
- `cat /etc/*release*` - to see the OS version
- `-l` is for getting pods with specific labels, while `--labels` is for setting labels on a resource

# Rollout Commands
- `k rollout history deploy <deploy-name>`
- `kubectl annotate <resource-type> <resource-name> update-reason="Updated image to latest version"` or `kubectl annotate deployment nginx kubernetes.io/change-cause="Pick up patch version"`
  - the `update-reason` only adds an annotation, but not a rollout history note. 
  - this will add the reason to the rollout histories most recent revision
- `kubectl rollout undo deploy <deploy-name> --to-revision=1`

#### HUGE `k explain pod.spec` 
- get help for the pod.spec field
- just add a `.` with the field you want to get options for. cant remember the field name?



- kubectl config use-context <context-name> 
  - switch between contexts (clusters, users, namespaces)
  - kubectl config set-context <context-name> --cluster=<cluster-name> --user=<user-name> --namespace=<namespace>
    - defines a context you can switch to
    - recommend naming the context by the question on the exam like q1 --cluster=q1 --user=q1 --namespace=q1

- sh into a pod
  - `kubectl exec -it <pod-name> -- sh` might need `/bin/sh`


# Versioning
## /v1
- pods, services, namespaces, rc, endpoints, events, nodes, bindings, PV, PVC, configmaps, secrets, services.

## /v1/apps
- deployments, replicasets, statefulsets, daemonsets.

## /batch/v1
- jobs, cronjobs
## rbac.authorization.k8s.io/v1
- roles, rolebindings, clusterroles, clusterrolebindings
## networking.k8s.io/v1
- networkpolicies, ingresses

# Logging
- `kubectl logs -f <pod-name> <container-name>`

# Full Deployment (Pod with all the bells and whistles)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  annotations: # Unlike labels, annotations are not used to identify and select objects. Instead, they are used to store additional information that can be useful for various purposes. build details, contact information, or any other data that might be relevant to the object. can be used by tools and automation scripts to store and retrieve information. For example, CI/CD pipelines might use annotations to store build and deployment information.
    commit: 2d3mg3 # Stores the commit hash of the deployed code
    contact: John Doe # Stores contact information for the person responsible for the deployment
  labels:
    app: myapp
    mvc: view
spec:
  strategy:
    type: RollingUpdate # RollingUpdate or Recreate
    rollingUpdate:
      maxSurge: 1 # The number of pods that can be created above the desired number of pods during the update process
      maxUnavailable: 2 # The number of pods that can be unavailable during the update process
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      # kubectl create serviceaccount my-service-account (bot account)
      # setup token for service account Configuration/serviceAccounts.md
      securityContext: # applied to pod or container level (container level takes precedence)
        runAsUser: 1000 # all processes in the container will run as this user, to run as root use 0
        # cannot add capabilities here, must be added to the container level (spec.containers.securityContext.capabilities.add)
        capabilities:
          add: ["MAC_ADMIN"] # allows for container to perform Mandatory Access Control operations
        allowPrivilegeEscalation: false # prevents the container from gaining more privileges than it started with
        readOnlyRootFilesystem: true # prevents the container from writing to the root filesystem, (stops the container from writing to the temporary filesystem)
      serviceAccountName: my-service-account # provides the pod with AuthN/AuthZ (tokens, roles, secrets). Links to the sa created in the namespace, usually for tokens.
      volumes:
      - name: mypodvolume
        persistentVolumeClaim:
          claimName: mypvc
      - name: mypodvolume2
        emptyDir: {} # ephemeral storage, deleted when pod is deleted
      - name: mypodvolume3
        hostPath:
          path: /tmp/data
          type: DirectoryOrCreate # create the directory if it does not exist
      - name: mypodvolume4
        awsElasticBlockStore:
          volumeID: <volume-id>
          fsType: ext4
      - name: mypodvolume5
        configMap:
          name: myconfigmap
          items: # only include specific keys from the configmap
          - key: key1
          - key: key2
      # kubectl label nodes <node-name> disktype=ssd
      affinity:
          nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                  nodeSelectorTerms:
                  - matchExpressions:
                      - key: disktype
                        operator: In
                        values:
                        - ssd
      # kubectl label nodes <node-name> disktype=ssd
      nodeSelector:
          disktype: ssd
      # kubectl taint nodes <node-name> ssd=value:NoSchedule
      # if the toleration is the same as the taint, the pod will be scheduled on the node
      tolerations:
      - key: "ssd"
        operator: "Equal"
        value: "value"
        effect: "NoSchedule"
      initContainers:
      - name: init-myservice
        image: busybox
        command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
        
      # this covers all the variables needed for test on the container 
      containers:
      - name: nginx-container
        image: nginx
        volumeMounts:
        - mountPath: /opt # where on the container the volume will be mounted
          name: mypodvolume # name of the volume (pod.spec.volumes.name)
        ports:
        - containerPort: 80 # port that the service will target
        # use sh -c when you need to run multiple commands. Could have just had ['sleep', '3600']
        command: ["/bin/sh", "-c", "echo Hello Kubernetes! && sleep 3600"] # /bin/sh is the most reliable way to run commands in a container, don't just use /sh or /bash
        readinessProbe:
          httpGet: # web server
            path: /index.html
            port: 80 # port that the service will target
          tcpSocket: # database connection
            port: 3306 
          exec: # script
            command:
            - cat
            - /app/is_ready
        livenessProbe:
          httpGet:
            path: /index.html
            port: 80
          tcpSocket:
            port: 3306
          exec:
            command:
            - cat
            - /app/is_alive
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
          requests: # The status OOMKilled indicates that it is failing because the pod ran out of memory
            memory: "64Mi"
            cpu: "200m" # 200m or 0.2 cores
        env:
        - name: MYVAR
          valueFrom:
            configMapKeyRef:
              name: mysecret # name of the configmap
              key: mykey # key in the configmap
        - name: MYVAR2
          valueFrom:
            secretKeyRef:
              name: mysecret # name of the secret
              key: mykey # key in the secret
        - name: MYVAR3
          value: "myvalue" # value directly in the env
        - name: MYVAR4
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace # namespace of the pod
      

---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypod-pv
spec:
    capacity:
        storage: 1Gi
    accessModes:
        - ReadWriteOnce
    hostPath:
        path: /tmp/data
--- 
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypod-pvc
spec:
    accessModes:
        - ReadWriteOnce
    resources:
        requests:
            storage: 1Gi
    # use matchLabels or volumeName to bind the PVC to a PV
    selector:
        matchLabels:
            app: myapp
    # or 
    volumeName: mypod-pv
---
apiVersion: v1
kind: Service
metadata:
  name: myNodeService # expose the service to the outside world, can use LoadBalancer for cloud providers 
spec:
    type: NodePort
    selector:
        app: myapp
    ports:
        - protocol: TCP
          port: 80 # port on the service
          targetPort: 80 # port on the pod
          nodePort: 30080 # port that can be accessed from outside the cluster (192.168.2:30080) 
---
apiVersion: v1
kind: Service
metadata:
  name: myClusterService # expose the service to the cluster
spec:
    type: ClusterIP
    selector:
        app: myapp
    ports:
        - protocol: TCP
          port: 80
          targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysecret # same name as the pod.spec.containers.env.valueFrom.configMapKeyRef.name
data:
    mykey: myvalue # mykey is the name of the key from the pod. myvalue is the value of the key 
---
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3
  parallelism: 3
  template:
    spec:
      containers:
      - name: math-add
        image: ubuntu
        command: ['expr', '3', '+', '2']
      restartPolicy: Never
  backoffLimit: 4 
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: reporting-cron-job
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      completions: 1
      parallelism: 1
      template:
        spec:
          containers:
          - name: reporting-container
            image: my-reporting-image
            command: ["generate-report"]
          restartPolicy: OnFailure
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-to-db
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: frontend # The network policy applies to pods with the label app: frontend
  policyTypes:
  - Egress
  egress:
  - to: # The egress rule allows traffic to the database pod
    - podSelector:
        matchLabels:
          app: db
    ports: # note port is on the same level as to, not under to
    - protocol: TCP
      port: 3306 # Assuming the db pod is listening on port 3306
```

# Docker
- `docker build -t myapp .` - build a docker image from the Dockerfile in the current directory and tag it as myapp
- `docker run -d -p 3080:80 myapp` - run the myapp image in a container in detached mode and map port 3080 on the host to port 80 in the container
  - `docker ps` - list running containers 
  - `docker exec -it <container-id> sh` - sh into the container
  - `curl localhost:3080` - test the container

## Pushing to Docker Hub
- `docker tag myapp <dockerhub-username>/myapp` - tag the myapp image with your Docker Hub username
- `docker login` - log in to Docker Hub
- `docker push <dockerhub-username>/myapp` - push the image to Docker Hub


# Review Ingress
1. create deployment(name=webapp, label:app=webapp) and service (80:80)
2. create ingress resource

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webapp-ingress
  # When to Use nginx.ingress.kubernetes.io/rewrite-target
    # Path Rewriting: If you want to rewrite the incoming request path to a different path before forwarding it to the backend service, you should use this annotation.
    # Simplifying Backend Paths: This is useful when your backend service expects requests to a specific path, but you want to expose a different path to the clients.
  annotations:
    # This annotation rewrites the incoming request path /backend to / before forwarding it to the backend-service (the code).
    nginx.ingress.kubernetes.io/rewrite-target: /

spec:
  rules:
  # host must follow DNS naming conventions
  # can use a wildcard to match all subdomains (*.example.com) 
  # each host value must be unique if you define multiple rules with the same host, only the first rule will be used
  # host is an optional field, if not defined, the rule applies to all inbound HTTP traffic
  # host must match the Common Name or Subject Alternative Name in the SSL or TLS certificate
  # should match the domain purchased from a domain registrar
  - host: webapp.example.com # domain name (not dictated by the path)
    http:
      # the paths should match the routes in the application
      # by usign Prefix, you can access any path that starts with /backend without having to define each path individually 
      paths:
      - path: /backend # path to match for the rule to apply 
        pathType: Exact # this means you can only access the backend service by going to webapp.example.com/backend (no trailing slash - webapp.example.com/backend/api)
        backend:
          service:
            name: backend-service # name of the service to forward the request to 
            port:
              number: 80
      - path: /frontend
        pathType: Prefix
        backend: 
          service:
            name: frontend-service
            port:
              number: 80
  - host: example.com
    http:
      paths: 
      - path: /
        pathType: Prefix
        backend:
          service:
            name: webapp-service
            port:
              number: 80
```



![resource shortcuts](image.png)













The ReplicaSet "replicaset-2" is invalid: spec.template.metadata.labels: Invalid value: map[string]string{"tier":"nginx"}: `selector` does not match template `labels`