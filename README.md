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

5. Access bookinfo Product Page via correct route, e.g. http://istio-ingressgateway-istio-system.$ROUTE-HOSTNAME/productpage

## Generating the Istio manifests (if necessary)

1. Download `istioctl` from [istio.io](https://istio.io/latest/docs/setup/getting-started/#download)

2. Generate istio manifests via `istioctl` and append

```
istioctl manifest generate --set profile=openshift > manifests.yaml
```