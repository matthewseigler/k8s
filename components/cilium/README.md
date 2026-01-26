> [!NOTE]
> Talos OS recommends installing the gateway controller CRDs, available [here](https://gateway-api.sigs.k8s.io/guides/getting-started/#installing-a-gateway-controller) prior to installing Cilium.  This is due to the fact the we might be setting `gatewayAPI.enable=true` and allows us to later use the Gateway API with Istio, which Cilium will then be looking for.

## Installation

Install the [Cilium CLI](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli).  

Install [Cilium](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli) using `cilium install --version <version>`.

## Validate the Installation

To validate Cilium has been instlled, run `cilium status --wait`. 

To validate cluster network connectivity run `cilium connectivity test`.