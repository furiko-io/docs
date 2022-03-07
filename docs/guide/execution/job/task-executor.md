# Task Executors

Furiko supports an extensible task executor interface, which allows tasks to be created, managed and reconciled in the same way regardless of the actual backing object.

A Job creates one or more tasks during its lifecycle. Each task corresponds to an attempt to execute the Job. For example, for a Job that has a `maxRetryAttempts` of `2`, it may create up to 3 tasks if they had all consecutively failed.

Currently, all tasks are created as Pods, but more task executors could be added in the future to support different types of tasks.

## Executors

### PodTaskExecutor

The PodTaskExecutor is the default and currently only task executor available in Furiko. Each task will be created as a Pod. On most Kubernetes clusters, this translates to a single container that runs to completion.

Additional behavior is as follows:

1. A task's kill timestamp will translate into the Pod's `activeDeadlineSeconds`.
2. If the Pod is not yet scheduled by kube-scheduler, it will be deleted instead of using active deadline.

#### Virtual Pods

Using tools such as [Virtual Kubelet](https://github.com/virtual-kubelet/virtual-kubelet), it may be possible for the Pods to manifest as objects other than a CRI container in Kubernetes.

If Virtual Kubelet is provisioned in the cluster, some possible use cases include:

1. Running Pods on serverless compute platforms like [AWS Fargate](https://github.com/virtual-kubelet/virtual-kubelet#aws-fargate-provider)
2. Extending to multi-cluster with [Admiralty](https://github.com/virtual-kubelet/virtual-kubelet#admiralty-multi-cluster-scheduler)

The usage and support of Virtual Kubelet is outside the scope of the Furiko project.

### Potential Executors

The following is a list of potential executors that could be supported in the future:

- Serverless function invocations (e.g. Knative)
- SSH/bare metal executors
