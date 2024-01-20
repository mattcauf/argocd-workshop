# Workshop
The goal is to setup a gitops environment perform a deployment of a stateless app.
We will use a kubernetes cluster and argocd.


![](img/argocd-sync-flow.png)
## Setup K8s
rancher k8s distro will be use.
Download:
https://rancherdesktop.io/
### Windows
wsl --install

## Argo cd

### Install argoCd
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pod -n argocd
kubectl port-forward service/argocd-server 8080:80 -n argocd

```

#### Get argocd password Unix
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```
#### Get argocd password Windows
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}"
```
take the output and decode it by using the following website
https://www.base64decode.org/

#### make Argocd an admin of your cluster

```
kubectl apply -f .\apps\argocd\role.yaml -n argocd
```

This apply the following yaml.
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: argocd-application-controller
rules:
  - apiGroups:
      - '*'
    resources:
      - '*'
    verbs:
      - '*'
  - nonResourceURLs:
      - '*'
    verbs:
      - '*'
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: argocd-application-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: argocd-application-controller
subjects:
- kind: ServiceAccount
  name: argocd-application-controller
  namespace: argocd
```


Go to http://localhost:8080 
the username is admin

### Create an app
click on new app then edit Yaml and paste the following 
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: snake-app
  namespace: argocd
spec:
  destination:
    namespace: snake
    server: https://kubernetes.default.svc
  project: default
  source: 
    path: apps/snake/base/
    repoURL: https://github.com/mattcauf/argocd-workshop.git
    targetRevision: main
  syncPolicy:
    automated:
      prune: true
      selfHeal: false
    syncOptions:
    - CreateNamespace=true
```
kubectl port-forward service/snake 8080:80 -n snake