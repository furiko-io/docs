# Features

Furiko specializes in ensuring **correctness and scale**, for time-based scheduling of scheduled and adhoc jobs in an enterprise setting. The following is a non-exhaustive list of features and enhancements offered by Furiko.

## Feature List

### Timezone-aware Cron Scheduling

Furiko allows users to specify a [cron schedule](./execution/jobconfig/scheduling.md#cronexpression) to automatically schedule jobs, supporting up to a per-second granularity.

Furiko also offers native support for scheduling jobs on a [cron schedule with timezones](./execution/jobconfig/scheduling.md#crontimezone). This allows users to specify their cron schedule in a timezone that is familiar to them, or one that is standardized in their application. If not specified, Furiko also supports specifying a cluster-wide default timezone for all cron schedules.

### Cluster-wide Load Balancing

Furiko offers a unique, [extended cron syntax](./execution/jobconfig/cron-syntax.md#hash-based-load-balancing) that drastically improves the performance of running distributed cron at scale.

Furiko introduces a new syntax like `H/5 * * * *`. The `H` token means to deterministically choose a "random" number in the given range, avoiding thundering herd effects when thousands of other jobs also start at the exact same time. This allows users to still be able to run their job at a fixed interval of 5 minutes, but evenly spread across the time range.

### Stronger, More Flexible Concurrency Handling

Furiko introduces stronger and more flexible [handling of multiple concurrent jobs](./execution/jobconfig/concurrency.md).

Using the `Forbid` concurrency policy, Furiko takes a very [strict approach](./development/architecture/execution-controller.md#jobqueuecontroller) to ensuring that multiple jobs will never be started at the same time, free of race conditions. Another example is `Enqueue`, which allows multiple jobs to be queued in a FIFO manner for [executing at a later time](./execution/job/adhoc-execution.md#scheduling-adhoc-future-executions).

### Exactly-once Scheduling

Furiko provides strong guarantees to provide **exactly-once scheduling** of jobs even in the face of failure, by preventing both double runs and missed runs in most cases.

Furiko prevents double scheduled runs by using [deterministic name formats](./development/architecture/execution-controller.md#jobcontroller). It also prevents missed runs using [back-scheduling](./execution/jobconfig/scheduling/#back-scheduling) to tolerate short downtime, allowing the cluster administrator to safely restart or upgrade Furiko at any time.

### Enhanced Timeout Handling

Furiko offers various knobs to control various types of timeouts for jobs, including [pending timeouts](./execution/job/timeout-retries.md#pendingtimeoutseconds) to better handle stuck jobs or dead nodes.

### Parameterization of Job Inputs

Furiko offers templating and parameterization of a single JobConfig using [context variables](./execution/jobconfig/context-variables.md), which performs substitution into the task template at runtime.

To parameterize the JobConfig, Furiko offers configuration of [Job Options](./execution/jobconfig/job-options.md), which describes and enforces the expected input for a Job.

### Federation Across Multiple Clusters

Furiko also provides add-on support for federation, cordoning and draining of JobConfigs across multiple Kubernetes clusters safely.

<!-- prettier-ignore -->
!!! info
    This feature is currently a work-in-progress.

### Monitoring, Notifications and Telemetry

Furiko also provides add-on support for monitoring and notifications of job executions and failures, via a variety of methods including Prometheus metrics, Slack webhooks and email notifications. In addition, Furiko provides alternative API servers to query, store and analyze large amounts of job executions per day, reducing the stress on `kube-apiserver`.

<!-- prettier-ignore -->
!!! info
    This feature is currently a work-in-progress.

## Comparison with `batch/v1`

The following is a short comparison between Furiko and `batch/v1` [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) and [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/):

| Feature                                                                | Furiko             | `batch/v1`                                                                                                              |
| ---------------------------------------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| **Scheduling**                                                         |                    |                                                                                                                         |
| Basic cron scheduling                                                  | :white_check_mark: | :white_check_mark:                                                                                                      |
| Specifying timezone of cron schedule                                   | :white_check_mark: | :grey_question:<br />([not officially supported](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)) |
| Load balancing using `H` in cron expressions                           | :white_check_mark: | :x:                                                                                                                     |
| [Back-scheduling](./execution/jobconfig/scheduling.md#back-scheduling) | :white_check_mark: | :white_check_mark:<br />(via `startingDeadlineSeconds`)                                                                 |
| Enqueue jobs for later                                                 | :white_check_mark: | :x:                                                                                                                     |
| **Task Execution**                                                     |                    |                                                                                                                         |
| Creation of tasks using PodTemplate                                    | :white_check_mark: | :white_check_mark:                                                                                                      |
| Retries using separate Pods                                            | :white_check_mark: | :x:                                                                                                                     |
| Pending timeouts for dead nodes                                        | :white_check_mark: | :x:                                                                                                                     |
| Task-level parallelism                                                 | :x:                | :white_check_mark:<br />([Job Patterns](https://kubernetes.io/docs/concepts/workloads/controllers/job/#job-patterns))   |
| **General**                                                            |                    |                                                                                                                         |
| Automatic cleanup via TTL                                              | :white_check_mark: | :white_check_mark:                                                                                                      |
| Concurrency-aware ad-hoc job execution                                 | :white_check_mark: | :x:                                                                                                                     |
| Parameterization of job inputs                                         | :white_check_mark: | :x:                                                                                                                     |
