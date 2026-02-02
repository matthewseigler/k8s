Base: oci://gcr.io/istio-release/charts/base

Istiod: oci://gcr.io/istio-release/charts/istiod

kubectl create namespace istio-system

helm install istio-base oci://gcr.io/istio-release/charts/base -n istio-system --set defaultRevision=default --create-namespace

helm install istiod oci://gcr.io/istio-release/charts/istiod -n istio-system --wait
