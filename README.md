# kustomize-demo
## 1-orignal-app

### Deploys the app from single `yaml` file

```
git checkout 1-orignal-app
kubectl apply -f rsvp.yaml
```

### Access the application

```
kubectl get svc
```
```
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        34s
mongodb      ClusterIP   10.107.208.233   <none>        27017/TCP      14s
rsvp         NodePort    10.102.228.221   <none>        80:32037/TCP   15s
```

### Access the Application
```
open http://localhost:32037
```

