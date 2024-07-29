- There are two types of accounts in k8s
    - User account
    - Service account
- User account is used by humans to carry out tasks for eg. an admin or a developer
- A Service account is used by an application like jenkins or prometheous

A service account is a type of non-human account that, in Kubernetes, provides a distinct identity in a Kubernetes cluster. Application Pods, system components, and entities inside and outside the cluster can use a specific ServiceAccount's credentials to identify as that ServiceAccount. This identity is useful in various situations, including authenticating to the API server or implementing identity-based security policies.

## Creating a SA

NOTE: In the K8s version before 1.24, every time we would create a service account, a non-expiring secret token (Mountable secrets & Tokens) was created by default. However, from version 1.24 onwards, it was disbanded and **NO secret token is created by default** when we create a service account

```bash
$ k create serviceaccount dashboard-sa
serviceaccount/dashboard-sa created

$ k get sa
NAME           SECRETS   AGE
dashboard-sa   0         6s

$ k get sa dashboard-sa -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "2024-01-10T12:25:28Z"
  name: dashboard-sa
  namespace: default
  resourceVersion: "2404"
  uid: 3bde7491-2ac0-43f9-9617-c05a85d43ed3

controlplane $ k describe sa dashboard-sa   
Name:                dashboard-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
```

You can see it doesn't create a secret and token because in version 1.24 onwards it was removed. So we need to create a token manually with the name of the SA that we want to associate it with

```
controlplane $ k create token dashboard-sa
eyJhbGciOiJSUzI1NiIsImtpZCI6IkowZVA4T2w0MXVuLWpDcXFiZXNYRmt1OEZDYUtRMjdHWlU1bFlCSUdBZjQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzA0ODk1NTQyLCJpYXQiOjE3MDQ4OTE5NDIsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRhc2hib2FyZC1zYSIsInVpZCI6IjNiZGU3NDkxLTJhYzAtNDNmOS05NjE3LWMwNWE4NWQ0M2VkMyJ9fSwibmJmIjoxNzA0ODkxOTQyLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.bZ3s38Kg4Qfum0K1Qz1d6QSZOumEE9O2r_WtwgOnLMG8uxph52w5auv-2WZCWCBjaCxCS9GNL4O0V6h5xjMCzdW2ptq8sFV-wiDAZUQe-JkucmYG5lhsCPDNWLcJJQsPMGQ3V27PqAOaL35HipCAqR7Q1JCsXaS6hFIuV6dwwdJvT1a9dajX02UFKtSSLswjPxfm9nTlX1RlIigT1jK61xVdDPrhFtjoYCdmIy1131BgMCNql5oIK-OlAUdS6LIOVSO66cVKFGmuag0w5NkZObGSc101CROfTBSrV5oNKD2VBU28LlzdOg-tgYQWU5HHrDX4CbCjbuDp8263UF5VTw
```

## Adding a SA to an existing Pod

NOTE: You cannot edit the SA of an existing Pod. You must delete and recreate  
NOTE: For Deployment objects we can edit the SA

**Editing Deployment**

Use the `kubectl edit` command for the deployment and specify the `serviceAccountName` field inside the pod spec. Or make use of the `kubectl set` command. Run the following command to use the newly created service account: - `kubectl set serviceaccount deploy/web-dashboard dashboard-sa`

OR

- Save the deployment yaml
    - `k get deployment web-dashboard -o yaml > web-dashboard-deployment.yaml`
- Edit to add the serviceAccountName: &lt;SA_NAME&gt; inside the deployments containers' spec
- “containerPort” defines the port on which app can be reached out inside the container.

```yaml
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      name: web-dashboard
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        name: web-dashboard
    spec:
      containers:
      - env:
        - name: PYTHONUNBUFFERED
          value: "1"
        image: gcr.io/kodekloud/customimage/my-kubernetes-dashboard
        imagePullPolicy: Always
        name: web-dashboard
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      serviceAccountName: dashboard-sa
    ...
```

- delete teh deployment
    - controlplane ~ ➜ k delete deployment web-dashboard
- recreate the deployment
    - controlplane ~ ➜ k apply -f web-dashboard-deployment.yaml

**Editing a Pod**

```
...
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx-app
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-n7fmj
      readOnly: true
  dnsPolicy: ClusterFirst
  serviceAccountName: dashboard-sa
  ...
```

If the pod is created in declarative way - Just recreate it

```
controlplane $ k apply -f pod.yaml  
```

Else delete thw pod first

```
- controlplane $ k delete pod nginx-app
pod "nginx-app" deleted

controlplane $ k create -f pod.yaml
```

* * *

**NOTE:** So remember, Kubernetes automatically mounts the default service account if you haven't explicitly specified any. You may choose **not** to mount a service account automatically by setting the automountserviceaccount token field to false in the pods spec section.

Example -

