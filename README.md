# falkordb-kubernetes (Helm)

Standalone source repo: [github.com/Fluder-Paradyne/falkordb-kubernetes-helm](https://github.com/Fluder-Paradyne/falkordb-kubernetes-helm)

Example Helm chart that runs **FalkorDB** using the **Bitnami Redis** chart in **replication + Sentinel** mode, with the fixes required for a non-Bitnami Redis image.

## Why this exists

The [FalkorDB Kubernetes guide](https://docs.falkordb.com/operations/k8s-support.html) suggests swapping the Bitnami image for `falkordb/falkordb` and adding `extraFlags` for `loadmodule`. That is not sufficient on current Bitnami Redis charts because:

1. Default `commonConfiguration` loads RedisSearch/ReJSON from paths that exist only in the Bitnami image.
2. `start-node.sh` sources `/opt/bitnami/scripts/lib*.sh`, which are not present in the FalkorDB image.
3. An init container that uses `cp -a` into `emptyDir` can fail with `preserving times: Operation not permitted`.

This chart applies the corrected `values.yaml` pattern (custom `commonConfiguration`, init container with `cp -r`, staging volume).

## Prerequisites

- Helm 3.8+ (OCI dependencies)
- Kubernetes cluster

## Install

```bash
cd falkordb-kubernetes-helm
helm dependency update
helm install falkordb . --namespace falkordb --create-namespace \
  --set redis.auth.password='YOUR_SECRET_PASSWORD'
```

If you omit `redis.auth.password`, Bitnami generates a random password; read it from the secret named `<release>-redis` as shown in the post-install notes.

## Upgrade

```bash
helm dependency update
helm upgrade falkordb . --namespace falkordb
```

## Uninstall

```bash
helm uninstall falkordb --namespace falkordb
```

## Values

All Bitnami Redis settings are under the `redis` key. See `values.yaml`.

## Packaged chart (OCI or chart museum)

```bash
helm dependency update
helm package .
# Push to OCI registry, e.g.:
# helm push falkordb-kubernetes-0.1.0.tgz oci://registry.example.com/charts
```

## License

Chart metadata only; FalkorDB and Bitnami images/charts have their own licenses.
