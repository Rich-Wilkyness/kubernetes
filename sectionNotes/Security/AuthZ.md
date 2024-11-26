# API Groups
- **API Groups** are a way to organize resources in Kubernetes. They are used to group related resources together. For example, the `Pod` resource is in the `v1` API group, and the `Deployment` resource is in the `apps/v1` API group.
- kubernetes is organized into API groups. Each API group is a collection of resources that are related and share a common purpose.
    - /metrics, /healthz, /logs, /version, /api, /apis are all API groups. 

- We will focus on /api and /apis
    - /api (core = /v1) is the core API group. It contains the core resources like: 
        - pods, services, namespaces, rc, endpoints, events, nodes, bindings, PV, PVC, configmaps, secrets, services.

    - /apis (named) is the extended API group.
        - it is more organized and scalable than the core API group.
        - all newer kubernetes features will be through the /apis group.
        - examples: /apps, /extensions, /networking.k8s.io, /authentication.k8s.io, /certificates.k8s.io
            - /apps --> /deployments, /statefulsets, /daemonsets, /replicasets 
                - these are considered resources that are part of the /apps API group.
                - each resource has a set of operations (verbs) that can be performed on it, like get, create, read, update, delete, list, watch, etc.
            - /networking.k8s.io --> /networkpolicies
            - /certificates.k8s.io --> /certificatesigningrequests


- to access these resources you can use kubectl proxy or a curl command with the appropriate authentication and authorization. 
    - `kubectl proxy` will create a proxy server on your local machine that will forward requests to the kubernetes API server. 
    - you cn then access the resources through the proxy server. `curl http://localhost:8001/apis/apps/v1/deployments`
        - kubernetes runs on port 6443 by default but is accessible through the proxy server on port 8001.
    - Kube proxy != Kubectl Proxy

# Authorization
- **Authorization** is the process of determining what actions a user can perform on a resource. It is the process of granting or denying access to a resource based on the user's identity and the resource's policies.
- Admins can do all actions on all resources. 
- Users (developers, testers, bots, etc.) can only do actions on resources they have been granted access to.
    - We want to restrict access to only what is necessary for the user to do their job.
- We set the Authorization mode in the kube-apiserver.yaml file. 
    - `--authorization-mode=AlwaysAllow` or `--authorization-mode=AlwaysDeny`
    - `--authorization-mode=RBAC` or `--authorization-mode=ABAC` or `--authorization-mode=Webhook` or `--authorization-mode=Node`
    - or a combination of these modes. `--authorization-mode=RBAC,Node`
        - the order of the modes is important. The first mode that grants access will be used.
        - if the first mode denies access, the next mode will be checked.

# Different ways to authorize users:
## Role-Based Access Control (RBAC)
- Is a method of regulating access to resources based on the roles of individual users within an organization. 
- Roles are namespaced - they are specific to a namespace and must be created in each namespace where they are used.
- In Kubernetes, RBAC uses roles and role bindings to control access to resources.
    - define roles instead of users/groups and then bind the roles to users/groups.
    - Example: Developer role can read/write to deployments, but not to secrets. 
    - When a change is made to the role, it is immediately applied to all users/groups that are bound to that role (no need to restart the kube-apiserver).
    - Example:
        ```yaml
        apiVersion: rbac.authorization.k8s.io/v1
        kind: Role # can also be a ClusterRole for cluster-wide permissions
        metadata:
            name: developer
            namespace: my-app # if not specified it will be in the default namespace
        rules:
        - apiGroups: [""]
            resources: ["pods"]
            verbs: ["get", "list", "create", "delete", "update", "watch", "patch"] # each resource has a set of operations that can be performed on it 
            resourceNames: ["blue-pod", "orange-pod"] # restrict access to a specific resource on a namespace
        - apiGroups: ["apps"] 
            resources: ["deployments"]
            verbs: ["get", "list", "create", "delete", "update"]
        ---
        apiVersion: rbac.authorization.k8s.io/v1
        kind: RoleBinding # this will bind the role to a user/group
        metadata:
            name: dev-user-developer-binding
            namespace: my-app 
        subjects:
        - kind: User # can be a user, group, or service account
            name: alice
            apiGroup: rbac.authorization.k8s.io
        roleRef:
            kind: Role
            name: developer
            apiGroup: rbac.authorization.k8s.io
        ```
    - **Cluster Roles**
        - resources are either namespaced or cluster-scoped.
        - kubernetes sets up default ClusterRoles when it is installed. 
        - Examples of Cluster-scoped resources: nodes, PV, clusterroles, clusterrolebindings, namespaces, certificatesigningrequests, etc.
        - Examples of Namespaced resources: pods, services, deployments, configmaps, secrets, replicasets, jobs, PVC, roles, rolebindings, etc.
        - ClusterRoles are used to define permissions that apply across the entire cluster, regardless of namespace.
            - can be used to grant permissions to resources that are namespaced or cluster-scoped.
            ```yaml
            apiVersion: rbac.authorization.k8s.io/v1
            kind: ClusterRole
            metadata:
              name: cluster-admin
            rules:
            - apiGroups: [""]
              resources: ["nodes"]
              verbs: ["list", "get", "watch", "create", "update", "delete"]
            ---
            apiVersion: rbac.authorization.k8s.io/v1
            kind: ClusterRoleBinding
            metadata:
              name: alice-cluster-admin-binding
            subjects:
            - kind: User
              name: alice
              apiGroup: rbac.authorization.k8s.io
            roleRef:
              kind: ClusterRole
              name: cluster-admin
              apiGroup: rbac.authorization.k8s.io
            ```
        

