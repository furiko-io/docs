# Using the CLI

This guide provides details on how to install and use the Furiko CLI tool, `furiko`.

## Installation Instructions

Currently, `furiko` is only released via Furiko's GitHub releases. You can install `furiko` by downloading the executable binary from [Furiko's GitHub releases page](https://github.com/furiko-io/furiko/releases) directly.

1. Download the binary that matches your system's OS and architecture (e.g. `furiko_darwin_amd64`).
2. Rename the binary to `furiko` and make it executable:

   ```sh
   chmod +x furiko
   ```

## Usage Instructions

Running `furiko --help` will list all available commands for the CLI tool.

```sh
$ furiko --help
Command-line utility to manage Furiko.

Usage:
  furiko [command]

Available Commands:
  completion  generate the autocompletion script for the specified shell
  disable     Disable automatic scheduling for a JobConfig.
  enable      Enable automatic scheduling for a JobConfig.
  get         Get one or more resources by name.
  help        Help about any command
  kill        Kill an ongoing Job.
  list        List all resources by kind.
  run         Run a new Job.

Flags:
      --dynamic-config-name string        Overrides the name of the dynamic cluster config. (default "execution-dynamic-config")
      --dynamic-config-namespace string   Overrides the namespace of the dynamic cluster config. (default "furiko-system")
  -h, --help                              help for furiko
      --kubeconfig string                 Path to the kubeconfig file to use for CLI requests.
  -n, --namespace string                  If present, the namespace scope for this CLI request.
  -v, --v int                             Sets the log level verbosity.

Use "furiko [command] --help" for more information about a command.
```

### Example Usage

- List all JobConfigs:

  ```sh
  $ furiko list jobconfigs
  NAME                               STATE           ACTIVE   QUEUED   CRON SCHEDULE   LAST SCHEDULED
  daily-cronjob-sends-email-report   ReadyEnabled    0        0        15 8 * * *      8h
  weekly-report-job                  ReadyEnabled    0        0        0 8 * * MON     2d
  refund-customer-payments           Ready           1        2
  ```

- Get a single JobConfig:

  ```sh
  $ furiko get jobconfig weekly-report-job
  JOB CONFIG METADATA
  Name:       weekly-report-job
  Namespace:  default
  Created:    Tue, 24 May 2022 16:05:16 +08:00 (8 days ago)

  CONCURRENCY
  Policy:  Forbid

  JOB STATUS
  State:           ReadyEnabled
  Queued Jobs:     0
  Active Jobs:     0
  Last Scheduled:  Mon, 30 May 2022 08:00:00 +08:00 (2 days ago)
  ```

- Run a JobConfig to create an [ad-hoc Job](../execution/job/adhoc-execution.md), prompting the user to enter [job options](../execution/jobconfig/job-options.md):

  ```sh
  $ furiko run jobconfig-sample
  ```

  ![Example of `furiko run` in action](../execution/job/img/furiko-run.png)
