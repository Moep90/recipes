# cadvisor

cAdvisor 0.61.1 as a DaemonSet that serves the memory metrics of every container on its node, labelled with pod, namespace and container, designed in kadet-hub on cAdvisor's image and rendered by the kadet-hub `kubernetes` generator 1.14.0.

Source: `ghcr.io/google/cadvisor:0.61.1@sha256:4929b70e82079d001d0028b85d6a4dd25ad4666b719455c9542b983489963916`

Level: `deploys`

google/cadvisor ships a Kustomize base in `deploy/kubernetes/base`, which
`check` does not take, and at the tag v0.61.1 the base still runs 0.54.0.
The recipe is therefore designed on the image and follows the base where
the image leaves a choice open (DEC-368 in
[docs/specs/recipes.md](../docs/specs/recipes.md)). The image is documented
in [docs/runtime_options.md](https://github.com/google/cadvisor/blob/v0.61.1/docs/runtime_options.md)
and [deploy/kubernetes](https://github.com/google/cadvisor/tree/v0.61.1/deploy/kubernetes).
cAdvisor is Apache-2.0. The recipe follows cAdvisor's newest release,
0.61.1 of 2026-11-18. The digest is that of its index for linux/amd64 and
linux/arm64.

The recipe renders the DaemonSet `cadvisor` and the Service `cadvisor` on
port 8080, whose endpoints are the pods on every node. The pod runs as root
and mounts the node's `/`, `/sys` and `/run/containerd` read-only. Pod
Security `baseline` forbids `hostPath` volumes, so the namespace needs the
level `privileged`. cAdvisor names each container's pod through the
containerd socket in `/run/containerd`. The read-only mount does not keep
the pod from using containerd's API, which controls every container of the
node (DEC-369).

## Include

```yaml
classes:
  - recipes.cadvisor
parameters:
  namespace: cadvisor
  kapitan:
    vars:
      target: cadvisor
```

## Required parameters

No parameter has to be set.

## Settings

| Setting | Inventory | Default |
|---|---|---|
| Version | `components.cadvisor.image`, tag and digest | 0.61.1 |
| Metric families, as `--enable_metrics` takes them | `applications.cadvisor.enable_metrics` | `memory` |
| Resources | `components.cadvisor.resources` | requests 50m CPU and 128Mi, limit 512Mi |
| Tolerations | `components.cadvisor.tolerations` | every `NoSchedule` taint |
| Nodes | `components.cadvisor.node_selector`, `components.cadvisor.affinity` | every node |
| Arguments | `components.cadvisor.args.<key>`, such as `housekeeping: --housekeeping_interval=30s`, `~` drops one | see `cadvisor.yml` |

Metrics carry the labels `container_label_io_kubernetes_pod_name`,
`container_label_io_kubernetes_pod_namespace` and
`container_label_io_kubernetes_container_name` and no other label of the
container (DEC-370). The base also mounts `/dev/disk` for the device names
of disk metrics. A target that enables `disk` or `diskIO` adds it through
`components.cadvisor.volumes` and `components.cadvisor.volume_mounts`. Only
`memory` is proven.

Any list set in the target is appended to the recipe's list. To replace
one, such as `tolerations`, set the key to `~` in a class included after
the recipe and the new list in the target.

## Left out

The base's Namespace and ServiceAccount `cadvisor`: the target names the
namespace, and the pod mounts no token and is bound to no role. Its mounts
of `/var/run` and `/var/lib/docker`, the paths of the Docker runtime, and
of `/dev/disk`. The example overlays' Stackdriver sidecar, priority class
and perf events.

## Proof

`check cadvisor --level deploys` with `tests/target.yml` on kind for each
Kubernetes minor CI tests. A designed recipe has no objects to compare
with. `tests/smoke` reads `/metrics` through the Service and requires
`cadvisor_version_info` with the version of the DaemonSet's image. Through
the API server's pod proxy it reads every pod and requires
`container_memory_working_set_bytes` of the pod's own container with its
pod, namespace and container name, and fails on a label with the pod's UID
or a CPU series.
