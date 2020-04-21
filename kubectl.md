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

## Accessing resources

TODO
