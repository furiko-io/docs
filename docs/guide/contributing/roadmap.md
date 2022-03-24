# Roadmap

Here we will briefly outline plans for the future of Furiko.

<!-- prettier-ignore -->
!!! info
    The following planned features are either already implemented in Furiko's internal closed-source project and will be eventually ported over, or is currently planned for future implementation.

    This document may not be up-to-date. Refer to [the GitHub repository](https://github.com/furiko-io/furiko/) for the latest progress.

## Main Components

### Execution

- Multiple cron schedules
- Extended topologies for concurrency policies
- Concurrency limits above 1 job at a time
- More complex scheduling rules
- More complex timeout handling
- Multi-stage job workflows
- Extensible task executors

### Other Components

- Federation across multiple Kubernetes clusters safely (drain and cordon semantics)
- Monitoring and notification of job executions (Prometheus metrics, Slack notifications, etc.)
- Telemetry to export historical execution data

## Add-on Components

- Command-line tools to execute and manage job configs
- User interface
