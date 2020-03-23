# kustomize-demo

## 7-kustomize-configmap

In the [application code](https://github.com/cloudyuga/rsvpapp/blob/master/rsvp.py), we defined some environment variables, which changes the text and look & feel of the application. 

```
LINK=os.environ.get('LINK', "www.cloudyuga.guru")
TEXT1=os.environ.get('TEXT1', "CloudYuga")
TEXT2=os.environ.get('TEXT2', "Garage RSVP")
LOGO=os.environ.get('LOGO', "https://raw.githubusercontent.com/cloudyuga/rsvpapp/master/static/cloudyuga.png")
COMPANY=os.environ.get('COMPANY', "CloudYuga Technology Pvt. Ltd.")
```

As we would deploy these apps for different environment/customers, we would like to change them at the time deployment. `Kustomize` has a feature called `configMapGenerator` using we create/update the `configMaps` at the time deployment.

### Update `overlays/staging/kustomization.yaml` file to add `configMapGenerator` section 

```
configMapGenerator:
- name: rsvpconfig-staging
  literals:
    - TEXT1="Welcome to"
    - TEXT2="Staging"
```

### Create a `patch` file `overlays/staging/staging-configmap.yaml` to update the `frontend deployment`, while doing the deployment

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
        - name: TEXT1
          valueFrom:
            configMapKeyRef:
              name: rsvpconfig-staging
              key: TEXT1
        - name: TEXT2
          valueFrom:
            configMapKeyRef:
              name: rsvpconfig-staging
              key: TEXT2
``` 

### Update the `overlays/staging/kustomization.yaml` to apply the patch. So the file looks something like following
```
$ cat overlays/staging/kustomization.yaml
```
```
bases:
- ../../base
namePrefix: staging-
patchesStrategicMerge:
- staging-configmap.yaml
configMapGenerator:
- name: rsvpconfig-staging
  literals:
    - TEXT1="Welcome to"
    - TEXT2="Staging"
```

### Build the `YAML` file

```
$ kustomize build overlays/staging/
```
```
apiVersion: v1
data:
  TEXT1: Welcome to
  TEXT2: Staging
kind: ConfigMap
metadata:
  name: staging-rsvpconfig-staging-9c2f82kg64
---
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
        - name: TEXT1
          valueFrom:
            configMapKeyRef:
              key: TEXT1
              name: staging-rsvpconfig-staging-9c2f82kg64
        - name: TEXT2
          valueFrom:
            configMapKeyRef:
              key: TEXT2
              name: staging-rsvpconfig-staging-9c2f82kg64
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
$ kustomize build overlays/staging/ | kubectl apply -f -
configmap/staging-rsvpconfig-staging-9c2f82kg64 configured
service/staging-meetup-mongodb created
service/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp-db created
```

### Access the application
```
$ kubectl get svc
```
```
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes               ClusterIP   10.96.0.1      <none>        443/TCP        18h
staging-meetup-mongodb   ClusterIP   10.97.181.62   <none>        27017/TCP      47s
staging-meetup-rsvp      NodePort    10.105.42.59   <none>        80:30146/TCP   46s
$ open http://localhost:30146
```

### Update the `configMap` 
You might have noticed additional hash value after the `configMap` name, `staging-rsvpconfig-staging-9c2f82kg64`. `Kustomizes` changes this everytime we update the `configMap`, which triggers the re-deployment of the application. So let's change `configMap` configuration in the `overlays/staging/kustomization.yaml` and re-deploy the application. 

```
bases:
- ../../base
namePrefix: staging-
patchesStrategicMerge:
- staging-configmap.yaml
configMapGenerator:
- name: rsvpconfig-staging
  literals:
    - TEXT1="Welcome to"
    - TEXT2="Updated Staging"
```
 
```
$ kustomize build overlays/staging/ | kubectl apply -f -
configmap/staging-rsvpconfig-staging-mkcm45999t created
service/staging-meetup-mongodb unchanged
service/staging-meetup-rsvp unchanged
deployment.apps/staging-meetup-rsvp configured
deployment.apps/staging-meetup-rsvp-db unchanged
```

### Access the update application
```
$ kubectl get svc
NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes               ClusterIP   10.96.0.1      <none>        443/TCP        18h
staging-meetup-mongodb   ClusterIP   10.97.181.62   <none>        27017/TCP      7m35s
staging-meetup-rsvp      NodePort    10.105.42.59   <none>        80:30146/TCP   7m34s
```

```
$ open http://localhost:30146
```

### Delete the application
```
$ kustomize build overlays/staging/ |  kubectl delete -f -
configmap "staging-rsvpconfig-staging-mkcm45999t" deleted
service "staging-meetup-mongodb" deleted
service "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp-db" deleted
``
