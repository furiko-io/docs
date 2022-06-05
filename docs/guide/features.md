# Features

Furiko specializes in time-based scheduling of scheduled and adhoc jobs. The following is a non-exhaustive list of features and enhancements offered by Furiko.

## Feature List

### Timezone-aware Cron Scheduling

Furiko allows users to specify a [cron schedule](./execution/jobconfig/scheduling.md#cronexpression) to automatically schedule jobs, supporting up to a per-second granularity.

Furiko also offers native support for scheduling jobs on a [cron schedule with timezones](./execution/jobconfig/scheduling.md#crontimezone). This allows users to specify their cron schedule in a timezone that is familiar to them, or one that is standardized in their application. If not specified, Furiko also supports specifying a cluster-wide default timezone for all cron schedules.

### Cluster-wide Load Balancing

Furiko offers a unique, [extended cron syntax](./execution/jobconfig/cron-syntax.md#hash-based-load-balancing) that may drastically improve the performance of running distributed cron at scale.

Furiko introduces a new syntax like `H/5 * * * *`. The `H` token means to deterministically choose a "random" number in the given range, avoiding thundering herd effects when thousands of other jobs also start at the exact same time. This allows users to still be able to run their job at a fixed interval of 5 minutes, but evenly spread across the time range.

### Strong Concurrency Handling

Furiko introduces stronger [handling of multiple concurrent jobs](./execution/jobconfig/concurrency.md).Using the `Forbid` concurrency policy, Furiko takes a very [strict approach](./development/architecture/execution-controller.md#jobqueuecontroller) to ensuring that multiple jobs will never be started at the same time, free of race conditions.

### Scheduling Future Executions

Furiko allows multiple jobs to be queued for [executing at a later time](./execution/job/adhoc-execution.md#scheduling-adhoc-future-executions), or to start a new job execution immediately after the current job execution is finished.

### Preventing Missed and Double Executions

Furiko provides strong guarantees to schedule jobs even in the face of failure, and is able to prevent both double runs and missed runs in many cases.

Furiko prevents double scheduled runs by using [deterministic name formats](./development/architecture/execution-controller.md#jobcontroller). It also prevents missed runs using [back-scheduling](./execution/jobconfig/scheduling/#back-scheduling) to tolerate short downtime, allowing the cluster administrator to safely restart or upgrade Furiko at any time.

### Enhanced Timeout Handling

Furiko offers various types of timeouts when creating a job. For example, the [pending timeouts](./execution/job/timeout-retries.md#pendingtimeoutseconds) is able to gracefully handle node outages, preventing a scheduled job from not progressing because one execution got stuck.

### Job Options

Furiko allows you to use JobConfigs as templates for Jobs, by passing parameters into the Job. The JobConfig can be parameterized with [Job Options](./execution/jobconfig/job-options.md), which defines a structured input that will be validated and substituted at runtime.

### Running multiple parallel tasks per Job

Furiko supports parallelism of tasks within a Job, which supports index number-based, index key-based or matrix-based expansion of tasks that will be run in parallel. A completion strategy can also be defined that will automatically terminate all parallel tasks when certain conditions are met.

### Federation Across Multiple Clusters

Furiko also provides add-on support for federating, cordoning and draining of JobConfigs across multiple Kubernetes clusters safely.

<!-- prettier-ignore -->
!!! info
    This feature is currently planned in the [Roadmap](./contributing/roadmap.md).

### Monitoring, Notifications and Telemetry

Furiko also provides add-on support for monitoring and notifications of job executions and failures, via a variety of methods including Prometheus metrics, Slack webhooks and email notifications. In addition, Furiko provides alternative API servers to query, store and analyze large amounts of job executions per day, reducing the stress on `kube-apiserver`.

<!-- prettier-ignore -->
!!! info
    This feature is currently planned in the [Roadmap](./contributing/roadmap.md).

## Comparison with `batch/v1`

The following is a short comparison between Furiko and `batch/v1` [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) and [CronJobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/):

| Feature                                                                                         | Furiko             | `batch/v1`                                                                                                              |
| ----------------------------------------------------------------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| **Scheduling**                                                                                  |                    |                                                                                                                         |
| [Cron expressions](./execution/jobconfig/scheduling.md#cronexpression)                          | :white_check_mark: | :white_check_mark:                                                                                                      |
| [Cron timezone](./execution/jobconfig/scheduling.md#crontimezone)                               | :white_check_mark: | :grey_question:<br />([not officially supported](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/)) |
| [Cron load balancing](./execution/jobconfig/cron-syntax.md#hash-based-load-balancing)           | :white_check_mark: | :x:                                                                                                                     |
| [Back-scheduling](./execution/jobconfig/scheduling.md#back-scheduling)                          | :white_check_mark: | :white_check_mark:<br />(via `startingDeadlineSeconds`)                                                                 |
| [Enqueue jobs for later](./execution/job/adhoc-execution.md#scheduling-adhoc-future-executions) | :white_check_mark: | :x:                                                                                                                     |
| **Task Execution**                                                                              |                    |                                                                                                                         |
| [Retries using separate Pods](./execution/job/timeout-retries.md#retries)                       | :white_check_mark: | :x:                                                                                                                     |
| [Pending timeouts for dead nodes](./execution/job/timeout-retries.md#pendingtimeoutseconds)     | :white_check_mark: | :x:                                                                                                                     |
| [Multiple parallel Pods per Job](./execution/job/parallelism.md)                                | :white_check_mark: | :white_check_mark:<br />([Job Patterns](https://kubernetes.io/docs/concepts/workloads/controllers/job/#job-patterns))   |
| **General**                                                                                     |                    |                                                                                                                         |
| [Automatic cleanup with TTL](./execution/job/garbage-collection.md)                             | :white_check_mark: | :white_check_mark:                                                                                                      |
| [Concurrency-aware ad-hoc job execution](./execution/job/adhoc-execution.md#concurrency)        | :white_check_mark: | :x:                                                                                                                     |
| [Parameterization of job inputs](./execution/jobconfig/job-options.md)                          | :white_check_mark: | :x:                                                                                                                     |
