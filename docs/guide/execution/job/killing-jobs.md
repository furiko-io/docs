# Killing Jobs

A Job can be killed after it is started, either manually or through a [timeout](./timeout-retries.md).

## Kill Timestamp

Each Job has a `killTimestamp` in its spec, which specifies the timestamp before the JobControll tries to kill all active tasks.

To kill a Job explicitly, set `killTimestamp` to the current time or earlier:

```yaml
apiVersion: execution.furiko.io/v1alpha1
kind: Job
spec:
  killTimestamp: "2021-01-01T00:00:00Z"
```

You can also set a time in the future to kill the Job by setting a future time. The JobController will only start killing active tasks once killTimestamp is passed.

## Timeouts

When a Job reaches one of its various [timeout](./timeout-retries.md), it will also be killed in the same way as if its `killTimestamp` was set.

## Termination Process

The JobController uses a few methods to kill Jobs. In order of preference, the JobController uses **active deadline**, **API deletion** and finally **force deletion** as a last resort.

### Active Deadline

When the JobController wishes to kill a Job, the first thing it will do is to mark all of its active tasks as exceeding its active deadline, which uses the Pod's `spec.activeDeadlineSeconds` to achieve this.

The reason for first doing so is to keep the Pod around as much as possible, which is useful for several reasons. For example, viewing the logs or raw `PodStatus` of the task would be possible via `kubectl` or from `kube-apiserver` directly.

<!-- prettier-ignore -->
!!! info
    Kubernetes does not offer a native way to "kill" Pods without deleting them, and thus Furiko uses active deadline as a means to approximate this.

    See [this Kubernetes GitHub issue](https://github.com/kubernetes/kubernetes/issues/14602) for a related discussion on this issue.

### API Deletion

In some cases, using `activeDeadlineSeconds` may not be sufficient to terminate the task, and the JobController would then fall back to using normal API deletion to terminate it instead.

The JobController switches to this termination mode once `deleteKillingTasksTimeoutSeconds` is passed since its first attempt to kill the task. For more information on this timeout configuration value, see the [Execution Dynamic Config](../../../reference/configuration/execution/dynamic-config.md#deletekillingtaskstimeoutseconds) documentation.

Additionally, Pods in certain states may not respond to setting the active deadline (e.g. the Pod is not yet scheduled by kube-scheduler). In such cases, the controller would fall back to deletion to terminate the Job timely instead.

### Force Deletion

When a Job's task is running on a node that is no longer available, above methods to kill the Job will have no effect until the node comes back up. As a last resort, the controller may opt to use **force deletion** to kill the task instead, which does not block any forward progress of the JobConfig.

For more details and implications of using such a termination method, see the [Force Deletion](./force-deletion.md) documentation.
