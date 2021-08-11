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
* [An OpenShift `Route` resource](istio-manifests/ingres-gateway-route.yaml) exposing the `istio-ingressgateway` for ingress traffic to the mesh

3. Label `bookinfo` namespace with injection label
```
oc label namespace bookinfo istio-injection=enabled
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