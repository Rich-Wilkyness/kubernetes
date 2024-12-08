# Resource Requirements
- the kube-scheduler is responsible for scheduling pods to nodes based on the resource requirements of the pods and the capacity of the nodes in the cluster 

- if a pod has a resource request, the kube-scheduler will only place the pod on a node that has enough available resources to meet the request
- if the resource limit is reached, you will get an error message FailedScheduling Insufficient cpu/memory
  - a pod cannot use more cpu than the limit, but it can use more memory. 
    - if it does this for too long, it will terminate (OOM = Out of Memory).

- each container has a resource request and a resource limit
    - the resource request is the amount of resources that the container needs to run
    - the resource limit is the maximum amount of resources that the container can use
        - if the resource limit is reached, the container will be terminated

- the resource requirements are specified in the pod definition file
    - the resource requirements are specified in the spec.containers.resources section of the pod definition file
    - the resource requirements can be specified in terms of CPU and memory
        - CPU is measured in cores
        - memory is measured in bytes
```yaml
apiVersion: v1
kind: Pod # must be capital
metadata:
  name: myapp-pod
  labels:
    app: myapp
    tier: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx 
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: 2 
      limits:
        memory: "128Mi"
        cpu: 3


```
# Resource Sizes
## CPU
- 1 AWS vCPU, 1 GCP Core, 1 Azure Core, 1 Hyperthread. These are all equivalent to 1 Kubernetes CPU
- CPU can be in millicores (200m = 0.2). 1 core = 1000m. you can only go as low as 1m.
## Memory
- 1 G (Gigabyte) = 1,000,000,000 bytes
- 1 M (Megabyte) = 1,000,000 bytes
- 1 K (Kilobyte) = 1,000 bytes

- 1 Gi (Gibibyte) = 1,073,741,824 bytes
- 1 Mi (Mebibyte) = 1,048,576 bytes
- 1 Ki (Kibibyte) = 1,024 bytes

- 256 Mi = 268435456 bytes = 268M.


- by default when creating a pod
    - there are no resource requests or limits set
    - the pod will be scheduled to any node that has enough available resources to run the pod
- if limits are set. Requests are set to the same value as the limit
- what if we do not want to unecessarily limit the pod?
    - set the request, but not the limit
    - this means that if 2 pods are running on a node, the minimum resources required for the pod to run will be available and any unused resources will be shared between the pods

# Resource Limit Range
- this will only affect new pods created after the limit range is created or updated
- cpu and memory can be broken up into 2 different files if you want

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: myapp-limit-range
spec:
  limits:
  - default: # this is the default limit
      memory: 512Mi
      cpu: 500m
    defaultRequest:
      memory: 256Mi
      cpu: 250m
    max:
      memory: 1Gi
      cpu: 1
    min:
      memory: 64Mi
      cpu: 100m
    type: Container
```

# Resource Quotas
- resource quotas can be used to limit the amount of resources that can be used in a namespace
    - this can be used to prevent a single pod from using all the resources in a namespace
    - If a resource quota is created after pods have already been deployed in the namespace, Kubernetes will enforce the quota on any new resource requests or changes to existing resources.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
  namespace: my-namespace
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```
