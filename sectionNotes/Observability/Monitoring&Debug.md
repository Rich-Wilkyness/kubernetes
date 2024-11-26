# Logging
- Docker command:
    - `docker run -d <container-name>`
    - then run: `docker logs -f ecf` to see the logs 
        - `-f` flag is for "follow", which will show the logs in real-time
- Kubernetes command:
    - create a pod with a container that logs
        - only containers have logs (not pods, nodes, deployments, etc.)
    - then run: `kubectl logs -f <pod-name>`
    - this will fail if the pod has more than one container
        - to see logs from a specific container, use: `kubectl logs -f <pod-name> <container-name>`


# Monitoring & Debugging
- I want to know how many nodes are running in my cluster
    - are they healthy?
    - resource usage? (CPU, memory, disk, network)
- I want to know how many pods are running in my cluster
    - are they healthy?
    - resource usage? (CPU, memory, disk, network)

- we want to measure and store this data
    - kubernetes does not provide this out of the box
    - there are many open-source tools that can help with this
        - Prometheus
        - Metrics Server
        - Elastic Stack
    - proprietary tools
        - Datadog
        - dynatrace

- only going to cover Metrics Server (minimal knowledge required for CKAD)
    - 1 metric server per cluster
    - gets metrics from each node and pod -> aggregates them --> stores them in-memory (no long-term storage)
        - no historical data
        - must use another tool for long-term storage
    - how does this work?
        - kubelet (kubernetes agent) runs on each node, this contains a subcomponent called cAdvisor
            - cAdvisor collects metrics about the node and pods
            - kubelet exposes these metrics on a REST endpoint
    - commands for minikube
        - `minikube addons enable metrics-server`
    - commands for all other services
        - git clone https://github.com/kubernetes-incubator/metrics-server
        - kubectl create/apply -f deploy/1.8+/
        - or go to the directory and run `kubectl create -f .`
    - commands to view metrics
        - `kubectl top node`
        - `kubectl top pod`