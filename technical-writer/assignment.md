# Debug Your Kubernetes Application

Kubernetes can simplify a lot of application deployment.
That said, troubleshooting Kubernetes applications might seem daunting
with all of the layers of abstraction that make up one deployment.

At its core, Kubernetes application consists of _[Pods][k8s-debug-pods]_
and [Services][k8s-services].
Each Pod represents one or more [containers][k8s-container]
for application components that share storage and network resources.
Each Service abstracts how Pods communicate with each other
and the Internet.
Before Kubernetes, you had to spin up multiple servers that each ran
one part of an Internet application.
With Kubernetes, you can deploy one Pod that serves that whole application
through one Service.

This guide focuses on Pods.
The Debugging Services page covers the troubleshooting of Services.

:scroll:
You can also debug the components that serve Kubernetes clusters.
To learn how to debug a cluster,
consult [Troubleshooting Clusters][k8s-tb-clusters] in the Kubernetes documentation.

## How Do You Debug an Application?

Debugging a Kubernetes application involves three activities:

1. Find which Pod doesn't work
2. Find out why that Pod doesn't work
   - If needed, find when it stopped working
3. Fix the Pod

Like most Kubernetes management tasks,
you debug your application using the [`kubectl`][] command line (CLI) tool.

## Find the Misbehaving Pod

The complexity of a failing Kubernetes application might make 
troubleshooting seem difficult. To reduce this complexity,
you can find which Pods aren't working.
To find which Pods don't work, run the `kubectl get pods` command.
If your Pods uses [namespaces][k8s-namespaces],
add the `--namespace` option to this command.

### Example of kubectl get Pods

```terminal
$ kubectl get Pods [--namespace=<namespace-name>|--all-namespaces]

NAME                                READY     STATUS    RESTARTS   AGE
apache-deployment-1006230814-6winp   1/1       Running   0         15m
apache-deployment-1006230814-fmgu3   1/1       Running   0         15m
apache-deployment-1370807587-6ekbw   1/1       Running   0          1m
apache-deployment-1370807587-fg172   0/1       Pending   0          1m
apache-deployment-1370807587-fz9sd   0/1       Pending   0          1m

```

If the status of any Pod displays a [phase][k8s-phase] other than `Running`,
it might have a problem.
The last two results in the preceeding response might have an issue.
If no Pod displays a phase other than `Running`,

To learn more about how to debug a Pod,
consult [Debugging Pods][k8s-debug-Pods] in the Kubernetes documentation.

## Find Out Why the Pod Doesn't Work

With the misbehaving Pod or Pods identified,
you can check each one for possible issues.
To check a Pod, ask for a description of that Pod
with the `kubectl describe Pod` command.
This command returns the details of the Pod and its related resources
like containers and storage volumes. It also returns the latest events
that occurred with those resources.

### Example of kubectl describe Pod

```terminal
$ kubectl describe Pod apache-deployment-1370807587-fz9sd

  Name:     apache-deployment-1370807587-fz9sd
  Namespace:    default

  ...

  fit failure on node (kubernetes-node-8tb9): Node didn't have enough resource: CPU, requested: 1000, used: 1420, capacity: 2000
  fit failure on node (kubernetes-node-xfl4): Node didn't have enough resource: CPU, requested: 1000, used: 1100, capacity: 2000
```

_This Pod couldn't start._
Neither of the two hosting nodes could provision the required compute resources.
Both nodes can provision up to two cores, or 2,000 [millicores][millicore], each.
The first node was using 1,420 millicores and the second was using 1,100.
Neither node had enough spare millicores to start the Apache Pod.

## Check When the Misbehavior Started

Not all issues might reveal themselves with this little effort.
These logs return error messages and stack traces to help you identify
a configuration error, missing dependency, or a runtime exception.
To get the logs of a specific Pod,
run the `kubectl logs` command appending the `Pod` name.

### Example of kubectl logs

```terminal
$ kubectl logs apache-deployment-1370807587-fz9sd --all-containers=true
```

To learn more about Kubernetes logs,
consult [kubectl logs][k8s-logs] in the Kubernetes documentation.

## Fix the Misbehaving Pod

How you fix your Pod depends on the nature of the issues you find.

To find potential solutions to Kubernetes application errors,
consult [Troubleshoot applications][k8s-tb-apps] in the
Kubernetes documentation.

## To Learn More

To learn more about application debugging,
consult these guides in the Kubernetes documentation

### Debugging Guides

* [Troubleshooting Applications][k8s-tb-apps]
* [Debug Pods][k8s-debug-pods]

### `kubectl` commands reference

* [`kubectl`][]
* [`kubectl debug`][]
* [`kubectl logs`][k8s-logs]

[`kubectl`]: https://kubernetes.io/docs/reference/kubectl/

[k8s-tb-clusters]: https://kubernetes.io/docs/tasks/debug/debug-cluster/
[k8s-tb-apps]: https://kubernetes.io/docs/tasks/debug/debug-application/

[`kubectl debug`]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/

[k8s-namespaces]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

[k8s-debug-pods]: https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/

[millicore]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu

[k8s-container]: https://kubernetes.io/docs/concepts/containers/

[k8s-phase]: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase

[k8s-logs]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/

[k8s-debug-pods]: https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/#debugging-pods
