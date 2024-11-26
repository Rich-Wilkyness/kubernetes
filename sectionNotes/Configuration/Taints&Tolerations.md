# Taints and Tolerations
- Taints and tolerations are used to ensure that pods are not scheduled (by the schedular) on inappropriate nodes.
- analogy: taints are like a bad smell that repels pods, and tolerations are like a gas mask that allows pods to tolerate the bad smell and be scheduled on the node.
- these are mainly used in the following scenarios:
    - to prevent pods from being scheduled on nodes that do not have the required hardware
    - to prevent pods from being scheduled on nodes that are already running critical workloads
    - to prevent pods from being scheduled on nodes that are running non-critical workloads

- by default, there are no taints on nodes and no tolerations on pods

- commands
    - taint a node:
        `kubectl taint nodes <node-name> <key>=<value>:<taint-effect>` 
        - keys must be defined, but values are optional
        - taint-effect determines what happens to the pods that DO NOT tolerate the taint
        - taint-effect can be one of the following:
            - NoSchedule: pods will not be scheduled on the node
            - PreferNoSchedule: the scheduler will try to avoid placing pods on the node
            - NoExecute: existing pods on the node will be evicted if they do not tolerate the tainted node
    - remove a taint from a node:
        - `kubectl taint nodes <node-name> <key>:<value>:<taint-effect>-`
        - the "-" at the end of the command removes the taint
        - if the taint-effect is not specified, all taints with the specified key will be removed
        - if a value is not specified, all taints with the specified key will be removed
        - to find the key, value, and effect, use `kubectl describe node <node-name> | grep Taint` or search through the describe output

- tolerations are defined in the pod.spec.tolerations field 
    - example at pods/pod-w-toleration.yaml

- we have a master node, but the schedular does not schedule pods here. 
    - why? a taint has been applied to the master node automatically by the k8s control plane
    - `kubectl describe node <node-name> | grep Taint` to see the taints on a node (master = kubemaster)
    - it is recommended to not run pods on the master node, as it is a single point of failure


- Examples:
    1. Ensuring that certain nodes are dedicated to specific types of workloads, such as backend services, frontend services, or databases.
        - `kubectl taint nodes node1 dedicated=backend:NoSchedule`
        - ```yaml
            tolerations:
            key: "dedicated"
            operator: "Equal"
            value: "backend"
            effect: "NoSchedule"
        ```
    2. Ensuring that critical workloads run on specific nodes and are not affected by other less critical workloads.
        - `kubectl taint nodes node2 critical=true:NoSchedule`
        - ```yaml
            tolerations:
            - key: "critical"
            operator: "Equal"
            value: "true"
            effect: "NoSchedule"
        ```
    3. Temporarily preventing new pods from being scheduled on a node that is undergoing maintenance.
        - `kubectl taint nodes node3 maintenance=true:NoSchedule`
        - ```yaml
            tolerations:
            - key: "maintenance"
            operator: "Equal"
            value: "true"
            effect: "NoSchedule"
        ``` 
    4. Evicting all pods from a node that is being decommissioned.
        - `kubectl taint nodes node4 decommissioned=true:NoExecute`
    5. Ensuring that nodes with specific hardware or resource constraints are used only by compatible pods.
        - `kubectl taint nodes node5 gpu=true:NoSchedule`
        - ```yaml
            tolerations:
            - key: "gpu"
            operator: "Equal"
            value: "true"
            effect: "NoSchedule"
        ```

