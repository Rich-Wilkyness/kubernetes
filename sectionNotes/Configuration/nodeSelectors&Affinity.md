# Node Selectors
- Node selectors are used to control which nodes a pod is eligible to be scheduled based on labels on the node. 
- Node selectors are defined in the pod.spec.nodeSelector field
- example at pods/pod-w-nodeSelector.yaml
- why use node selectors?
    - to ensure that pods are scheduled on nodes that have the required hardware
    - to ensure that pods are scheduled on nodes that are running the required software
    - to ensure that pods are scheduled on nodes that are running the required workloads
    - to ensure that pods are scheduled on nodes that are running the required services
- has limitations
    - it must match the labels on the node
    - you cannot say schedule on a node that is large or medium. you cannot say do not schedule on a node that is small
        - this is where node affinity comes in

- commands:
    - label a node:
        - kubectl label nodes <node-name> <label-key>=<label-value>

# Node Affinity
- Node affinity is a property of pods that attracts them to a set of nodes (either as a preference or a hard requirement).
- Node affinity is specified as pod.spec.affinity.nodeAffinity field
- example at pods/pod-w-nodeAffinity.yaml
- notice the difference in complexity between nodeSelector and nodeAffinity

- Node Affinity Types
    - requiredDuringSchedulingIgnoredDuringExecution
        - This indicates that the scheduler must place the pod on a node that meets the specified criteria when scheduling. If no suitable nodes are available, the pod will not be scheduled at all.    
        - The criteria are ignored once the pod is running. This means that if the node's labels change after the pod has started, it won't affect the running pod
    - preferredDuringSchedulingIgnoredDuringExecution
        - This indicates a soft preference for scheduling the pod on nodes that meet the specified criteria. The scheduler will attempt to place the pod on such nodes if they are available but will still schedule the pod on other nodes if necessary.
        - Like the previous case, the criteria are ignored once the pod is running.
    - requiredDuringSchedulingRequiredDuringExecution
        - This indicates a hard requirement for both scheduling and execution. The pod must be scheduled on a node that satisfies the criteria, and it must remain on that node throughout its execution.
        - If the node's labels change after the pod has started, the pod will be evicted from the node and will not be rescheduled.