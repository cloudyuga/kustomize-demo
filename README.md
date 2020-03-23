# kustomize-demo

## 8-putting-it-all-together

Let us now create one for overlay called `prod`, which is similar to `staging`. In `prod` setup,  I would like 3 replicas of the application instead of 1. So we'll create one more `patch` and apply it while deploying the app. 

### Create a `patch` file `overlays/prod/replica-count.yaml` with following content
```
$ cat overlays/prod/replica-count.yaml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rsvp
spec:
  replicas: 3
```


### Update the `overlays/prod/kustomization.yaml` file accordingly
```
$ cat overlays/prod/kustomization.yaml
```
```
bases:
- ../../base
namePrefix: prod-
patchesStrategicMerge:
- prod-configmap.yaml
- replica-count.yaml
configMapGenerator:
- name: rsvpconfig-staging
  literals:
    - TEXT1="Welcome to"
    - TEXT2="Production"
```

We have updated the `configMap`

### Build the `YAML` file

```
$ kustomize build overlays/prod/
```
```
apiVersion: v1
data:
  TEXT1: Welcome to
  TEXT2: Production
kind: ConfigMap
metadata:
  name: prod-rsvpconfig-staging-m42dtkmhfh
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    meetup: First Kubernetes & CNCF Online Meetup
  labels:
    app: rsvpdb
    demo: online
  name: prod-meetup-mongodb
spec:
  ports:
  - port: 27017
    protocol: TCP
  selector:
    appdb: rsvpdb
    demo: online
---
apiVersion: v1
kind: Service
metadata:
  annotations:
    meetup: First Kubernetes & CNCF Online Meetup
  labels:
    app: rsvp
    demo: online
  name: prod-meetup-rsvp
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: web-port
  selector:
    app: rsvp
    demo: online
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    meetup: First Kubernetes & CNCF Online Meetup
  labels:
    demo: online
  name: prod-meetup-rsvp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: rsvp
      demo: online
  template:
    metadata:
      annotations:
        meetup: First Kubernetes & CNCF Online Meetup
      labels:
        app: rsvp
        demo: online
    spec:
      containers:
      - env:
        - name: TEXT1
          valueFrom:
            configMapKeyRef:
              key: TEXT1
              name: prod-rsvpconfig-staging-m42dtkmhfh
        - name: TEXT2
          valueFrom:
            configMapKeyRef:
              key: TEXT2
              name: prod-rsvpconfig-staging-m42dtkmhfh
        - name: MONGODB_HOST
          value: prod-meetup-mongodb
        image: nkhare/rsvpapp
        imagePullPolicy: Always
        livenessProbe:
          httpGet:
            path: /
            port: 5000
          initialDelaySeconds: 50
          periodSeconds: 30
          timeoutSeconds: 1
        name: rsvp-app
        ports:
        - containerPort: 5000
          name: web-port
---
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    meetup: First Kubernetes & CNCF Online Meetup
  labels:
    demo: online
  name: prod-meetup-rsvp-db
spec:
  replicas: 1
  selector:
    matchLabels:
      appdb: rsvpdb
      demo: online
  template:
    metadata:
      annotations:
        meetup: First Kubernetes & CNCF Online Meetup
      labels:
        appdb: rsvpdb
        demo: online
    spec:
      containers:
      - image: mongo:3.3
        name: rsvpd-db
        ports:
        - containerPort: 27017
```

### Deploy the application from the `prod` environment
```
$ kustomize build overlays/prod/ | kubectl apply -f -
```
```
configmap/prod-rsvpconfig-staging-m42dtkmhfh unchanged
service/prod-meetup-mongodb created
service/prod-meetup-rsvp created
deployment.apps/prod-meetup-rsvp created
deployment.apps/prod-meetup-rsvp-db created
```

