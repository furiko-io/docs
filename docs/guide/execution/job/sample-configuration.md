# Sample Configuration

This page contains sample configurations for creating a Job.

## From JobConfig

Most of the time, you should be creating Jobs from a JobConfig as follows, as described in [Adhoc Execution](./adhoc-execution.md).

```yaml
apiVersion: execution.furiko.io/v1alpha1
kind: Job
metadata:
  generateName: jobconfig-sample-
spec:
  # Define the JobConfig (in the same namespace) to use as the template for the Job.
  configName: jobconfig-sample

  # YAML or JSON encoded string of option values to pass to the JobConfig.
  # Will be evaluated and added to substitutions.
  optionValues: |-
    username: Example User
    image-tag: 3.15

  # Optionally specify conditions for when the Job can start.
  startPolicy:
    # Defines the behavior for multiple concurrent Jobs.
    # If not specified, will default to the JobConfig's concurrency.policy.
    concurrencyPolicy: Enqueue

    # Start only after this time.
    startAfter: 2022-03-06T00:27:00+08:00
```

## Standalone Job

You can also create a Job independent of a JobConfig, as described [here](./adhoc-execution.md#independent-jobs). You must specify the `template` yourself.

```yaml
apiVersion: execution.furiko.io/v1alpha1
kind: Job
metadata:
  generateName: jobconfig-sample-
spec:
  # Map of key-value pairs to substitute into the task template.
  substitutions:
    option.username: Example User

  # Describes how to create the Job.
  template:
    # Task configuration for the Job.
    task:
      # Optional duration in seconds for how long the task should be pending
      # until it gets killed.
      pendingTimeoutSeconds: 1800

      # Specify the template to create the task.
      template:
        spec:
          containers:
            - args:
                - echo
                - "Hello world, ${option.username}!"
              env:
                - name: JOBCONFIG_NAME
                  value: jobconfig-sample
                - name: JOB_NAME
                  value: ${job.name}
                - name: TASK_NAME
                  value: ${task.name}
              image: alpine
              name: job-container
              resources:
                limits:
                  cpu: 100m
                  memory: 64Mi
          restartPolicy: Never

    # Optional duration that the Job should live after it is finished.
    ttlSecondsAfterFinished: 3600
```
