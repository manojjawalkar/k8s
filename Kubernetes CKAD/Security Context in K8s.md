## At Pod level
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	securityContext:
		runAsUser: 1000
		runAsGroup: 3000
		fsGroup: 2000
  containers:
  - name: weapp-color
    image: kodekloud/webapp-color 
```

- In the configuration file, the runAsUser field specifies that for any Containers in the Pod, all processes run with user ID 1000. The runAsGroup field specifies the primary group ID of 3000 for all processes within any containers of the Pod. If this field is omitted, the primary group ID of the containers will be root(0). Any files created will also be owned by user 1000 and group 3000 when runAsGroup is specified. Since fsGroup field is specified, all processes of the container are also part of the supplementary group ID 2000. The owner for volume /data/demo and any files created in that volume will be Group ID 2000

## At Container level

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
  containers:
  - name: weapp-color
    image: kodekloud/webapp-color
		securityContext:
			runAsUser: 1000
			capabilities:
				add: ["MAC_ADMIN"]
```

***NOTE: *`Capabilities`* are ONLY added at the container level***


You may choose to configure the security settings at a container level, or at a pod level. If you configure it at a **pod level, the settings will carry-over to all the containers within the pod**. If you configure it at both the pod and the container, **the settings on the container will override the settings on the pod**.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  -  image: ubuntu
     name: web
     command: ["sleep", "5000"]
     securityContext:
      runAsUser: 1002

  -  image: ubuntu
     name: sidecar
     command: ["sleep", "5000"]
```

* * *
**Reference**: 
[1] https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
