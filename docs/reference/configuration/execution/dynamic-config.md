# Execution Dynamic Config

This page documents the [dynamic configuration](../overview.md#dynamic-configuration) for all execution components.

## Configuration Options

Each of the following sections corresponds to a single key in the dynamic configuration ConfigMap.

### `job-controller`

Defines configuration for Jobs and the JobController.

#### `defaultTTLSecondsAfterFinished`

Defines the default value of `ttlSecondsAfterFinished` for a Job if it is not defined. For more information, see [Garbage Collection](../../../guide/execution/job/garbage-collection.md).

To avoid issues with huge amounts of finished Jobs building up, a non-zero TTL is enforced.

Defaults to 3600 (1 hour).

#### `defaultPendingTimeoutSeconds`

Defines the default value of `pendingTimeoutSeconds` for a Job if it is not defined. For more information, see [Pending Timeouts](../../../guide/execution/job/timeout-retries.md#pendingtimeoutseconds).

Defaults to 900 (15 minutes).

#### `deleteKillingTasksTimeoutSeconds`

Defines the duration that the controller should wait to kill tasks via API deletion instead of using the active deadline, if previous efforts were ineffective.

Defaults to 180 (3 minutes).

#### `forceDeleteKillingTasksTimeoutSeconds`

Defines the duration that the controller should wait to kill tasks via force deletion after first using normal API deletion, if previous efforts were ineffective. This timeout is computed from the deletionTimestamp of the object, which also includes `deletionGracePeriodSeconds`. For more information, see [Force Deletion](../../../guide/execution/job/force-deletion.md).

Defaults to 120 (2 minutes).

### `cron-controller`

Defines configuration for CronController, as well as how to [parse cron expressions](../../../guide/execution/jobconfig/cron-syntax.md) within the cluster.

#### `cronFormat`

Specifies the format used to parse cron expressions. The only difference is in the [Day of Week digit parsing](../../../guide/execution/jobconfig/cron-syntax.md#day-of-week).

Only the following values are supported:

- `standard` (default): Standard cron format.
- `quartz`: [Quartz](http://www.quartz-scheduler.org/documentation/quartz-2.3.0/tutorials/crontrigger.html) cron format.

#### `cronHashNames`

Specifies if cron expressions should be hashed using the JobConfig's name. Enabling this option would [allow `H` tokens to be used](../../../guide/execution/jobconfig/cron-syntax.md#hash-based-load-balancing). If disabled, any JobConfigs that use the `H` syntax will throw a parse error.

Defaults to `true`.

#### `cronHashSecondsByDefault`

Specifies if the seconds field of a cron expression should be a `H` or `0` by default if omitted. If enabled, the default seconds field will be `H`, otherwise it will be `0`.

For JobConfigs which use a short cron expression format (i.e. 5 or 6 tokens long), the seconds field is omitted and is typically assumed to be `0` (e.g. `5 10 * * *` means to run at 10:05:00 every day). Enabling this option will allow JobConfigs to be scheduled across the minute, improving load balancing.

Users can still choose to start at 0 seconds by explicitly specifying a long cron expression format with `0` in the seconds field. In the above example, this would be `0 5 10 * * * *`.

Defaults to `false`.

#### `cronHashFields`

Specifies if the "type of field" should be hashed along with the JobConfig's name.

For example, `H H * * * * *` will always hash the seconds and minutes to the same value, for example 00:37:37, 01:37:37, etc. Enabling this option will append additional keys to be hashed to introduce additional non-determinism.

Defaults to `true`.

## Sample Configuration

The following sample shows how to configure the full dynamic configuration ConfigMap of the execution component, as well as the default configuration values for each field.

<!-- prettier-ignore -->
??? example "Full Sample Configuration"

    ```yaml
    # Here we define the dynamic config for execution-controller.
    # We can tune several knobs in execution-controller without requiring a restart.
    # Each file in the ConfigMap defines a group of configuration knobs available for
    # the user to tweak as desired. As a start, we have populated a set of sane default
    # values for you.
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: execution-dynamic-config
      namespace: furiko-system
    data:
      job-controller: |
        # defaultTTLSecondsAfterFinished is the default time-to-live (TTL) for a Job
        # after it has finished. Lower this value to reduce the strain on the
        # cluster/kubelet. This field can be set separately on each Job as well.
        defaultTTLSecondsAfterFinished: 3600

        # defaultPendingTimeoutSeconds is default timeout to use if job does not
        # specify the pending timeout. By default, this is a non-zero value to prevent
        # permanently stuck jobs. To disable default pending timeout, set this to 0 or
        # a negative number.
        defaultPendingTimeoutSeconds: 900

        # deleteKillingTasksTimeoutSeconds is the duration we delete the task to kill
        # it instead of using active deadline, if previous efforts were ineffective.
        deleteKillingTasksTimeoutSeconds: 180

        # forceDeleteKillingTasksTimeoutSeconds is the duration before we use force
        # deletion instead of normal deletion. This timeout is computed from the
        # deletionTimestamp of the object, which may also include an additional delay
        # of deletionGracePeriodSeconds.
        forceDeleteKillingTasksTimeoutSeconds: 120

      cron-controller: |
        # cronFormat specifies the format used to parse cron expressions. Select
        # between "standard" (default) or "quartz".
        cronFormat: "standard"

        # cronHashNames specifies if cron expressions should be hashed using the
        # JobConfig's name.
        #
        # This enables "hash cron expressions", which looks like `0 H * * *`. This
        # particular example means to run once a day on the 0th minute of some hour,
        # which will be determined by hashing the JobConfig's name. By enabling this
        # option, JobConfigs that use such cron schedules will be load balanced across
        # the cluster.
        #
        # If disabled, any JobConfigs that use the `H` syntax will throw a parse error.
        cronHashNames: true

        # cronHashSecondsByDefault specifies if the seconds field of a cron expression
        # should be a `H` or `0` by default. If enabled, it will be `H`, otherwise it
        # will default to `0`.
        #
        # For JobConfigs which use a short cron expression format (i.e. 5 or 6 tokens
        # long), the seconds field is omitted and is typically assumed to be `0` (e.g.
        # `5 10 * * *` means to run at 10:05:00 every day). Enabling this option will
        # allow JobConfigs to be scheduled across the minute, improving load balancing.
        #
        # Users can still choose to start at 0 seconds by explicitly specifying a long
        # cron expression format with `0` in the seconds field. In the above example,
        # this would be `0 5 10 * * * *`.
        cronHashSecondsByDefault: false

        # cronHashFields specifies if the fields should be hashed along with the
        # JobConfig's name.
        #
        # For example, `H H * * * * *` will always hash the seconds and minutes to the
        # same value, for example 00:37:37, 01:37:37, etc. Enabling this option will
        # append additional keys to be hashed to introduce additional non-determinism.
        cronHashFields: true

        # defaultTimezone defines a default timezone to use for JobConfigs that do not
        # specify a timezone. If left empty, UTC will be used as the default timezone.
        defaultTimezone: "UTC"

        # maxMissedSchedules defines a maximum number of jobs that the controller
        # should back-schedule, or attempt to create after coming back up from
        # downtime. Having a sane value here would prevent a thundering herd of jobs
        # being scheduled that would exhaust resources in the cluster.
        maxMissedSchedules: 5

        # maxDowntimeThresholdSeconds defines the maximum downtime that the controller
        # can tolerate. If the controller was intentionally shut down for an extended
        # period of time, we should not attempt to back-schedule jobs once it was
        # started.
        maxDowntimeThresholdSeconds: 300
    ```
