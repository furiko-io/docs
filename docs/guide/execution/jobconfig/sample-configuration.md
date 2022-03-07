# Sample Configuration

The following is a full sample configuration.

```yaml
apiVersion: execution.furiko.io/v1alpha1
kind: JobConfig
metadata:
  name: jobconfig-sample
spec:
  # Concurrency configuration.
  concurrency:
    policy: Forbid

  # Schedule configuration.
  schedule:
    cron:
      expression: "0 */15 * * * * *"
      timezone: "Asia/Singapore"
    disabled: false

  # Job options.
  option:
    options:
      - type: String
        name: username
        label: Username
        string:
          default: Example User
          trimSpaces: true

  # Template for the Job to be created.
  template:
    # Any labels and annotations will be automatically added to downstream Jobs.
    metadata:
      annotations:
        annotations.furiko.io/job-group: "cool-jobs"
    spec:
      task:
        # Optional duration in seconds for how long the task should be pending
        # until it gets killed.
        pendingTimeoutSeconds: 1800

        # Define how the task should be created. This is just a PodTemplateSpec.
        template:
          spec:
            containers:
              # Notice how we can use context variables and job options inside
              # the PodSpec freely to be substituted at runtime.
              - name: job-container
                args:
                  - echo
                  - "Hello world, ${option.username}!"
                env:
                  - name: JOBCONFIG_NAME
                    value: "${jobconfig.name}"
                  - name: JOB_NAME
                    value: "${job.name}"
                image: "alpine"
                resources:
                  limits:
                    cpu: 100m
                    memory: 64Mi
```
