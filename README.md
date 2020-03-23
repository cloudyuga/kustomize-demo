# kustomize-demo

## 5-kustomize-patch

In the last section we saw that after adding `namePrefix`,`the frontend` could not connect to `the backend` because `backend's` name got changed. Somehow we need to make sure that in the frontend always to correct backend. For that we need to do two things.

### 1. Make the `MONGODB_HOST` environment as variable in the kustomization configuration. For that let us update the `base/kustomization.yaml` file and following:-

```
vars:
  - name: MONGODB_HOST
    objref:
      kind: Service
      name: mongodb
      apiVersion: v1
```

using which we are configuring `MONGODB_HOST` variable to always refer to the `backend's` service name. 

### 2. Once above is done, we need to update the `the frontend's` deployment configuration file such that it refer's to `the backend` via the variable we defined in the last step. For that we need to `patch` `the frontend's` deployment configuration file, as we should not modify original one. For that we'll do following:-
 
#### Create a `base/patch.yaml` file, with following content 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rsvp
spec:
  template:
    spec:
      containers:
      - name: rsvp-app
        env:
        - name: MONGODB_HOST
          value: $(MONGODB_HOST)
```

Using which we would be updating `the frontend's` deployment configuration file to refer the `MONGODB_HOST` environment variable with the variable we defined earlier. 

#### Update the `base/kustomization.yaml` with following to apply the patch 
```
patchesStrategicMerge:
- patch.yaml
```

So the `base/kustomization.yaml` file look's something like following:-

```
resources:
- backend
- frontend
commonLabels:
  demo: online
commonAnnotations:
  meetup: First Kubernetes & CNCF Online Meetup
namePrefix: meetup-
patchesStrategicMerge:
- patch.yaml
vars:
  - name: MONGODB_HOST
    objref:
      kind: Service
      name: mongodb
      apiVersion: v1
```


### Generate the final YANL configuration 

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
  name: meetup-mongodb
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
  name: meetup-rsvp
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
  name: meetup-rsvp
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
          value: meetup-mongodb
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
  name: meetup-rsvp-db
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
service/meetup-mongodb created
service/meetup-rsvp created
deployment.apps/meetup-rsvp created
deployment.apps/meetup-rsvp-db created
```

### Access the application
```
$ kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        14h
meetup-mongodb   ClusterIP   10.100.113.204   <none>        27017/TCP      26s
meetup-rsvp      NodePort    10.105.119.97    <none>        80:30390/TCP   25s
```
```
$ open http://localhost:30390
```

### Delete the application
```
kustomize build base/ | kubectl delete -f -
```

