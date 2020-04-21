# ANZBOI Kubernetes crash course

## What is kubernetes

Container orchstration engine. Provides a high level API to tell a cluster to run, manage and provide networking infrastructure for containers.

The kubernetes API is largely a declarative API that allows you to specify the state you wish you cluster to exist in, rather than operations you want the cluster to perform. For example, a deployment defines a set of containers that you wish to be running, as well as what context you want them to run in. You don't tell a cluster to deploy a container, you tell the cluster there is a deployed container, and it tries to fulfill that deployment declaration.

The way you interact with kubernetes through the API is entirely through kubernetes resources. These resources define the state that kubernetes should be in, and the cluster will work to reach that state.

## Kubernetes resources

There are a few base resources anyone deploying containers should be aware of.

### [Node](https://kubernetes.io/docs/concepts/architecture/nodes/)

A node is an abstraction of a physical machine. In a real cluster, nodes would usually map one-to-one with a physical machine. The node exposes information about how much CPU and RAM is available on that machine so the kubernetes API can know whether a container with certain resource requirements can be deployed on a node.

### [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

A kubernetes namespace is a logical grouping of resources based on a common attribute between them. Namespaces exist primarily as a way to separate different resources, and control access to them.

Namespaces are also exclusive. Every kubernetes resource belong to EXACTLY ONE namespace. Membership to a particular namespace can affects many things, what other kubernetes resources you can see, whether you can send a request to a pod in another namespace, etc.

### [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)

A pod is an abstraction on top of a set of running containers. Pods are the basic unit of running containers as seen by kubernetes. A pod has a single cluster IP address and set of ports, and processes in a pod can communicate over localhost or POSIX shared memory.

Pods may contain multiple containers if it makes sense for multiple processes to be running in the same context (eg: a service with istio proxy).

Pods SHOULD be considered volatile. A cluster can decide at any moment to bring a pod down or create a new one.

### [Replica Set](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

A replica set is a set of identical pods. It allows kubernetes to take a single pod template and deploy multiple identical pods. In practice, replica sets are managed by the more useful **Deployment** resource.

### [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

Deployments are very similar in function to replica sets, but are much nicer to work with. The biggest advantage a deployment offers is rolling updates, the ability to roll out an update wth 0 downtime. It does this by managing multiple replica sets, one old and one new, and waiting until the new deployment is complete before deleting the old replica set.

### [Service](https://kubernetes.io/docs/concepts/services-networking/service/)

A service is an abstraction of a real running service that may be running on multiple pods. The goal of a service is to expose your application as a single entity to other applications in the cluster.

To do this it does two things

1. Uses a selector to identify which pods to send requests to
2. Provides a single kubedns entry and load balancing.

### [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

Ingress defines how requests come into the cluster. They provide a place where routing can ocurr to connect an external `host+path` to the relevant internal services

At ANZ we don't use native ingress resources, rather we use istio resources to manage more rich networking, ingress and egress.

### [Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

Storage is a method to provide persisted data to a pod. In practice, volumes are not used much to store real business data (you should use a database), but there are two commonly used types of Volumes

#### [ConfigMap](https://kubernetes.io/docs/concepts/storage/volumes/#configmap)

A configmap is typically used for storing configuration of a container. Data in a config map SHOULD NOT be considered secure

#### [Secret](https://kubernetes.io/docs/concepts/configuration/secret/)

A secret stores secure information, such as passwords, tokens, keys, etc.

Secrets are by nature difficult to manage. At ANZ we use a resource type `SealedSecret` to encrypt a secret that can only be decrypted in the cluster.

Both config maps and secrets can be volumed into a container, either as a file or set of environment variables.

## Manifests and deploying services

As previously mentioned. The primary way you interact with a cluster is via creating, updating or deleting resources. All of the resources above are described via yaml blobs. A collection of these blobs is typically called a manifest (think the documented cargo an ocean freighter). Deploying a service is essentially a process of creating these manifests, then sending it to the cluster.

As a basic example, suppose you just wrote a service call `my-service`, and you would like to deploy 3 instance of it to your cluster exposed on port 8080. You might write something like the following

```yaml
# manifest.yaml
---
apiVersion: apps/v1
kind: Deployment # how to run the service
metadata:
  name: my-service
  namespace: default
  labels:
    app: my-service
spec:
  # 3 instances of the service
  replicas: 3

  # this defines each pod
  template:
    metadata:
      name: my-service
      namespace: default
      labels:
        app: my-service
    spec:
      containers:
        - name: my-service
          image: my-service:1.0.0
          imagePullPolicy: Always
          ports:
            - 8080
---
apiVersion: v1
kind: Service # Exposing the service
metadata:
  name: my-service
  namespace: default
  labels:
    app: my-service
spec:
  selector:
    label:
      app: my-service
  ports:
    - name: grpc-my-service
      port: 8080
      containerPort: 8080
      protocol: TCP
```

This will deploy 3 instances of `my-service` with the image of your choice, and exposes the following url `http://my-service.default.svc.cluster.local:8080`. This is NOT accessible outside the cluster yet, only internally. For that you need more setup and to understand more about the cluster you are deploying to.

### Helm

I will briefly mention helm here. Helm is a tool that makes creating kubernetes manifests easier by providing a way to template them. Helm actually attempts to do much more than this, trying to manage running services in the cluster as well, but we don't use those features, but we don't use those features normally.
