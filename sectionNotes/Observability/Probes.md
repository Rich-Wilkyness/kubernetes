# Readiness and Liveness Probes
# Readiness Probes
- Pod lifecyle
    - Pending -> ContainerCreating -> Running -> Succeeded/Failed -> Terminating
    - can use the "get" or "describe" command to see the "Status" of a pod
- POD Conditions - these are True/False states
    - PodScheduled -> Initialized -> ContainersReady -> Ready
    - can use the "describe" command to see the "Conditions" of a pod     
    - or use the "get" command to see the "Ready" of a pod

- What does Ready mean?
    - Ready means that the pod is ready to accept traffic
    - This is not necessarily true. A pod can say Ready, but not be ready to accept traffic
        - example: Jenkins pod is ready, but the Jenkins app is still starting up and users cannot access it 
- How do we tie the Readiness of a pod to the "Ready" condition?
    - We use Readiness Probes (Tests)
    - Readiness Probes are used to determine if a pod is ready to accept traffic
    - If the Readiness Probe fails, the pod is not ready to accept traffic
    - If the Readiness Probe passes, the pod is ready to accept traffic
    - Examples of Readiness Probes:
        - HTTP GET request (web server)
        - TCP Socket (database connection)
        - Command (script)
- Example of Pod at /pods/pod-w-readyProbe.yaml

```yaml
apiVersion: v1
kind: Pod 
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
    readinessProbe:
      httpGet:
        path: /index.html
        port: 80
      tcpSocket:
        port: 3306
      exec:
        command:
        - cat
        - /app/is_ready
      initialDelaySeconds: 5 # wait 5 seconds before running the probe (if there is a bootup time for the container)
      periodSeconds: 5 # run the probe every 5 seconds 
      failureThreshold: 3 # if the probe fails 3 times, the container will be marked as not ready
```

# Liveness Probes
- tests the health of a pod and restarts the pod if the test fails 
- this is useful for applications that are not able to recover from a failure on their own 
    - it runs a test, and if the test fails, the pod is deleted and a new pod is created 
- Examples of Liveness Probes:
    - HTTP GET request (web server)
    - TCP Socket (database connection)
    - Command (script)
- Example of Pod at /pods/pod-w-liveProbe.yaml

```yaml
apiVersion: v1
kind: Pod 
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
    livenessProbe:
      httpGet:
        path: /index.html
        port: 80
      tcpSocket:
        port: 3306
      exec:
        command:
        - cat
        - /app/is_ready
```

- As you can see, the only difference between the two files is the readinessProbe field in the first file is replaced with livenessProbe in the second file.