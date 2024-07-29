## Selector

Selector is used to filter k8s objects based on their labels. For example, filtering based on label app=front-end

```
k get pods --selector app=front-end
```

While labels and selectors are used to group and select objects, annotations are used to record other details for informatory purpose. For example, tool details like name, version, build information, et cetera, or contact details, phone numbers, email ids, et cetera, that may be used for some kind of integration purpose.

Some examples

```
controlplane ~ ➜  k get pods --selector env=dev --no-headers | wc -l
7

controlplane ~ ➜  k get pods --selector env=finance --no-headers | wc -l
No resources found in default namespace.
0

controlplane ~ ➜  k get pods --selector bu=finance --no-headers | wc -l
6

controlplane ~ ➜  k get all --selector env=prod --no-headers | wc -l
7

controlplane ~ ➜  k get all --selector env=prod 
NAME              READY   STATUS    RESTARTS   AGE
pod/app-2-r9sgr   1/1     Running   0          2m39s
pod/auth          1/1     Running   0          2m38s
pod/app-1-zzxdf   1/1     Running   0          2m38s
pod/db-2-vxlql    1/1     Running   0          2m38s

NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/app-1   ClusterIP   10.43.252.14   <none>        3306/TCP   2m38s

NAME                    DESIRED   CURRENT   READY   AGE
replicaset.apps/app-2   1         1         1       2m39s
replicaset.apps/db-2    1         1         1       2m38s

controlplane ~ ➜  

controlplane ~ ➜  k get pods --selector env=prod bu=finance tier=frontend
error: name cannot be provided when a selector is specified

controlplane ~ ✖ k get pods --selector env=prod,bu=finance,tier=frontend
NAME          READY   STATUS    RESTARTS   AGE
app-1-zzxdf   1/1     Running   0          3m51s
 
```


## How does a service know which pod (or replica of pod) it has to manage?
- Service uses `selector` under spec where we can give labels that match the pods/deployment we want that service to manage.
- It is important that the selector must have all the matching labels specified for it to work. 
- If our deployment has more than one label then the service has to specify all those under the `selector` else it won't match any pods/deployments