- Pros: 
    - Simplicity: Easy to understand and implement.
    - Scalability: Suitable for large organizations with many users and roles.
    - Granularity: Allows fine-grained control over access to resources.
    - Standardization: Widely adopted and supported by many systems.
- Cons:
    - Complexity: Can be difficult to manage and maintain.
    - Overhead: Requires additional resources to manage roles and permissions.
    - Inflexibility: Can be difficult to modify roles and permissions once they are set.

- Commands: 
    - `kubectl get roles` - list all roles in the cluster
    - `kubectl get rolebindings` - list all rolebindings in the cluster
    - `kubectl describe role <role-name>` - get details about a specific role
    - `kubectl auth can-i create deployments --as alice` - check if alice can create deployments
        - `--as` is used to specify the user (only admin), if not specified it will use the current user.


## Attribute-Based Access Control (ABAC)
- Is a method of regulating access based on attributes of the user, the resource, and the environment. 
- In Kubernetes, ABAC policies are defined using JSON files that specify the attributes and conditions for access. 
    - everytime a change is made to the policy, the kube-apiserver must be restarted.
    - Example: 
        ```json
        {
            "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
            "kind": "Policy",
            "spec": {
                "user": "alice", // can be a user, group, or service account
                "resource": "pods", // can be a resource, namespace, or cluster
                "namespace": "default", 
                "readonly": true // can be true or false 
            }
        }
        ```
- Pros:
    - Flexibility: Allows for complex access control policies based on multiple attributes.
    - Granularity: Provides fine-grained control over access to resources.
    - Customization: Can be customized to meet the specific needs of an organization.
- Cons:
    - Complexity: More complex to implement and manage compared to RBAC.
    - Performance: Requires additional resources to evaluate policies.
    - Inflexibility: Can be difficult to modify policies once they are set.
    - ABAC is not recommended for production use.

## Webhook
- allows Kubernetes to delegate authorization decisions to an external service. When a request is made, Kubernetes sends an HTTP request to the external service, which then returns an authorization decision.
- Pros:
    - Extensibility: Allows for custom authorization logic and integration with external systems.
    - Flexibility: Can implement complex and dynamic authorization policies.
- Cons:
    - Complexity: Requires additional infrastructure to manage the external service.
    - Performance: Adds latency to authorization decisions.
    - Security: Requires secure communication between Kubernetes and the external service.

## Node Authorizer
- Node authorization is disabled by default and is not recommended for production use.
- is a special-purpose authorizer in Kubernetes that restricts nodes to only modify their own resources. It is designed to limit the permissions of kubelets and nodes.
- Pros:
    - Security: Enhances security by limiting node permissions to only their own resources.
    - Simplicity: Simple to understand and implement for its specific use case.
- Cons:
    - Limited Scope: Only applicable to node and kubelet permissions, not suitable for general-purpose authorization.

## Always Allow 
- doesnt check for permissions. Default behavior.

## Always Deny 
- denies all requests.


# Admission Controllers
- **Admission Controllers** are plugins in the Kubernetes API server that intercept requests to the API server before they are persisted to the etcd database. They can be used to enforce policies, validate requests, and modify resources before they are created or updated.
- Examples:
    - Only permit images from a specific registry.
    - Do not permit pods to run as root user.
    - Only permit certain capabilities in the pod.
    - Enforce that all pods have resource limits, labels, etc.

- There are several enabled default controllers 
    - AlwaysPullImages - ensures that all pods have their images pulled before they are started.
    - DefaultStorageClass - sets the default storage class for persistent volumes.
    - EventRateLimit - limits the rate of events that can be sent to the API server.
    - NamespaceExists - ensures that namespaces exist before resources are created in them.

- There are a few controllers that are disabled by default
    - NamespaceAutoProvision - automatically creates namespaces when resources are created in them.

