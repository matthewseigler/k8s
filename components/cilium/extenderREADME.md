> [!NOTE]
> Cilium is a complex CNI and has added complexity when used with Talos OS.  As such, both [Talos OS Official Documentation](https://docs.siderolabs.com/kubernetes-guides/cni/deploying-cilium) and [Cilium Official Documentation](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#k8s-quick-install) are referenced. 

# Talos OS Documentation

Talos OS has several recommendations prior to jumping to the Cilium installation so they will be referenced here first.

> [!NOTE]
> This documentation will outline installing Cilium CNI v1.18.0 on Talos in six different ways. Adhering to Talos principles we’ll deploy Cilium with IPAM mode set to Kubernetes, and using the cgroupv2 and bpffs mount that talos already provides. As Talos does not allow loading kernel modules by Kubernetes workloads, SYS_MODULE capability needs to be dropped from the Cilium default set of values, this override can be seen in the helm/cilium cli install commands. Each method can either install Cilium using kube proxy (default) or without: [Kubernetes Without kube-proxy](https://docs.cilium.io/en/stable/network/kubernetes/kubeproxy-free/). 
> In this guide we assume that [KubePrism](https://docs.siderolabs.com/kubernetes-guides/advanced-guides/kubeprism) is enabled and configured to use the port 7445.

## KubePrism

Since the Talos OS guide assumes KubePrism is installed, here is the information for configuration it.

KubePrism is installed by default, with a default port of 7445.  To enable KubePrism, apply the following machine config (this should be applied to all nodes):

```
machine:
  features:
    kubePrism:
      enabled: true
      port: 7445
```

## Machine config preparation

When generating the machine config for a node set the CNI to none.  This prevents Flannel from being installed.  If deploying without kube-proxy, then kube proxy should be disabled as well:

```
cluster:
  network:
    cni:
      name: none
  proxy:
    disabled: true
```

## Installation

> [!NOTE]
> Note: It is recommended to template the cilium manifest using helm and use it as part of Talos machine config.
> There are steps to install Cilium using the Cilium CLI if you would prefer, however since it is not how Talos OS recommends it will not be documented here.
> With that said, it is advised to install Cilium CLI if you wish to utilize its local commands such as validating the installation with `cilium status --wait` or validating network connectivity with `cilium connectivity test`.

Refer to [Cilium's Documentation](https://docs.cilium.io/en/stable/installation/k8s-install-helm/) for more information.

Add the helm repo for Cilium:

```
helm repo add cilium https://helm.cilium.io/
helm repo update
```

> [!NOTE]
> While there are several ways to install Cilium, for this installation we will be doing so without kube-proxy and with Gateway API support.
> If using a newer version of Cilium make sure you change the `--version`.

```
helm install \
    cilium \
    cilium/cilium \
    --version 1.18.0 \
    --namespace kube-system \
    --set ipam.mode=kubernetes \
    --set kubeProxyReplacement=true \
    --set securityContext.capabilities.ciliumAgent="{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}" \
    --set securityContext.capabilities.cleanCiliumState="{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}" \
    --set cgroup.autoMount.enabled=false \
    --set cgroup.hostRoot=/sys/fs/cgroup \
    --set k8sServiceHost=localhost \
    --set k8sServicePort=7445
    --set=gatewayAPI.enabled=true \
    --set=gatewayAPI.enableAlpn=true \
    --set=gatewayAPI.enableAppProtocol=true
```

