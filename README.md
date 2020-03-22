# kustomize-demo

## 4-kustomize-namePrefix

### Update the `base's kustomization.yaml` file to add `namePrefix`
```
$ cat base/kustomization.yaml
resources:
- backend
- frontend
commonLabels:
  demo: online
commonAnnotations:
  meetup: First Kubernetes & CNCF Online Meetup
namePrefix: meetup-
```

### Deploy the application
```
$ kustomize build base/ | kubectl apply -f -
service/meetup-mongodb created
service/meetup-rsvp created
deployment.apps/meetup-rsvp created
deployment.apps/meetup-rsvp-db created
```
```
$ kubectl get deploy
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
meetup-rsvp      1/1     1            1           19s
meetup-rsvp-db   1/1     1            1           19s
```
```
$ kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        48m
meetup-mongodb   ClusterIP   10.108.108.110   <none>        27017/TCP      22s
meetup-rsvp      NodePort    10.108.148.145   <none>        80:30630/TCP   22s
```

### Access the application
```
$ open http://localhost:30630
```

Application access fails with following error on the browser
***pymongo.errors.ServerSelectionTimeoutError: mongodb:27017: [Errno -2] Name does not resolve***

If we list the pods, we see some restarts
```
$ kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
meetup-rsvp-5ff948df9d-qpwtg      1/1     Running   3          10m
meetup-rsvp-db-7b59f95444-ntk9h   1/1     Running   0          10m
```

And look the logs on the `Frontend Pod`
```
$ kubectl logs meetup-rsvp-5ff948df9d-qpwtg
....
....
Traceback (most recent call last):
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 2309, in __call__
    return self.wsgi_app(environ, start_response)
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 2295, in wsgi_app
    response = self.handle_exception(e)
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 1741, in handle_exception
    reraise(exc_type, exc_value, tb)
  File "/usr/local/lib/python3.6/site-packages/flask/_compat.py", line 35, in reraise
    raise value
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 2292, in wsgi_app
    response = self.full_dispatch_request()
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 1815, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 1718, in handle_user_exception
    reraise(exc_type, exc_value, tb)
  File "/usr/local/lib/python3.6/site-packages/flask/_compat.py", line 35, in reraise
    raise value
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 1813, in full_dispatch_request
    rv = self.dispatch_request()
  File "/usr/local/lib/python3.6/site-packages/flask/app.py", line 1799, in dispatch_request
    return self.view_functions[rule.endpoint](**req.view_args)
  File "/usr/src/app/rsvp.py", line 59, in rsvp
    items = [item for item in _items]

```

We are getting above because we are referring the Backend service name with `MONGODB_HOST` environment variable, which is currently set to `mongodb`
```
      containers:
      - name: rsvp-app
        image: nkhare/rsvpapp
        imagePullPolicy: Always
        env:
        - name: MONGODB_HOST
          value: mongodb
```

With our `namePrefix` setting, the Backend Service is currently named as `meetup-mongodb`

```
$ kubectl get svc
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP        66m
meetup-mongodb   ClusterIP   10.108.108.110   <none>        27017/TCP      18m
meetup-rsvp      NodePort    10.108.148.145   <none>        80:30630/TCP   18m
```

### Delete the application
```
$ kustomize build base/ | kubectl delete -f -
service "meetup-mongodb" deleted
service "meetup-rsvp" deleted
deployment.apps "meetup-rsvp" deleted
deployment.apps "meetup-rsvp-db" deleted
```