- To check which controllers are enabled, you can run:
    - `kube-apiserver -h | grep enable-admission-plugins`
    - If running in a Kube ADM setup:
        - `kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins`

- To update which controllers are enabled, you can update the kube-apiserver.yaml file. (this is for ADM setup)
    - /etc/kubernetes/manifests/kube-apiserver.yaml
    - go to spec.containers.command and add the `--enable-admission-plugins` flag with the desired controllers.
        - Example: `--enable-admission-plugins=AlwaysPullImages,DefaultStorageClass,EventRateLimit,NamespaceExists,NamespaceAutoProvision`
    - when editing the file, the kube-apiserver will automatically restart and apply the changes. but might take a few seconds to come back up.

- Note that the NamespaceExists and NamespaceAutoProvision admission controllers are deprecated and now replaced by NamespaceLifecycle admission controller.
- The NamespaceLifecycle admission controller will make sure that requests to a non-existent namespace is rejected and that the default namespaces such as default, kube-system and kube-public cannot be deleted.

## Validating Admission Controllers
- validates the object to ensure that it meets the requirements of the controller.
- Example: 
    - if a pod is created with a label that is not allowed, the ValidatingAdmissionController can reject the pod creation request.
    - if a pod is created with a resource limit that is too high, the ValidatingAdmissionController can reject the pod creation request.
- ValidatingAdmissionWebhook
    - is a special type of validating admission controller that allows Kubernetes to delegate validation decisions to an external/cluster service. When a request is made, Kubernetes sends an HTTP request to the external service, which then returns a validation decision.
    - Setup: 
        - Deploy a Webook Server
            - can be written in any language that can handle HTTP requests.
            - must accept the mutate and validate requests from the kube-apiserver.
            - must respond with a JSON object that contains the mutated or validated object.

        - Example:
            ```python
            @app.route('/validate', methods=['POST'])
            def validate():
                object_name = request.json['request']['object']['metadata']['name']
                user_name = request.json['request']['userInfo']['username']
                status = True
                if object_name == user_name:
                    status = False
                    message = "Object name cannot be the same as the user name."
                return jsonify({
                    "response": {
                        "allowed": status,
                        "uid": request.json['request']['uid'],
                        "status": {
                            "message": message
                        }
                    }
                })
            
            @app.route('/mutate', methods=['POST'])
            def mutate():
                user_name = request.json['request']['userInfo']['username']
                patch = [{"op": "add", "path": "/metadata/labels", "value": {"user": user_name}}]
                return jsonify({
                    "response": {
                        "allowed": True,
                        "uid": request.json['request']['uid'],
                        "patchType": "JSONPatch",
                        "patch": base64.b64encode(patch),
                    }
                })
            ```
        - Deploy the Webhook Server to Kubernetes
            - create a deployment and service for the webhook server so that it can be accessed by the kube-apiserver.
            - create a MutatingWebhookConfiguration or ValidatingWebhookConfiguration that points to the webhook server.
                ```yaml
                apiVersion: admissionregistration.k8s.io/v1
                kind: ValidatingWebhookConfiguration
                metadata:
                    name: my-validating-webhook
                webhooks:
                - name: "pod-policy.example.com"
                    clientConfig:
                        # can use url or service. 
                        # url is if the webhook server is external to the cluster.
                        # service is if the webhook server is in the cluster.
                        url: "https://my-webhook-service.default.svc:443/validate" 
                        service:
                            name: "my-webhook-service"
                            namespace: "webook-service"
                        caBundle: "base64-encoded-ca-cert" # must setup the ca cert for the webhook server, this is a TLS secret
                        # kubectl -n <namespace> create secret tls <name-of-cert> \
                        #    --cert "/root/keys/webhook-server-tls.crt" \
                        #    --key "/root/keys/webhook-server-tls.key"
                    rules: # these specify when the webhook should be called
                    - apiGroups: [""]
                      apiVersions: ["v1"]
                      operations: ["CREATE", "UPDATE"]
                      resources: ["pods"]
                      scope: "Namespaced"
                ```


## Mutating Admission Controllers
- mutates the object to meet the requirements of the controller.
- Example: 
    - if a pod is created without a label, the MutatingAdmissionController can add a default label to the pod.
    - if a pod is created without a resource limit, the MutatingAdmissionController can add a default resource limit to the pod.
- MutatingAdmissionWebhook
    - is a special type of mutating admission controller that allows Kubernetes to delegate mutation decisions to an external/cluster service. When a request is made, Kubernetes sends an HTTP request to the external service, which then returns a mutated object.


- Mutation Controllers are applied first, then Validation Controllers. This is because the object must be mutated before it can be validated.

