# Debug Your Running Kubernetes Deployment

You can debug a Kubernetes deployment, the components of a Kubernetes-hosted app,
or a Kubernetes cluster, the group of Kubernetes workers that run your app.
This guide covers debugging a Kubernetes deployment.

As with other actions performed on a Kubernetes clusters,
use the [`kubectl`][] command line (CLI) tool for debugging tasks.

::: note
To learn how to debug your Kubernetes cluster,
refer to the [Kubernetes Troubleshooting Clusters guide][k8s-tb-clusters].
::: 

A Kubernetes deployment consists of _pods_.
Each pod represents a group of containers deployed together that run one application.
A single Kubernetes deployment might run more than one pod for fault tolerance
and scale. You can use the methods in this guide on any pod.

Debugging a Kubernetes deployment involves finding a pod with issues,
identifying those issues, and then resolving those issues.

## Find Which Pods Have Potential Issues

To find which pod doesn't behave as expected,
run the `kubectl get pods` command.
If you configured your cluster to use [namespaces][k8s-namespaces],
include that namespace as an option to this command.

```terminal
$ kubectl get pods [--namespace=<namespace-name>|--all-namespaces]

NAME                                READY     STATUS    RESTARTS   AGE
apache-deployment-1006230814-6winp   1/1       Running   0         15m
apache-deployment-1006230814-fmgu3   1/1       Running   0         15m
apache-deployment-1370807587-6ekbw   1/1       Running   0          1m
apache-deployment-1370807587-fg172   0/1       Pending   0          1m
apache-deployment-1370807587-fz9sd   0/1       Pending   0          1m
```

If any pod returns a status other than `Running`, it might have a problem.

## Check Pod for Issues

To find out why one of your pods isn't running,
you need to get more details about that pod.
To get those details, run the `kubectl describe pod` command.

```terminal
$ kubectl describe pod apache-deployment-1370807587-fz9sd

  Name:		apache-deployment-1370807587-fz9sd
  Namespace:	default
...
  fit failure on node (kubernetes-node-8tb9): Node didn't have enough resource: CPU, requested: 1000, used: 1420, capacity: 2000
  fit failure on node (kubernetes-node-xfl4): Node didn't have enough resource: CPU, requested: 1000, used: 1100, capacity: 2000
```

This pod couldn't start because the node on which it runs couldn't provide
the required resources. Both nodes have two cores or 2,000 millicores each.
The first node had 1,420 millicores in use and the second had 1,100.
Both nodes didn't have enough millicores available to start this pod.

## Check for When Pod First Had an Issue

To review logs when debugging a container, run the `kubectl logs` command.
This retrieves the logs of a specific pod.

```terminal
kubectl logs ${POD_NAME} ${CONTAINER_NAME}
```

To run commands on a running Kubernetes pod, run the `kubectl exec` command.


When debugging, start with `kubectl get pods`, `kubectl logs`, and `kubectl exec` to explore the inside of the container and review other log files or configurations.

::: note
The command `kubectl debug` is another option to consider when debugging a container.
This command can be used to create a clone of a pod that doesn't terminate if an error is experienced inside the container.
:::

# References

- https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-strong-getting-started-strong-

- [What is Kubernetes](https://kubernetes.io/docs/concepts/overview/)

[`kubectl`]: https://kubernetes.io/docs/reference/kubectl/
[k8s-tb-clusters]: https://kubernetes.io/docs/tasks/debug/debug-cluster/
[`kubectl debug`] https://kubernetes.io/docs/reference/kubectl/generated/kubectl_debug/
[k8s-namespaces]: https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/
[k8s-debug-pods]: https://kubernetes.io/docs/tasks/debug/debug-application/debug-pods/
