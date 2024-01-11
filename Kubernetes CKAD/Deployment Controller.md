- Handles upgrading your Docker instances seamlessly as soon as the new build is available on the registry.
    
- Rolling updates - not all the pods are upgraded at once. They are upgraded one by one
    
- Each container is encapsulated in pods. Multiple such pods are deployed using replication controllers or replica sets. And then comes deployment, which is a Kubernetes object that comes higher in the hierarchy.  
    ![394673569ad9010d3baa837dbc2e50bc.png](../_resources/394673569ad9010d3baa837dbc2e50bc.png)
    
- Deployment is similar to a replicaset.
    
- Deployment creates a replicaset as well
    

* * *

**Deployment definition file**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd-frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      name: httpd-frontend
  template:
    metadata:
      labels:
        name: httpd-frontend
    spec:
      containers:
      - name: httpd-frontend
        image: httpd:2.4-alpine
```

![d622c3bb8a0b355f68154ce4207b3bc1.png](../_resources/d622c3bb8a0b355f68154ce4207b3bc1.png)

Output of get all comand -  
![f4ca2af07e62897c6860a5595a9c3576.png](../_resources/f4ca2af07e62897c6860a5595a9c3576.png)