# Force Deletion

This page describes when and how **force deletion** is used in the JobController.

## Motivation

When [killing a Job](./killing-jobs.md), the controller first attempts to use active deadline to gracefully terminate the task, followed by using normal API deletion.

However, if the node is not responsive (e.g. kubelet is not running, unreachable via network, etc.), such deletion may not be effective, since Kubernetes ensures that the Pod will not be deleted until it has confirmed that the container is fully removed and cleaned up on the node itself.

If the JobConfig uses a concurrency policy like `Forbid`, this may **indefinitely block future progress** of the JobConfig from creating and starting new Jobs. As such, Furiko supports "force deletion" as a last resort, which forcefully deletes the Pod, allowing the Job to be terminated in order for the JobConfig to schedule new Jobs.

## Deletion Process

The "force deletion" process is analogous to simply running:

```
kubectl delete <pod name> --grace-period=0 --force
```

<!-- prettier-ignore -->
!!! warning
    The [official Kubernetes official documentation](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete) mentions the following warning about force deletion (emphasis added):

    > IMPORTANT: Force deleting pods does not wait for confirmation that the pod's processes have been terminated, which **can leave those processes running until the node detects the deletion and completes graceful deletion**. If your processes use shared storage or talk to a remote API and depend on the name of the pod to identify themselves, force deleting those pods **may result in multiple processes running on different machines using the same identification which may lead to data corruption or inconsistency**. Only force delete pods when you are sure the pod is terminated, or if your application can tolerate multiple copies of the same pod running at once. Also, if you force delete pods the scheduler may place new pods on those nodes before the node has released those resources and causing those pods to be evicted immediately.

Typically when running cron jobs force deletion is safe most of the time, and in most cases would prefer availability of scheduling over strict guarantees. However, jobs which are more crucial may opt to disable this functionality, and instead trigger notifications to investigate node-level issues.

## Configuration

### Task: `forbidForceDeletion`

You can prevent force deletion by specifying `forbidForceDeletion: true` in the JobTaskSpec, as follows:

```yaml
apiVersion: execution.furiko.io/v1alpha1
kind: Job
spec:
  template:
    task:
      # The normal task template
      template: ...

      # Forbids force deletion of the task.
      forbidForceDeletion: true
```

In such a scenario, the controller will simply just wait until the Pod transitions into a terminated state before marking the Job as terminated as well.

### Global: `forceDeleteKillingTasksTimeoutSeconds`

Specifies the default duration that the controller should wait after the `deletionTimestamp` of a task before forcefully deleting the task.

For more information, see the [Execution Dynamic Config](../../../reference/configuration/execution/dynamic-config.md#forcedeletekillingtaskstimeoutseconds) documentation.
