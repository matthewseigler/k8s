> [!NOTE]
> Talos OS recommends installing the gateway controller CRDs, available [here](https://gateway-api.sigs.k8s.io/guides/getting-started/#installing-a-gateway-controller) prior to installing Cilium.  This is due to the fact the we might be setting `gatewayAPI.enable=true` and allows us to later use the Gateway API with Istio, which Cilium will then be looking for.

## Installation

### Using Cilium CLI

> [!NOTE]
> For Talos OS installations it is not recommended to use Cilium CLI but instead to use Helm.  However the Cilium CLI can still be useful for things like verifying connectivity using `cilium status --wait`.

Install the [Cilium CLI](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli).  

Install [Cilium](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli) using `cilium install --version <version>`.

#### Validate the Installation

To validate Cilium has been instlled, run `cilium status --wait`. 

To validate cluster network connectivity run `cilium connectivity test`.

### Using Helm

Install and update the Helm repos:

```
helm repo add cilium https://helm.cilium.io/
helm repo update
```

#### Create the Cilium values file.

> [!CAUTION]
> There is a known bug between Cilium and Talos OS which is corrected with `bpf.hostLegacyRouting` set to `true`.

> [!TIP]
> Set `cni.exclusive` to `false` to allow Istio to work alongside Cilium

#### Run the installation

Run `helm search repo cilium/cilium -l | head` to get the latest version of Cilium.

Run the following command to install Cilium with your Cilium values file in the `kube-system` namespace:

```
helm install cilium cilium/cilium --version <latestVersion> --namespace kube-system -f <cilium.yaml file>
```

> [!TIP] 
> Label the worker nodes so that Cilium can identify which nodes should handle external traffic using:
> ```
> kubectl label node <node-name> node.role.kubernetes.io/worker=""
> ```

