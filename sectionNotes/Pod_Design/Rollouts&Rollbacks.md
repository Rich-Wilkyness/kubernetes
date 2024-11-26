# Rollout and Versioning
- Deployment commands:
    - kubectl create -f deployment-definition.yml --save-config 
        - creates an initial deployment and saves the configuration in the deployment object (annotations), you need to do this if you want to use apply
    - kubectl apply -f deployment-definition.yml
        - rollout changes to a deployment
        - this will compare the current deployment to the new deployment and make the necessary changes
        - Immutable Fields: fields that cannot be changed after the object is created
            - Deployments: 
                - spec.selector.matchLabels: 
                    - The selector labels used to identify the pods managed by the deployment.
                - spec.template.metadata.labels: 
                    - The labels applied to the pod template.
                - spec.template.spec.containers[*].name: 
                    - The name of the containers within the pod template.
                - spec.template.spec.initContainers[*].name: 
                    - The name of the init containers within the pod template.
                - spec.strategy.type: 
                    - The strategy used to replace old pods with new ones (e.g., RollingUpdate, Recreate).
                    - default is RollingUpdate
            - StatefulSets:
                - spec.selector.matchLabels: 
                    - The selector labels used to identify the pods managed by the StatefulSet.
                - spec.template.metadata.labels: 
                    - The labels applied to the pod template.
                - spec.template.spec.containers[*].name: 
                    - The name of the containers within the pod template.
                - spec.template.spec.initContainers[*].name: 
                    - The name of the init containers within the pod template.
                - spec.serviceName: 
                    - The name of the service that governs this StatefulSet.
            - PersistentVolumeClaims:
                - spec.storageClassName: 
                    - The storage class of the PVC.
                - spec.accessModes: 
                    - The access modes of the PVC (ReadWriteOnce, ReadOnlyMany, ReadWriteMany).
                - spec.resources.requests.storage: 
                    - The requested storage size for the PVC.
            - PersistentVolumes: 
                - spec.capacity: 
                    - The capacity of the PV.
                - spec.capacity.storage: 
                    - The storage capacity of the PV.
                - spec.accessModes: 
                    - The access modes of the PV (ReadWriteOnce, ReadOnlyMany, ReadWriteMany).
                - spec.persistentVolumeReclaimPolicy: 
                    - The reclaim policy of the PV (Retain, Recycle, Delete).
                - spec.storageClassName: 
                    - The storage class of the PV.
            - Services:
                - spec.type: 
                    - The type of the service (NodePort, LoadBalancer, ClusterIP).
                - spec.clusterIP: 
                    - The cluster IP address of the service (if set to a specific value).


    - can add --record to keep track of changes
        - kubectl apply -f deployment-definition.yml --record
        - Change-cause will be recorded in the deployment history for each change made to the deployment using --record flag 
    - kubernetes scale deployment <deployment-name> --replicas=<number-of-replicas>
        - careful as this does not change the file
    
    
- Query commands:
    - kubectl rollout status <object-type>/<object-name> 
        - kubectl rollout status deployment/myapp
    - kubectl rollout history <object-type>/<object-name>
    - kubectl rollout history <object-type>/<object-name> --revision=<revision-number>
        - check the history of a specific deployment
- Undo/rollback commands:
    - kubectl rollout undo <object-type>/<object-name>
    - kubectl rollout undo <object-type>/<object-name> --to-revision=<revision-number>



# Rollout Strategies
1. Delete and Recreate
    - delete all pods and create new ones
    - downtime
    - not recommended
2. Rolling Update
    - default strategy
    - update pods in a controlled manner
    - no downtime
    - canary deployments
3. Blue/Green Deployment (video is really good for visualizing this)
    - have two identical environments (except for the version changes)
    - one is live (old), the other is not (new)
    - switch to the new deployment once it is ready 
    - no downtime
    - HOW TO USE IT?
        - We use a service to direct traffic to the old deployment
        - Once the new deployment is ready, we switch the service to direct traffic to the new deployment
        - This is done by changing the selector in the service 
            - the Service could be looking for version=v1 on the deployment, then we switch it to version=v2
            - remember to label the deployments with what the selector is looking for
        - This is a manual process and is not provided by kubernetes
4. Canary Deployments (video is really good for visualizing this)
    - deploy a new version of an application to a small subset of users
        - 
    - monitor the new version for issues
    - if no issues, deploy to the rest of the users
    - if issues, rollback to the previous version
