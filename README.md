# Kubernetes kustomize manifest for polyfea artifacts

This repository contains the kustomize manifests for deploying polyfea artifacts on Kubernetes.

## Usage

### kubectl apply -k

This is basic deployment with default presets for polyfea artifacts.

* Create namespace

```shell
kubectl create namespace polyfea
```

* Deploy CRDs and controller

```shell
kubectl apply -k https://github.com/polyfea/manifests//controller --wait
```

* Deploy Material Design shell class

```shell
kubectl apply -k https://github.com/polyfea/manifests//md-shell --wait
```

* Port forward to controller service

```shell
kubectl port-forward svc/polyfea-controller 8080:8080 -n polyfea
```

* Access the controller

```shell
curl http://localhost:8080
```

### kustomize build

Similar to above, but patches material shell Microfrontend class to enable custom configuration.

* Create namespace

```shell
kubectl create namespace polyfea
```

* Deploy CRDs and controller

```shell
kubectl apply -k https://github.com/polyfea/manifests//controller --wait
```

* Create `my-polyfea` folder with `kustomization.yaml`

```yaml
resources:
- https://github.com/polyfea/manifests//md-shell

patches:
- path: microfrontendclass.patch.yaml
  target:
    group: polyfea.com
    version: v1alpha1
    kind: MicrofrontendClass
    name: material-shell


```

* Create JSON strategic patch `microfrontendclass.patch.yaml`

```yaml
spec:
    title: My Web App Title
    baseUri: /fea
    progressiveWebApp:
        webAppManifest:
            name: My Web App Title
            start_url: /fea
    # adapt CSP header if needed
    cspHeader: >-
        default-src 'self'; 
        font-src 'self'; s
        cript-src 'strict-dynamic' 'nonce-{NONCE_VALUE}'; 
        worker-src 'self'; 
        manifest-src 'self'; 
        style-src 'self' 'strict-dynamic' 'nonce-{NONCE_VALUE}'; 
        style-src-attr 'self' 'unsafe-inline'; 
        img-src *;
    ### depending if you use Gateway API or Ingress API, uncomment the respective section
    # routing:
        # Gateway API
        # parentRefs:
        #    - kind: Gateway
        #      name: my-gateway
        #      namespace: gw-namespace
        # Ingress API
        # ingressClass: nginx
```

* Deploy `my-polyfea` folder

```shell
kubectl apply -k my-polyfea --wait
```

* Port forward to controller service

```shell
kubectl port-forward svc/polyfea-controller 8080:8080 -n polyfea
```

* Access the controller

```shell
curl http://localhost:8080
```

### FlugCD GitOps

This is a more advanced deployment with FlugCD GitOps. It assumes you have a FlugCD instance running and configured.

* Create folder `my-polyfea` with `kustomization.yaml`

```yaml
resources:
- https://github.com/polyfea/manifests//gitops/fluxcd/default

# uncomment if you want to use Gateway API
# components: 
# - https://github.com/polyfea/manifests//gitops/fluxcd/with-gateway-cfg

# uncomment if you want to use Ingress API
# components: 
# - https://github.com/polyfea/manifests//gitops/fluxcd/with-ingress-cfg


configMapGenerator:
- name: polyfea-shell-cfg
  namespace: polyfea
  options:
    disableNameSuffixHash: true
  literals:
  - APP_TITLE=My Web App Title
  - APP_BASE_URI=/fea
  
  # uncomment if you want cutom CSP header
  # - CSP_HEADER=...;
  
  # uncomment if you want to use Gateway API 
  # - GATEWAY_NAME=my-gateway
  # - GATEWAY_NAMESPACE=gw-namespace

  # uncomment if you want to use Ingress API
  # - INGRESS_CLASS=nginx
```
