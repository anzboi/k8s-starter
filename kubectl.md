# kubectl

kubectl is the command line tool used to interact with kubernetes from the command line. It can be used to inspect what resources are loaded in the cluster, edit and delete them. It can also inspect what services are exposed, pods are running, whether or not deployments failed, etc.

## Installation

* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)
* [kubectx and kubens](https://github.com/ahmetb/kubectx#installation)

## Config

config for kubectl is not trivial. It needs to be configured to talk to a particular cluster, which may require certain access tokens, proxy configuration, etc. You may also wish to switch between clusters. Most clusters you work with will automatically attach their config to your system.

The `kubectx` command can be used to easily switch between kubectl configs. Docker for desktop also does this.

### gcp clusters

GCP provides a convenient way to configure kubectl. Go to the console, find the cluster you wish to connect to, and click `connect`. THis will give you a `gcloud` command to configure kubectl

### local clusters

Local clusters like kind, minikube and docker-desktop automatically configure kubectl for you.

## Using kubectl

### namespace

Everything you do with kubectl needs to happen to a particular namespace. Any command you use can use a `--namespace` or `-n` flag. For example, if you want to see pods in the `default` namespace, you can use `kubectl -n default get pod`. If you don't use `-n`, kubectl will use the default namespace

If you use `kubens`, you can set the default namespace you are working with

```bash
kubectl get pod # gets pods in default namespace
kubens my-namespace
kubectl get pod # gets pods in my-namespace
```

finally, you can access all resources across all namespaces using the `--all-namespaces` flag

### Applying manifests

With a set of yaml manifests ready to go, you can send them to kubernetes to get applied via the `apply` command

```bash
kubectl apply -f path/to/manifest.yaml
```

### Accessing resources

Resources are accessed with the get command

```
kubectl get [resource type] [options]
```

This is your primary tool of discovery. You can get a broad overview of whats in the cluster, or investigate a particular resource further

```bash
kubectl get pod --all-namespaces # list all pods in the cluster
kubectl get secret # list secrets in current namespace
kubectl get pod [pod-id] -o yaml # inspect the full manifest for a specific pod
```

An extremely common use of `get` is to find a particular pod you wish to debug with subsequent commands.

```bash
$ kubectl get pod
list of pod ids
```

#### describe

The yaml output of `get` can be pretty verbose, you can get a nicer formatted description of a resource with `describe`

```bash
kubectl describe pod [pod-id]
```

### Editing

Resource manifests can be edited on the fly with `edit`. This is extremely helpful when debugging issues in a cluster and you need to quickly try out something.

```bash
kubectl edit [resource type] [resouce id]
```

### Deleting

Resources can be delete with `delete`.

```bash
kubectl delete pod [pod-id] # deletes a pod
kubectl delete pod -l=app=my-service # deletes all pods with the label app: my-service
```

A common use of delete is to 'bounce' a pod. If a pod is created with a deployment, then delete the pod will cause the deployment to automatically create a new one. This is basically a 'turn it off and on again' for kuberenetes containers.

### Exec

You can execute commands directly inside a container with `exec`. This can be any command the container knows about, but is commonly used to get a shell in the container (this is known as 'exec'-ing into a container)

```bash
$ kubectl exec -it [pod-id] -- /bin/sh # gets an sh shell in the container
```

the `-it` option connects your terminal to the shell you are starting in the container

### Port-Forward

port-forward is used to connect a port on you machine to a port in a kubernetes container. This allows you to test a container by sending requests to it directly without having to worry about going through cluster ingress.

```bash
# exposes container 8080 to local 8080
kubectl port-forward [pod-id] 8080:8080
```

### Scale

A kubernetes replica set or deployment can be scaled using `scale`

```bash
# scales the deployment to 3 instances
kubectl scale deployment [deployment-name] --replicas=3
```
