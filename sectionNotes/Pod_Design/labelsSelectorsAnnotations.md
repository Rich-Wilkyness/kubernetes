# Labels and Selectors
- Labels are key/value pairs that are attached to objects, such as pods. They are used to organize and to select subsets of objects.
- on objects they are under metadata.labels
- replicaSets and deployments use selectors to determine which pods they manage
    - in the example below, the replicaSet is managing pods with the label app: myapp which is the same as the label in the pod template
    - we are not matching to the labels on the replicSet.metadata.labels
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicaset
  labels:
    app: myapp-replicaset
    type: front-end
spec:
  selector:
    matchLabels:
      app: myapp
    template:
      metadata:
        labels:
          app: myapp
```
          
- command:
    - kubectl get pods --selector env=prod

# Annotations
- Annotations are key/value pairs that are attached to objects, such as pods. They are used to attach non-identifying metadata to objects.
- on objects they are under metadata.annotations
- they are not used to select objects