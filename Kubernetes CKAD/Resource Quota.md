**Used to limit a namespace on how much resource it can use**

## Create a resource quota
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
	name: computeQuota
	namespace: dev
spec:
	hard:
		pods: "10"
		requests.cpu: "4"
		requests.memory: 5Gi
		limits.cpu: "10"
		limits.memory: 10Gi
```