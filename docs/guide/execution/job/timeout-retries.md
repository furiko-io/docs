# Timeouts and Retries

Furiko provides several mechanisms to impose timeouts and retrying of failed Jobs.

## Task-level Timeouts

The following timeouts apply to **individual tasks** in a Job.

### `pendingTimeoutSeconds`

Specifies the maximum duration that a single task can be pending for. This includes the time taken for scheduling, image pull and container creation.

If not specified, defaults to the global controller `defaultTaskPendingTimeoutSeconds` value.

<!-- prettier-ignore -->
!!! info
    If the Pod cannot pull the container image, it will remain in ImagePullBackOff indefinitely. A pending timeout helps to stop these Jobs eventually.

### `runningTimeoutSeconds`

Specifies the maximum duration that a single task can be running for. The time starts once the task starts running.

If not specified, defaults to no timeout.

<!-- prettier-ignore -->
!!! tip
    It is recommended to use [`timeout(1)`](https://man7.org/linux/man-pages/man1/timeout.1.html) available on Unix systems, which provides several additional mechanisms to control the exit code and signals being sent on timeout.

    Another way is to also use timeouts in your application directly, and read the value from a [Job Option](../jobconfig/job-options.md).

<!-- prettier-ignore -->
!!! todo
    This timeout is not yet implemented.

### `template.spec.activeDeadlineSeconds`

You can also set the task's `activeDeadlineSeconds` directly, which is the duration relative to the Pod's `startTime` before Kubelet will actively try to kill associated containers.

## Job-level Timeouts

The following timeouts apply to **the entire Job** across all tasks.

### `jobTimeoutSeconds`

Specifies a global timeout for the entire Job across all tasks. The time starts when the Job is started, and is inclusive of the retry delay and time spent waiting for the tasks to start running.

<!-- prettier-ignore -->
!!! todo
    This timeout is not yet implemented.

## Retries

Furiko retries failed Jobs by creating a new Task.

If a Node is misconfigured or has some host-level issue, using `restartPolicy: OnFailure` to recreate the container would not be sufficient to avoid spurious Job failures which may only be resolved by running the Task on a different Node. As such, Furiko recommends using Job-level retries, which recreates an entirely new Pod.

### `maxRetryAttempts`

Specifies the maximum number of retries to attempt, **after** the first failed task. That is, the `maxRetryAttempts` does **NOT** include the first task.

If not specified, it means that the Job will not retry.

### `retryDelaySeconds`

Specifies the duration between the last failed task and creation of the next task.

If not specified, it means there is no delay.

### `restartPolicy`

If the Job uses both `restartPolicy: OnFailure` in conjunction with Furiko Job-level tries, Jobs may take a longer time before finally terminating in failure.

If a Job is in a `CrashLoopBackOff`, it will be deemed to be still "pending", and if it remains/transitions to this state even after its [pending timeout](#pendingtimeoutseconds), it will be killed. The next Task will be only created after `retryDelaySeconds`, which results in the creation of a brand-new Pod.

This means that the total time taken before terminating in failure would be roughly around `pendingTimeoutSeconds * (maxRetryAttempts + 1) + retryDelaySeconds * maxRetryAttempts`, rather than simply the sum of all tasks' running durations.
