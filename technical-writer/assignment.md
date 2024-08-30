# Debug Your Kubernetes Application

Kubernetes can simplify deploying applications.
Before Kubernetes, you had to spin up multiple servers that each ran
one part of an Internet application.
With Kubernetes, you can deploy one item that serves that whole application.
Troubleshooting Kubernetes applications might seem daunting as
each application has layers of abstraction.

At its core, Kubernetes application consists of _[Pods][k8s-debug-pods]_
and [Services][k8s-services].
Each Pod represents one or more application [containers][k8s-container]
that share storage and network resources.
Each Service abstracts how Pods communicate with each other
and the Internet.

This guide focuses on Pods.
The Debugging Services page covers fixing Services.

:blue_book: You can also debug the components that serve Kubernetes clusters.
To learn how to debug a cluster,
consult [Troubleshooting Clusters][k8s-tb-clusters].

## How Do You Debug an Application?

Debugging a Kubernetes application involves three activities:

1. Find which Pod doesn't work
2. Find out why that Pod doesn't work
   - If needed, find when it stopped working
3. Fix the Pod

Like most Kubernetes tasks,
you use the [`kubectl`][] command line (CLI) tool to debug your application.

## Find the Misbehaving Pod

The complexity of a Kubernetes application makes troubleshooting difficult.
To reduce this complexity,
first run the `kubectl get pods` command to find which Pods aren't working.
If your Pods use [namespaces][k8s-namespaces],
add the `--namespace` option to this command.

### Example of `kubectl get pods`

```terminal
$ kubectl get pods [--namespace=<namespace-name>|--all-namespaces]

NAME                                READY     STATUS    RESTARTS   AGE
apache-deployment-1006230814-6winp   1/1       Running   0         15m
apache-deployment-1006230814-fmgu3   1/1       Running   0         15m
apache-deployment-1370807587-6ekbw   1/1       Running   0          1m
apache-deployment-1370807587-fg172   0/1       Pending   0          1m
apache-deployment-1370807587-fz9sd   0/1       Pending   0          1m

```

If the status of any Pod displays a [phase][k8s-phase] other than `Running`,
it might have a problem.
The last two results in the preceding response might have an issue.
If no Pod displays a phase other than Running, check each Pod for issues.

:blue_book: To learn more about how to debug a Pod,
consult [Debugging Pods][k8s-debug-pods].

## Find Out Why the Pod Doesn't Work

With the misbehaving Pods identified,
check each one for issues.
To check a Pod's status, run the `kubectl describe pod` command.
This command returns the details of the Pod and its related resources.
It also returns the latest events that occurred with those resources.

### Example of `kubectl describe pod`

```terminal
$ kubectl describe pod apache-deployment-1370807587-fz9sd

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
Check the logs for error messages and stack traces.
These log entries should identify issues like configuration errors
or runtime exceptions.
To get the logs for a specific Pod,
run the `kubectl logs <pod-name>` command.

### Example of `kubectl logs`

```terminal
$ kubectl logs apache-deployment-1370807587-fz9sd --all-containers=true

[08/30/2024:10:47:28] [info] [pid 1] AH00558: Apache/2.4.52 (Unix) configured -- resuming normal operations
[08/30/2024:10:47:28] [info] [pid 1] AH00557: Apache/2.4.52 (Unix) configured
[08/30/2024:10:47:28] [info] [pid 1] AH00559: Apache/2.4.52 (Unix) configured: listening on port 80
[08/30/2024:10:47:28] [info] [pid 1] AH00559: Apache/2.4.52 (Unix) configured: listening on port 443
[08/30/2024:10:47:28] [info] [pid 1] AH00559: Apache/2.4.52 (Unix) configured: listening on port 8080
[08/30/2024:10:47:28] [info] [pid 1] AH00559: Apache/2.4.52 (Unix) configured: listening on port 8443
[08/30/2024:10:47:28] [info] [pid 1] AH00559: Apache/2.4.52 (Unix) configured: listening on port 8000
```

:blue_book: To learn more about Kubernetes logs,
consult [kubectl logs][k8s-logs].

## Fix the Misbehaving Pod

How you fix your Pod depends on the nature of the issues you find.

:blue_book: To find solutions for Kubernetes application errors,
consult [Troubleshoot applications][k8s-tb-apps].

## To Learn More

To learn more about application debugging,
consult these guides in the Kubernetes documentation.

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

[k8s-services]: https://kubernetes.io/docs/concepts/services-networking/service/

[millicore]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu

[k8s-container]: https://kubernetes.io/docs/concepts/containers/

[k8s-phase]: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase

[k8s-logs]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_logs/

[k8s-debug-pods]: https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/#debugging-pods
