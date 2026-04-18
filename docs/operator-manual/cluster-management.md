# Cluster Management

This guide is for operators looking to manage clusters on the CLI. If you want to use Kubernetes resources for this, check out [Declarative Setup](./declarative-setup.md#clusters).

Not all commands are described here, see the [argocd cluster Command Reference](../user-guide/commands/argocd_cluster.md) for all available commands.

## Adding a cluster

Run `argocd cluster add context-name`.

If you're unsure about the context names, run `kubectl config get-contexts` to get them all listed.

This will connect to the cluster and install the necessary resources for ArgoCD to connect to it.
Note that you will need privileged access to the cluster.

## Skipping cluster reconciliation

You can stop the controller from reconciling a cluster without removing it by annotating its secret:

```bash
kubectl -n argocd annotate secret <cluster-secret-name> argocd.argoproj.io/skip-reconcile=true
```

The cluster will still appear in `argocd cluster list` but the controller will skip reconciliation
for all apps targeting it. To resume, remove the annotation:

```bash
kubectl -n argocd annotate secret <cluster-secret-name> argocd.argoproj.io/skip-reconcile-
```

See [Declarative Setup - Skipping Cluster Reconciliation](./declarative-setup.md#skipping-cluster-reconciliation) for details.

## Managing cluster annotations

You can attach arbitrary key/value annotations to a cluster using `argocd cluster set`.
The syntax matches `kubectl annotate`: use `key=value` to add or update an annotation,
and append `-` to a key to remove it.

### Adding annotations

```bash
argocd cluster set my-cluster --annotation team=platform --annotation env=production
```

Verify with:

```bash
argocd cluster get my-cluster -o yaml
```

```yaml
annotations:
  env: production
  team: platform
name: my-cluster
server: https://my-cluster.example.com
...
```

### Removing an annotation

Append `-` to the annotation key to remove it, the same way `kubectl annotate` works:

```bash
argocd cluster set my-cluster --annotation env-
```

The annotation is gone from the next `get`:

```bash
argocd cluster get my-cluster -o yaml
```

```yaml
annotations:
  team: platform
name: my-cluster
server: https://my-cluster.example.com
...
```

> [!NOTE]
> Multiple `--annotation` flags can be combined in a single call. You can add some
> keys and remove others at the same time:
>
> ```bash
> argocd cluster set my-cluster --annotation env- --annotation tier=backend
> ```

## Removing a cluster

Run `argocd cluster rm context-name`.

This removes the cluster with the specified name.

> [!NOTE]
> **in-cluster cannot be removed**
>
> The `in-cluster` cluster cannot be removed with this. If you want to disable the `in-cluster` configuration, you need to update your `argocd-cm` ConfigMap. Set [`cluster.inClusterEnabled`](./argocd-cm-yaml.md) to `"false"`
