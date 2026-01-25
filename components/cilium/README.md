## Installation

Install the [Cilium CLI](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli).  

Install [Cilium](https://docs.cilium.io/en/stable/gettingstarted/k8s-install-default/#install-the-cilium-cli) using `cilium install --version <version>`.

## Validate the Installation

To validate Cilium has been instlled, run `cilium status --wait`. 

To validate cluster network connectivity run `cilium connectivity test`.

