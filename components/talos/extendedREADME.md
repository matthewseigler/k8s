# Getting Started

See the official Talos OS documentation for [Getting Started](https://docs.siderolabs.com/talos/v1.12/getting-started/getting-started) and [Production Notes](https://docs.siderolabs.com/talos/v1.12/getting-started/prodnotes).

> [!NOTE]
> Several of these steps seem either obvious or unnecessary and as such will be skipped, however their outline might be left as a placeholder.

## Prerequisites

To create a Kubernetes cluster with Talos, you'll need `talosctl`.

`brew install siderolabs/tap/talosctl`

## Overview

1. Boot - Start machines with Talos Linux image
2. Configure - Create a root of trust certificate authority and generate configuration files
3. Apply - Apply machine configurations to the nodes
4. Create - Set up local `talosctl` client
5. Bootstrap - Initialize the Kubernetes cluster

## Downlaod Talos Linux Image

Create the latest ISO for your architecture from the [Image factory](https://factory.talos.dev/).

## Boot Machine

## Store Node IP Addresses in a Variable

> [!NOTE]
> Variable creation can be skipped and attributes can be hard coded instead.

1. Copy the IP address displayed on each machine console, including the control plane and any worker nodes you’ve created. 

If you don’t have a display connected, retrieve the IP addresses from your DHCP server.

2. Create a variable for your control plane node’s IP address by replacing `<your-control-plane-ip>` with the actual IP: 

`export CONTROL_PLANE_IP=<your-control-plane-ip>`

3. If you have worker nodes, store their IP addresses in a Bash array. Replace each `<worker-ip>` placeholder with the actual IP address of a worker node. You can include as many IP addresses as needed:

`WORKER_IP=("<worker-ip-1>" "<worker-ip-2>" "<worker-ip-3>"...)`

## Unmount the ISO

Unplugging the installation USB driver or unmounting the ISO will prevent accidental installation onto the USB drive and make it clearer which disk to select for installation.

## Learn About Your Installation Disks

`talosctl get disks --insecure --nodes $CONTROL_PLANE_IP`

## Generate Cluster Configuration

> [!NOTE]
> Variable creation can be skipped and hard coded instead.

Talos Linux is configured entirely using declarative configuration files avoiding the need to deal with SSH and running commands. 

To generate these declarative configuration files:

1. Define variables for your cluster name and the disk ID from step 5. Replace the placeholders with your actual values:

```
export CLUSTER_NAME=<cluster_name>
export DISK_NAME=<control_plane_disk_name>
```

2.  Run this command to generate the configuration file: 

`talosctl gen config $CLUSTER_NAME https://$CONTROL_PLANE_IP:6443 --install-disk /dev/$DISK_NAME`

This command will generate machine configurations that specify the Kubernetes API endpoint (which is your control plane node's IP) for cluster communication and the target disk for the Talos installation.

The three files generates:

- controlplane.yaml - The configuration for control plane nodes.
- worker.yaml - The configuration for worker nodes.
- talosconfig - The `talosctl` configuration file which is used to connect to and authenticate access to the cluster.

## Modify Configuration

> [!IMPORTANT]
> This is where the Talos OS documentation falls short, as it provides no configuration options in the [Getting Started](https://docs.siderolabs.com/talos/v1.12/getting-started/getting-started) or [Producation Notes](https://docs.siderolabs.com/talos/v1.12/getting-started/prodnotes) that I can tell.
> As such this is where a lot of the heavy lifting will start and are modifications that I have made.  As such, please take them at face value and be aware there can be (and most likely are) errors.

For ESXi installation `interfaces.interfaces` needs to set be to `ens33`.

> [!WARNING]
> If you run `talosctl get links --insecure --nodes <nodeIP>` at this stage it will most likely show `eth0` as your interface.  This seems to be a default interface name given, however upon restart this will change to `ens33` on some installations and as such this will cause the installation to hang.  
> An option to check this is `talosctl config endpoint <nodeIP>` followed by `talosctl --nodes <nodeIP> get addressspecs`.

If the node IP needs to be altered or set, use `interfaces.addresses` and set it to `<ip/CIDR>`.

Assign the default gateway with `interfaces.routes.gateway` set to `<defaultGateway>`

Create and assign a virtual IP for the Kubernetes API with `interfaces.vip.ip` set to `<ipAddress>`

Ensure that in the cluster section, `controlPlane.endpoint` is the same VIP as above.

An example of what this looks like can be seen below:

```
machine:
  network:
      interfaces:
          - interface: ens33
            addresses:
              - 10.0.80.11/24
            routes:
              - network: 0.0.0.0/0
                gateway: 10.0.80.1
            vip:
              ip: 10.0.80.10
      nameservers:
          - 1.1.1.1
```

```
cluster:
    controlPlane:
        endpoint: https://10.0.80.10:6443
```
The hostname can be set statically with the configuration below, or it can be automatically generated by comment out `hostname` and replacing it with `auto: stable`.

```
apiVersion: v1alpha1
kind: HostnameConfig
hostname: cp-01
```

### Cluster Network, Cilium, and Kube-Proxy

The goal of the cluster network is to update pod and service subnets.

When configuring the machine config file if you wish to use Cilium or another CNI then set the CNI to none.  This prevents Flannel from being installed.  If deploying without kube-proxy, then kube proxy should be disabled as well 

This should be applied only to the control plane nodes.

```
cluster:
  network:
    dnsDomain: cluster.local
    podSubnets:
      - 192.168.0.0/16
    serviceSubnets:
      - 192.0.0.0/12
    cni:
      name: none
  proxy:
    disabled: true
```

### KubePrism

Other Talos OS guides assumes KubePrism is enabled, thus here is the information for configuration it.

KubePrism is installed by default, with a default port of 7445.  To enable KubePrism, apply the following machine config (this should be applied to all nodes):

```
machine:
  features:
    kubePrism:
      enabled: true
      port: 7445
```

## Apply Configurations



