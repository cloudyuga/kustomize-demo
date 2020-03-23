# kustomize-demo

## 6-kustomize-overlay

Here we have a `overlays` folder, in-parallel to the `base` folder. With-in that we have a our first overlay called `staging`. 

```
├── README.md
├── base
│   ├── backend
│   │   ├── backend-svc.yaml
│   │   ├── backend.yaml
│   │   └── kustomization.yaml
│   ├── frontend
│   │   ├── frontend-svc.yaml
│   │   ├── frontend.yaml
│   │   └── kustomization.yaml
│   ├── kustomization.yaml
│   └── patch.yaml
└── overlays
    └── staging
        └── kustomization.yaml
```


Inside the `staging` folder we have `kustomization.yaml` file whose content is as following:-

```
resources:
- ../../base
namePrefix: staging-
```

Here we refer everthing from base and just add `namePrefix` as `staging-` .

### Build the `YAML` file
```
$ kustomize build overlays/staging/
```
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    meetup: First Kubernetes & CNCF Online Meetup
  labels:
    app: rsvpdb
    demo: online
  name: staging-meetup-mongodb
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
  name: staging-meetup-rsvp
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
  name: staging-meetup-rsvp
spec:
  replicas: 1
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
        - name: MONGODB_HOST
          value: staging-meetup-mongodb
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
  name: staging-meetup-rsvp-db
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

### Deploy the application
```
$ kustomize build overlays/staging/ |  kubectl apply -f -
service/staging-meetup-mongodb created
service/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp-db created
```

### Access the application
```$ kubectl get svc
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP        18h
staging-meetup-mongodb   ClusterIP   10.109.19.219   <none>        27017/TCP      108s
staging-meetup-rsvp      NodePort    10.109.95.23    <none>        80:30351/TCP   107s
```

```
$ open http://localhost:30351
```

### Delete the application
```
$ kustomize build overlays/staging/ |  kubectl delete -f -
service "staging-meetup-mongodb" deleted
service "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp-db" deleted
```
