# Carbon Black installed by Flux
 This repository provides an example of using Flux to install Carbon Black. We also install Sealed Secrets as the Carbon Black install requires an access token as a Kubernetes secret but we don't want to store the value in plain text or base64 encoded in Git.

## base directory

### carbon-black directory
 The carbon-black directory contains the common (shared) resources necessary to install Carbon Black on any cluster.
 * `cbcontainers-dataplane_namespace.yaml` creates the `cbcontainers-dataplane` Kubernetes Namespace with the required labels.
 * `octarine-operator_gitrepository.yaml` creates a Flux GitRepository that points to https://github.com/octarinesec/octarine-operator. In the future, when Carbon Black publishes their operator and agent charts in a Helm repository, this would be replaced with a Flux HelmRepository.
 * `cbcontainers-operator_helmrelease.yaml` creates a Flux HelmRelease which installs the Carbon Black operator using Helm.
 * `cbcontainers-agent_helmrelease.yaml` creates a Flux HelmRelease which installs the Carbon Black agent using Helm. This includes non-secret values that are common across all cluster installs. There is a dependency on the operator HelmRelease so that the agent install happens after the operator install is complete.
 * `kustomization.yaml`

### sealed-secrets directory
 * `sealed-secrets_helmrepository.yaml` creates a Flux HelmRepository that points to https://bitnami-labs.github.io/sealed-secrets.
 * `sealed-secrets_helmrelease.yaml` creates a Flux HelmRelease that installs Sealed Secrets using Helm.

## tkg-workload-azure directory
 This directory is used by Flux to declare the desired state for this cluster. You first bootstrap the cluster using Flux. Then, you add Sealed Secrets to it. Finally, you add Carbon Black to it.

### flux-system directory
 See [Bootstrap](https://fluxcd.io/flux/cheatsheets/bootstrap/) in the Flux documentation.

### sealed-secrets directory
 * `kustomization.yaml` only points Flux to the base sealed-secrets directory as there a no cluster specific kustomizations.

### carbon-black directory
 * `cbcontainers-access-token_sealedsecret.yaml` creates a Kubernetes SealedSecret and, thus, a Kubernetes Secret for the access token. The file is created using `kubeseal`.
 * `patch-cbcontainers-agent_helmrelease.yaml` patches the agent HelmRelease to specify the cluster name as a value to be used for the HelmRelease for this cluster.
 * `kustomization.yaml` points Flux to the base carbon-black directory along with the above kustomizations for this cluster.