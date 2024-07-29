# sidecar

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: app1
  name: app1
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
    ports:
      - containerPort: 8080
  - image: curlimages/curl
    name: curlpod
    command: ["/bin/sh"]
    args: ["-c","echo hello from sidecar; sleep 300"]
    ports:
      - containerPort: 8008
```

