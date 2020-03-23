# kustomize-demo

## 9-putting-it-all-together-fixing-labels

To fix the `Label/Selector` issue we have seen in the last section, we need to add a unique label for `staging` and `prod` overlay. 

### Add a `env=staging` `Label`, while deploying app in `staging` environment
```
$ cat overlays/staging/kustomization.yaml
```
```
resources:
- ../../base
namePrefix: staging-
commonLabels:
  env: staging
patchesStrategicMerge:
- staging-configmap.yaml
configMapGenerator:
- name: rsvpconfig-staging
  literals:
    - TEXT1="Welcome to"
    - TEXT2="Staging"
```

### Add a `env=prod` `Label`, while deploying app in `prod` environment
```
$ cat overlays/prod/kustomization.yaml
```
```
resources:
- ../../base
namePrefix: prod-
commonLabels:
  env: prod
patchesStrategicMerge:
- staging-configmap.yaml
- replica-count.yaml
configMapGenerator:
- name: rsvpconfig-staging
  literals:
    - TEXT1="Welcome to"
    - TEXT2="Production"
```

### Deploy the applications for `staging` and `prod` environments
```
$ kustomize build overlays/staging/ | kubectl apply -f -
```
```
configmap/staging-rsvpconfig-staging-9c2f82kg64 configured
service/staging-meetup-mongodb created
service/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp created
deployment.apps/staging-meetup-rsvp-db created
```
```
$ kustomize build overlays/prod/ | kubectl apply -f -
```
```
configmap/prod-rsvpconfig-staging-m42dtkmhfh created
service/prod-meetup-mongodb created
service/prod-meetup-rsvp created
deployment.apps/prod-meetup-rsvp created
deployment.apps/prod-meetup-rsvp-db created
```

### Access the applications
```
$ kubectl get svc
```
```
NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes               ClusterIP   10.96.0.1       <none>        443/TCP        20h
prod-meetup-mongodb      ClusterIP   10.97.226.218   <none>        27017/TCP      58s
prod-meetup-rsvp         NodePort    10.100.42.85    <none>        80:31761/TCP   57s
staging-meetup-mongodb   ClusterIP   10.103.245.73   <none>        27017/TCP      65s
staging-meetup-rsvp      NodePort    10.107.253.69   <none>        80:30627/TCP   64s
```

```
open http://localhost:30627
```

```
open http://localhost:31761
```

### Delete the applications
```
$ kustomize build overlays/prod |  kubectl delete -f -
```
```
configmap "prod-rsvpconfig-staging-m42dtkmhfh" deleted
service "prod-meetup-mongodb" deleted
service "prod-meetup-rsvp" deleted
deployment.apps "prod-meetup-rsvp" deleted
deployment.apps "prod-meetup-rsvp-db" deleted
```
```
$ kustomize build overlays/staging |  kubectl delete -f -
```
```
configmap "staging-rsvpconfig-staging-9c2f82kg64" deleted
service "staging-meetup-mongodb" deleted
service "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp" deleted
deployment.apps "staging-meetup-rsvp-db" deleted
```
