# Debug Your Kubernetes Deployment

In Kubernetes, you can debug either a deployment or a cluster.
A Kubernetes deployment consists of the parts of a Kubernetes-hosted app.
A Kubernetes cluster consists of the group of Kubernetes workers that run your app.
This guide covers debugging a Kubernetes deployment.

Like most Kubernetes management tasks,
debugging uses the [`kubectl`][] command line (CLI) tool.

:scroll:
To learn how to debug your cluster,
refer to the [Kubernetes Troubleshooting Clusters guide][k8s-tb-clusters].

## How Do You Debug a Deployment?

A Kubernetes deployment consists of _[pods][k8s-debug-pods]_.
Each pod represents a group of containers deployed together that run one application.
One Kubernetes deployment might run more than one pod.
You can use the methods in this guide on any single pod. 

To debug a Kubernetes deployment involves three activities:

1. Find the misbehaving pod
2. Find why the cause of the pod's behavior
   - If needed, find when the behavior started
3. Fix the pod's behavior

## Find the Misbehaving Pod

To find any misbehaving pods, run the `kubectl get pods` command.
If your pods uses [namespaces][k8s-namespaces],
add the `--namespace` option to this command.

```terminal
$ kubectl get pods [--namespace=<namespace-name>|--all-namespaces]

NAME                                READY     STATUS    RESTARTS   AGE
apache-deployment-1006230814-6winp   1/1       Running   0         15m
apache-deployment-1006230814-fmgu3   1/1       Running   0         15m
apache-deployment-1370807587-6ekbw   1/1       Running   0          1m
apache-deployment-1370807587-fg172   0/1       Pending   0          1m
apache-deployment-1370807587-fz9sd   0/1       Pending   0          1m

```

If any pod returns a status other than `Running`,
it might have a problem.
The last two results in the preceeding response might have an issue.

## Check Pod for Cause of Misbehavior

To reveal the health of a pod, ask for a description of that pod
with the `kubectl describe pod` command.
This command returns the details of the pod and its related resources
like containers and storage volumes. It also returns the latest events
that occurred with those resources.

```terminal
$ kubectl describe pod apache-deployment-1370807587-fz9sd

  Name:     apache-deployment-1370807587-fz9sd
  Namespace:    default

  ...

  fit failure on node (kubernetes-node-8tb9): Node didn't have enough resource: CPU, requested: 1000, used: 1420, capacity: 2000
  fit failure on node (kubernetes-node-xfl4): Node didn't have enough resource: CPU, requested: 1000, used: 1100, capacity: 2000
```

This pod couldn't start. Neither hosting node could provision the required compute resources.
Both nodes can provision up to two cores, or 2,000 [millicores][millicore], each.
The first node was using 1,420 millicores and the second was using 1,100.
Neither node had enough spare millicores to start the Apache pod.

## Check for When the Misbehavior Started

To find when an issue might have arisen, review the logs.
To get the logs of a specific pod,
run the `kubectl logs` command with the `pod` name.

```terminal
kubectl logs apache-deployment-1370807587-fz9sd --all-containers=true
```

To run commands on a running Kubernetes pod, run the `kubectl exec` command.

When debugging, start with `kubectl get pods`, `kubectl logs`, and `kubectl exec` to explore the inside of the container and review other log files or configurations.

:note:

The command `kubectl debug` is another option to consider when debugging a container.

This command can be used to create a clone of a pod that doesn't terminate if an error is experienced inside the container.

:::

## To Learn More

[How to debug Kubernetes Pods]: https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/

[`kubectl`]: https://kubernetes.io/docs/reference/kubectl/

[k8s-tb-clusters]: https://kubernetes.io/docs/tasks/debug/debug-cluster/

[`kubectl debug`]: https://kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/

[k8s-namespaces]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/

[k8s-debug-pods]: https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/

[millicore]: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-cpu