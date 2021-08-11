## ArgoCD based Istio Installation on OCP
1. Create `istio-system` and `bookinfo` namespace

2. Create istio app in ArgoCD e.g.
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: istio
spec:
  destination:
    name: ''
    namespace: istio-system
    server: 'https://kubernetes.default.svc'
  source:
    path: istio-manifests
    repoURL: '$REPO_URL_HERE'
    targetRevision: HEAD
  project: default
```
Note that the manifests folder contains the Istio manifests along with OpenShift specific resources:
* [SCC to grant `anyuid` permissions](istio-manifests/istio-sys-scc-anyuid.yaml) to istio pods and injected workloads
* [A `NetworkAttachmentDefinition` resource](istio-manifests/network-attach-istio-cni.yaml) which supports the `istio-cni` plugin 
* [An OpenShift `Route` resource](istio-manifests/ingress-gateway-route.yaml) exposing the `istio-ingressgateway` for ingress traffic to the mesh

See the [official docs](https://istio.io/latest/docs/setup/platform-setup/openshift/) for more info.

3. Label `bookinfo` namespace with the injection and discovery label and label `istio-system` with the discovery label.

The discovery label is used to limit the namespaces that will participate in the mesh since by default the full cluster is available. See [this blog post](https://istio.io/latest/blog/2021/discovery-selectors/) for more info. (Note: this feature was contributed to Istio by solo.io)
```
oc label namespace bookinfo istio-injection=enabled
oc label namespace bookinfo istio-discovery=enabled
oc label namespace istio-system istio-discovery=enabled
```

4. Create bookinfo app in ArgoCD e.g.
```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: bookinfo
spec:
  destination:
    name: ''
    namespace: bookinfo
    server: 'https://kubernetes.default.svc'
  source:
    path: bookinfo
    repoURL: '$REPO_URL_HERE'
    targetRevision: HEAD
  project: default
```

5. Access bookinfo Product Page via correct route, e.g. http://istio-ingressgateway-istio-system.$ROUTER-DOMAIN/productpage

## Generating the Istio manifests (if necessary)

1. Download `istioctl` from [istio.io](https://istio.io/latest/docs/setup/getting-started/#download)

2. Generate istio manifests via `istioctl` and append

```
istioctl manifest generate --set profile=openshift > manifests.yaml
```

Using discovery selectors:
```
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
spec:
  profile: openshift
  tag: 1.10.3
  meshConfig:
    discoverySelectors:
    - matchLabels:
        istio-discovery: enabled
```