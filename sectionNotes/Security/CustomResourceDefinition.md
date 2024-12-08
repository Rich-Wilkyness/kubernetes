# Custom Resource Definition
- deployment controller is responsible for creating the pods defined by the resources in the etcd database
    - it continuously monitors the etcd database for changes to the desired state of the cluster
    - there are specific controllers for each resource type

# Create a Custom Resource Definition (CRD)
- To create a custom resource, you must first define a Custom Resource Definition (CRD) resource object
    - CRD is an `API` extension that allows you to define your own resource types
    - CRD is a resource that defines a new resource type and its schema
    - once a CRD is created, you can create instances of the new resource type
    - CRD is a Kubernetes object that is stored in the etcd database

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flight.com
spec:
  scope: Namespaced # or Cluster
  group: flights.com
  names:
    kind: FlightTicket
    plural: flighttickets
    singular: flightticket
    shortNames:
    - ft
  versions:
    - name: v1
      served: true
      storage: true # only 1 version can be served and stored

      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                from:
                  type: string
                to:
                  type: string
                number:
                  type: integer
                  minimum: 1
                  maximum: 10

---
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: flightticket-sample
spec:
    from: LAX
    to: JFK
    number: 7
```

# Create a Custom Controller (not on the exam)
- second, create a controller that watches for the custom resource and takes action when a new instance is created
    - a controller is any process or code that runs in a loop continuously monitoring the state of the kubernetes cluster, listening to events of specific objects being changed
    - one can create a controller in any laguage, but using Go is recommended 
        - Go has a client library that makes it easy to interact with the Kubernetes API
            - shared informers, dynamic clients, and other tools are available (provide caching, rate limiting, and queueing mechanisms)
        - Kubernetes itself is written in Go
        - other languages can get expensive without the client library
    - the controller is responsible for creating the pods defined by the custom resource
    - the controller is a custom controller that watches for the custom resource and takes action when a new instance is created
    - the controller is a Kubernetes object that is stored in the etcd database
    - without the controller, the resource will sit as data in the etcd database and nothing will happen

    - example given at kubernetes/sample-controller
        - clone this repo and then make custom changes to the controller
    - next, build the controller and create a docker image
    - finally, deploy the controller to the cluster 
        - ./sample-controller -kubeconfig=$HOME/.kube/config 
            - we specify the kubeconfig file to use to connect to the cluster (authenticates the controller to the cluster)


# Operators (not on the exam)
- packages a CRD and a controller into a single unit 
- public operators are available at operatorhub.io
