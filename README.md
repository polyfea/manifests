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
  kubectl apply -k https://github.com/polyfea/manifests//controller
  kubectl wait -n polyfea  deployment/polyfea-controller --for condition=available=True --timeout=60s
  ```

* Deploy Material Design shell class

  ```shell
  kubectl apply -k https://github.com/polyfea/manifests//md-shell
  ```

* Port forward to controller service

  ```shell
  kubectl port-forward svc/polyfea-controller 8080:80 -n polyfea
  ```

* Open browser and access the controller at `http://localhost/fea`

* Apply simple sample microfrontends

  ```shell
  kubectl apply -k https://github.com/polyfea/manifests//sample-feas
  ```

  and reload the page `http://localhost/fea`-

### kustomize build

Similar to above, but patches material shell Microfrontend class to enable custom configuration.

* Create namespace

  ```shell
  kubectl create namespace polyfea
  ```

* Deploy CRDs and controller

  ```shell
  kubectl apply -k https://github.com/polyfea/manifests//controller
  kubectl wait -n polyfea  deployment/polyfea-controller --for condition=available=True --timeout=60s
  ```

* Create `my-polyfea` folder with `kustomization.yaml`

  ```yaml
  resources:
  - https://github.com/polyfea/manifests//md-shell
  # comment out if you don't want to use sample microfrontends
  - https://github.com/polyfea/manifests//sample-feas

  patches:
  - path: microfrontendclass.patch.yaml
  ```

* Create JSON strategic patch `microfrontendclass.patch.yaml`

  ```yaml
  apiVersion: polyfea.github.io/v1alpha1
  kind: MicroFrontendClass
  metadata:
    name: md-shell
    namespace: polyfea
  spec:
      title: My Web App Title
      baseUri: /fea
      progressiveWebApp:
          webAppManifest:
              name: My Web App Title
              start_url: /fea
      # adapt CSP header if needed
      # cspHeader: >-
      #     default-src 'self'; 
      #     font-src 'self'; s
      #     cript-src 'strict-dynamic' 'nonce-{NONCE_VALUE}'; 
      #     worker-src 'self'; 
      #     manifest-src 'self'; 
      #     style-src 'self' 'strict-dynamic' 'nonce-{NONCE_VALUE}'; 
      #     style-src-attr 'self' 'unsafe-inline'; 
      #     img-src *;
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
  kubectl port-forward svc/polyfea-controller 8080:80 -n polyfea
  ```

* Open browser and access the controller at `http://localhost/fea`

### FlugCD GitOps

This is a more advanced deployment with FlugCD GitOps. It assumes you have a FlugCD instance running on your cluster

* Create folder `my-polyfea` with `kustomization.yaml`

  ```yaml
  resources:
  - https://github.com/polyfea/manifests//gitops/fluxcd

  # uncomment if you want polyfea controller to manage HttpRoute or Ingress resources
  # patches:
  # - target:
  #     kind: Kustomization
  #     name: material-shell
  #     namespace: polyfea
  #   patch: |-
  #     - op: add
  #       path: /spec/components/-
  #       value: ./components/gateway-cfg # or ./components/ingress-cfg

  # This ConfigMap is used by postBuild event of FluxCD to update the configuration of the material shell
  configMapGenerator:
  - name: polyfea-shell-cfg
    namespace: polyfea
    options:
      disableNameSuffixHash: true
    literals:
    - APP_TITLE=My Web App Title
    - APP_BASE_URI=/fea
    
    # uncomment if you want custom CSP header
    # - CSP_HEADER=...;
    
    # uncomment if you want to use Gateway API 
    # - GATEWAY_NAME=my-gateway
    # - GATEWAY_NAMESPACE=gw-namespace

    # uncomment if you want to use Ingress API
    # - INGRESS_CLASS=nginx
  ```

* Deploy `my-polyfea` folder

  ```shell
  kubectl apply -k my-polyfea
  ```

* Port forward to controller service

  ```shell
  kubectl port-forward svc/polyfea-controller 8080:80 -n polyfea
  ```

* Open browser and access the controller at `http://localhost/fea`

