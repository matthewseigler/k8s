
## Generate the ISO

Get the Talos OS image from [Factory](https://factory.talos.dev).


## Install Talosctl

Install `talosctl` using `brew`:

```
brew install siderolabs/tap/talosctl
```

## Generate Configuration

`talosctl gen config <cluster-name> <cluster-endpoint>`

`cluster-name` is to label your local client configuration.  It should be unique to local installation and is arbitrary.

`cluster-endpoint` is the Kubernetes API Server.  It should be complete URL with `https://`.  If using a single control plane node it may be set as the `custer-endpoint`.

> [!NOTE]
> For production cluster, you should have three control plane nodes, and have the endpoint allocate traffic to all three nodes.
> There are three common ways to do this: using a load-balancer, using Talos Linux’s built in VIP functionality, or using multiple DNS records.

## Modify the configs

> [!NOTE]
> If using an alternative CNI, then when generating Talos machine configs, set the `cluster.network.cni.name` to `none` so Talos doesn't install Flannel.

This is a temporary placeholder for future configurations.

## Talosctl - endpoints and nodes

In short: `endpoints` are where `talosctl` sends commands to, but the comand operates on the specified `nodes`.  The endpoint will forward the comand to the nodes, if needed.

Endpoints automatically proxy requests destined to another node in the cluster.  This means that you only need access to the control plane nodes in order to manage the rest of the cluster.

You can pass in `--endpoints <Control Plane IP Address>` or `-e <Control Plane IP Address>` to the current `talosctl` command.

## Apply the configs

```
talosctl apply-config --insecure \
    --nodes <nodeIP> \
    --file <file.yaml>
```

> [!NOTE]
> When using `--insecure`, you cannot specify an endpoint, and must directly access the node on port 50000.

## Talosconfig configuration

You reference which configuration file to use by the `--talosconfig` parameter:

```
talosctl --talosconfig=./talosconfig \
    --nodes <nodeIP> \
    --endpoints <controlPlaneNodeIP> \
    version
```

## Kubernetes Bootstrap

```
talosctl bootstrap \
    --nodes <controlPlaneNode> \
    --endpoints <KubernetesAPIServer> \
    --talosconfig=./talosconfig
```

> [!CAUTION]
> This command should only be run once.

After a few moments, you may download your Kubernetes client configuration:

```
talosctl kubeconfig \
    --nodes <controlPlaneNode> \
    --endpoints <KubernetesAPIServer> \
    --talosconfig=./talosconfig
```