```
$ kubectl get pods
```
```
NAME                                      READY   STATUS    RESTARTS   AGE
prod-meetup-rsvp-698dc496f7-7xmm4         1/1     Running   0          8m3s
prod-meetup-rsvp-698dc496f7-lmsxr         1/1     Running   0          8m3s
prod-meetup-rsvp-698dc496f7-vn9q2         1/1     Running   0          8m3s
prod-meetup-rsvp-db-7b59f95444-9zf9p      1/1     Running   0          8m3s
````

### Access the application from the `prod` environment
```
$ kubectl get svc
```
```
NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes            ClusterIP   10.96.0.1        <none>        443/TCP        19h
prod-meetup-mongodb   ClusterIP   10.110.199.178   <none>        27017/TCP      35s
prod-meetup-rsvp      NodePort    10.104.225.227   <none>        80:32185/TCP   35s
```
```
$ open http://localhost:32185
```

## Deploy the `staging` and `prod` application together. 

### Deploy the application from the `staging` environment
```
$ kustomize build overlays/staging/ | kubectl apply -f -
```
```
configmap/staging-rsvpconfig-staging-mkcm45999t created
service/staging-meetup-mongodb created
service/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp-db created
```

### Access the application from the `staging` environment
```
$ kubectl get svc
```
NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes               ClusterIP   10.96.0.1        <none>        443/TCP        19h
prod-meetup-mongodb      ClusterIP   10.110.199.178   <none>        27017/TCP      8m59s
prod-meetup-rsvp         NodePort    10.104.225.227   <none>        80:32185/TCP   8m59s
staging-meetup-mongodb   ClusterIP   10.96.202.31     <none>        27017/TCP      2m16s
staging-meetup-rsvp      NodePort    10.103.103.176   <none>        80:30498/TCP   2m16s
```
```
$ open http://localhost:30498
````

**To your surpise, as you refresh the page; you would see the `frontend` from the `prod` environment as well.**
This is because both of the `frontend` services(staging and prod) select the same set of `pods`.

```
$ kubectl describe svc staging-meetup-rsvp prod-meetup-rsvp
```
```
Name:                     staging-meetup-rsvp
Namespace:                default
Labels:                   app=rsvp
                          demo=online
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{"meetup":"First Kubernetes \u0026 CNCF Online Meetup"},"labels":{"app":"rsv...
                          meetup: First Kubernetes & CNCF Online Meetup
Selector:                 app=rsvp,demo=online
Type:                     NodePort
IP:                       10.103.103.176
LoadBalancer Ingress:     localhost
Port:                     <unset>  80/TCP
TargetPort:               web-port/TCP
NodePort:                 <unset>  30498/TCP
Endpoints:                10.1.1.129:5000,10.1.1.130:5000,10.1.1.131:5000 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

```
Name:                     prod-meetup-rsvp
Namespace:                default
Labels:                   app=rsvp
                          demo=online
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{"meetup":"First Kubernetes \u0026 CNCF Online Meetup"},"labels":{"app":"rsv...
                          meetup: First Kubernetes & CNCF Online Meetup
Selector:                 app=rsvp,demo=online
Type:                     NodePort
IP:                       10.104.225.227
LoadBalancer Ingress:     localhost
Port:                     <unset>  80/TCP
TargetPort:               web-port/TCP
NodePort:                 <unset>  32185/TCP
Endpoints:                10.1.1.129:5000,10.1.1.130:5000,10.1.1.131:5000 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

### Delete the applications
```
$ kustomize build overlays/staging/ |  kubectl delete -f -
```
```
configmap "staging-rsvpconfig-staging-mkcm45999t" deleted
service "staging-meetup-mongodb" deleted
service "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp-db" deleted
```
```
$ kustomize build overlays/prod |  kubectl delete -f -
```
```
configmap "prod-rsvpconfig-staging-m42dtkmhfh" deleted
service "prod-meetup-mongodb" deleted
service "prod-meetup-rsvp" deleted
deployment.apps "prod-meetup-rsvp" deleted
deployment.apps "prod-meetup-rsvp-db" deleted
$
```
