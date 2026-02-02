Start with Talos => Cilium

Then grab the Gateway CRDs.  Their official documentation is [here](https://gateway-api.sigs.k8s.io/guides/getting-started/#installing-gateway-api) with the command (depending on version) looking something like this:

`kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/standard-install.yaml`

