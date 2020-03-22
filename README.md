# kustomize-demo

## 2-kustomize-base

### Create a base directory and respective sub-directory for backend and frontend

```
$ tree base
base
├── backend
│   ├── backend-svc.yaml
│   ├── backend.yaml
│   └── kustomization.yaml
├── frontend
│   ├── frontend-svc.yaml
│   ├── frontend.yaml
│   └── kustomization.yaml
└── kustomization.yaml

2 directories, 7 files
```

```
$ cat base/kustomization.yaml
resources:
- backend
- frontend
```
```
$ cat base/backend/kustomization.yaml
resources:
- backend.yaml
- backend-svc.yaml
```
```
$ cat base/frontend/kustomization.yaml
resources:
- frontend.yaml
- frontend-svc.yaml
```

### Deploy the app

```
$ kustomize build base/ | kubectl apply -f -
service/mongodb created
service/rsvp created
deployment.apps/rsvp created
deployment.apps/rsvp-db created
```

### Access the app
```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP        64s
mongodb      ClusterIP   10.105.254.5   <none>        27017/TCP      40s
rsvp         NodePort    10.108.122.2   <none>        80:31623/TCP   40s
```
```
$ open http://localhost:31623
```

### Delete the app
```
$ kustomize build base/ | kubectl delete -f -
service "mongodb" deleted
service "rsvp" deleted
deployment.apps "rsvp" deleted
deployment.apps "rsvp-db" deleted
```
