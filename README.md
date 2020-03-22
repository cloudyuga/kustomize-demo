# kustomize-demo

## 3-kustomize-labels-annotation

### Add a label and annotation in `base's kustomization.yaml` file
```
$ cat base/kustomization.yaml
resources:
- backend
- frontend
commonLabels:
  demo: online
commonAnnotations:
  meetup: First Kubernetes & CNCF Online Meetup
```

### Build the YAML files with `kustomize build` command 
```
$ kustomize build base/
apiVersion: v1
kind: Service
metadata:
  annotations:
    meetup: First Kubernetes & CNCF Online Meetup
  labels:
    app: rsvpdb
    demo: online
  name: mongodb
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
  name: rsvp
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
  name: rsvp
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
          value: mongodb
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
  name: rsvp-db
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
$ kustomize build base/ | kubectl apply -f -
service/mongodb created
service/rsvp created
deployment.apps/rsvp created
deployment.apps/rsvp-db created
```

### Lock at the Labels

```
$ kubectl get deploy --show-labels
NAME      READY   UP-TO-DATE   AVAILABLE   AGE     LABELS
rsvp      1/1     1            1           4m29s   demo=online
rsvp-db   1/1     1            1           4m28s   demo=online
```

```
$ kubectl get pods --show-labels
NAME                       READY   STATUS    RESTARTS   AGE   LABELS
rsvp-5ff948df9d-9kr7v      1/1     Running   0          14s   app=rsvp,demo=online,pod-template-hash=5ff948df9d
rsvp-db-7b59f95444-hprf8   1/1     Running   0          14s   appdb=rsvpdb,demo=online,pod-template-hash=7b59f95444
```

### Access the app
```
$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        32m
mongodb      ClusterIP   10.98.57.104    <none>        27017/TCP      5m46s
rsvp         NodePort    10.99.194.245   <none>        80:32412/TCP   5m46s
``

```
$ open http://localhost:32412
```

### Delete the app
```
$ kustomize build base/ | kubectl delete -f -
service "mongodb" deleted
service "rsvp" deleted
deployment.apps "rsvp" deleted
deployment.apps "rsvp-db" deleted
```


