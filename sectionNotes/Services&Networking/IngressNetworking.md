# Ingress
- Ingress is a Kubernetes resource that exposes HTTP and HTTPS routes from outside the cluster to services within the cluster. Traffic routing is controlled by rules defined on the Ingress resource.
- Ingress is not a service type, but rather a resource that manages external access to services in a cluster.
- provides users a single point of entry to access services within a cluster 
- can also implment SSL security, name-based virtual hosting, authentication, and path-based routing, among other things

# Big Picture
![Ingress Big Picture](image-2.png)

![Ingress Architecture](image-1.png)
# Ingress Architecture
- The image above shows the architecture of an Ingress in an application
- Ingress handles the top 3 rectangles (load balancers) inside kubernetes (box)

# Ingress Controller
- An Ingress Controller is a daemon that runs in a Kubernetes cluster and watches for Ingress resources to create load balancers and route traffic to the appropriate services. 
- examples: GCE, Nginx, Traefik, HAProxy, Contour, Istio and Envoy
- GCE and Nginx are the most popular Ingress Controllers in use today
    - both are being supported and maintained by the Kubernetes community
- a kubernetes cluster does not come with an Ingress Controller by default, so you must install one yourself (above examples)

# Ingress Resources
- Ingress resources are used to define how external traffic is routed to services within the cluster (rules)
    - rules are defined using hostnames and paths to match incoming requests to services within the cluster 
    - example: www.my-online-store.com/wear is routed to the `wear` service while www.my-online-store.com/electronics is routed to the `electronics` service
    - example 2: wear.my-online-store.com is routed to the `wear` service while electronics.my-online-store.com is routed to the `electronics` service
- Ingress resources are defined in YAML files and are created using the `kubectl apply -f <filename>` command




<!-- Ingress updates 
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-
https://kubernetes.io/docs/concepts/services-networking/ingress
https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types 
-->
<!-- available during the exam:
 https://helm.sh/docs/
 https://kubernetes.io/docs/
 https://github.com/kubernetes/
 https://kubernetes.io/blog/ 
 -->