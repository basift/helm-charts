# Kured Reboot Helm Chart

This Helm chart provides a way to automatically trigger node reboots in a Kubernetes cluster using a CronJob and a custom script.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+

## Installing the Chart

First, add the Helm repository:

```bash
helm repo add kured-reboot https://basift.github.io/helm-charts
```

Then, update the repository:

```bash
helm repo update
```

To install the chart with the release name `my-release`, run the following command:

```bash
helm install my-release kured-reboot/kured-reboot
```

## Uninstalling the Chart

To uninstall the chart, run the following command:

```bash
helm uninstall my-release
```

## Configuration

The following table lists the configurable parameters of the `kured-reboot` chart and their default values.

| Parameter                 | Description                                     | Default                       |
|---------------------------|-------------------------------------------------|-------------------------------|
| `cronJob.name`            | Name of the CronJob                             | `kured-reboot-cronjob`        |
| `cronJob.schedule`        | Schedule for the CronJob (in cron syntax)       | `"0 0 31 2 *"`                |
| `cronJob.suspend`         | Whether to suspend the CronJob                  | `true`                        |
| `cronJob.image.repository`| Repository for the CronJob container image      | `bitnami/kubectl`             |
| `cronJob.image.tag`       | Tag for the CronJob container image             | `latest`                      |

You can override the default values by providing a YAML file with the desired configuration during installation:

```bash
helm install my-release kured-reboot/kured-reboot -f values.yaml
```

## How It Works

The `kured-reboot` chart deploys a CronJob that has never happened scheduler and can be triggered only manually. The CronJob executes a custom script that performs the following steps:

1. Checks if any node in the cluster has the annotation `weave.works/kured-reboot-in-progress`. If an annotation is found, it means a reboot is already in progress, so the script exits.

2. If no reboot is in progress, the script iterates over all the nodes in the cluster (except the node where the script is running).

3. For each node, the script creates a debug pod, triggers a reboot by creating a file `/var/run/reboot-required` on the node, and deletes the debug pod.

4. The script waits for the rebooted node to become ready again before proceeding to the next node.

5. After rebooting all other nodes, the script triggers a reboot on the node where it is running.

This process ensures that nodes are rebooted one by one, minimizing disruption to the cluster.

## Note

Rebooting nodes in a Kubernetes cluster can have an impact on running workloads. Make sure to schedule the CronJob to run during a maintenance window or low-traffic period. It's also recommended to have enough replicas of your workloads to ensure high availability during the reboot process.

## License

This chart is released under the [Apache License 2.0](LICENSE).