```
controlplane $ k create sa dashboard-sa
serviceaccount/dashboard-sa created
controlplane $ k describe sa dashboard-sa
Name:                dashboard-sa
Namespace:           default
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
controlplane $ 
controlplane $ k create token dashboard-sa
eyJhbGciOiJSUzI1NiIsImtpZCI6IkowZVA4T2w0MXVuLWpDcXFiZXNYRmt1OEZDYUtRMjdHWlU1bFlCSUdBZjQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzA0ODk2NzczLCJpYXQiOjE3MDQ4OTMxNzMsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0Iiwic2VydmljZWFjY291bnQiOnsibmFtZSI6ImRhc2hib2FyZC1zYSIsInVpZCI6IjQ2NGJkNjIxLTZmOTUtNGI5YS04ZTBkLTNlZTRmMGZhNDRlNSJ9fSwibmJmIjoxNzA0ODkzMTczLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVmYXVsdDpkYXNoYm9hcmQtc2EifQ.MAjjeTDDY9CHZ0G71zr7_NMpLyCGebJCmltU70An-VTHPTjYClkTfR-re79u9f78l3xrxN3LQPF9744DjHy0Gp5gfRHQtK141P36dTM3TStEDm8nObbWvZ3E1jKX6udeanGBAXbe_aFrjy1p-xfTcpMnUJxtzTh0h1x2xblkmk4zZ1kPo6tztAY1d2g97E_hxnFirJSBSKyMc3RHtxSn_GYK5Q6uvd2d-acdzQLtJ2alZ9GqWpYSGYwdrZTngOHMb6fS5wiiF6mNzZb09aaWnSRPnQi9u4ZdnJmPJzxl2LVWC44pL8IZR8zxB1ANZXvvSF5shdW9h0ztIINDcb_fWg
controlplane $ 
controlplane $ k run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
controlplane $ k get pods
No resources found in default namespace.
controlplane $ less pod.yaml 
controlplane $ 
controlplane $ vim pod.yaml 
controlplane $ k apply -f pod.yaml 
pod/nginx created
controlplane $ k get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          17s
controlplane $ k get pods nginx -o yaml | grep serviceAccount
      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"creationTimestamp":null,"labels":{"run":"nginx"},"name":"nginx","namespace":"default"},"spec":{"containers":[{"image":"nginx","name":"nginx","resources":{}}],"dnsPolicy":"ClusterFirst","restartPolicy":"Always","serviceAccountName":"dashboard-sa"},"status":{}}
  serviceAccount: dashboard-sa
  serviceAccountName: dashboard-sa
      - serviceAccountToken:
controlplane $ k get pods nginx -o yaml | grep -A3 mount     
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: kube-api-access-9qgqt
      readOnly: true
  dnsPolicy: ClusterFirst
controlplane $ 
controlplane $ 
controlplane $ k exec -it nginx --ls /var/run/secrets/kubernetes.io/serviceaccount
error: unknown flag: --ls
See 'kubectl exec --help' for usage.
controlplane $ k exec -it nginx -- ls /var/run/secrets/kubernetes.io/serviceaccount
ca.crt  namespace  token
controlplane $ 
controlplane $ k exec -it nginx cat /var/run/secrets/kubernetes.io/serviceaccount/token
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl exec [POD] -- [COMMAND] instead.
eyJhbGciOiJSUzI1NiIsImtpZCI6IkowZVA4T2w0MXVuLWpDcXFiZXNYRmt1OEZDYUtRMjdHWlU1bFlCSUdBZjQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzM2NDI5Mjc2LCJpYXQiOjE3MDQ4OTMyNzYsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJuZ2lueCIsInVpZCI6IjY4NjUzYWI0LTBlOGUtNDQ5Mi04OTgxLTYzYWViMTJkOTNmYSJ9LCJzZXJ2aWNlYWNjb3VudCI6eyJuYW1lIjoiZGFzaGJvYXJkLXNhIiwidWlkIjoiNDY0YmQ2MjEtNmY5NS00YjlhLThlMGQtM2VlNGYwZmE0NGU1In0sIndhcm5hZnRlciI6MTcwNDg5Njg4M30sIm5iZiI6MTcwNDg5MzI3Niwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6ZGFzaGJvYXJkLXNhIn0.XzrY2UWi3gxvT1pZVjFu6OwGlCF6CurhZc1zt6dnzKf4nddDLePR1IFVWeKgj87KuOZ0fdpLgopBxLONE3rUvEqSaBLEsXBcmexcRBL2PpqWFidto4QjUnqS-eBDw9yjSf8FYOkzurAE-wLkAWZJleTr2eOfIlPzqGb6q30CyNyreTqFowhypiGJhtvsjTVVJB3yi4ct2whzM1e_RdTW14STR6AhJXrtUhxiF9Urb70ikSTt-AEGysv09Rqi_w6DRlrwsDFilw4etGKavUZClGro-uYca_9tZrKrz6pkV7UJZLKPc2D-d7Lcq9Y2L98Mko6Trx0xlHcIS5Os_31GmAcontrolplane $ 
```