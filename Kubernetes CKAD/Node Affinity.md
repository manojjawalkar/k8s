## Types
Node Affinity can be of two types
1. requiredDuringSchedulingIgnoredDuringExecution
2. preferredDuringSchedulingIgnoredDuringExecution

##  requiredDuringSchedulingIgnoredDuringExecution
**Scheduling** - When the pod is created for the first time
 - If the matching label is found, the pod will be scheduled to run on that node
 - If the matching label is not found, the pod is not scheduled anywhere

**Execution** - when the pod is already scheduled and someone changed the label on the nodes
- If the matching label is not found, the pod will keep running on the same node
- If the matching label is found on some other node, the pod will be evicted from the current node and scheduled on the node that matches the label. 

## preferredDuringSchedulingIgnoredDuringExecution
**Scheduling**
-  If the matching label is found, the pod will be scheduled to run on that node
- If the matching label is not found, the pod is placed on any of the available nodes

**Execution**
- If the matching label is not found, the pod will keep running on the same node
- If the matching label is found on some other node, the pod will be evicted from the current node and scheduled on the node that matches the label. 

There is one more type that is planned but not yet used
## requiredDuringSchedulingRequiredDuringExecution
The pod will be scheduled or run only and only if the node matches the label.


## Node Affinity in a pod
### size= large_or_medium
```yaml
apiVersion: v1
kind: Pod
metadata: 
	name: myapp-pod
specs:
	containers:
		- name: myapp
			image: nginx
	affinity:
		nodeAffinity:
			requiredDuringSchedulingIgnoredDuringExecution:
				nodeSelectorTerms:
					- matchExpressions:
						- key: size
							operator: in
							values:
								- large
								  medium
								  
```

### size = not_small
```yaml
...
		affinity:
			nodeAffinity:
				requiredDuringSchedulingIgnoredDuringExecution:
					nodeSelectorTerms:
						- matchExpressions:
							- key: size
								operator: NotIn
								values:
									- small
...
```

### Only if the label exists
Here the pod will be scheduled only when the label size is present on the node irrespective of the value of the label size (i.e, small, medium, large) it will be scheduled and run there
```yaml
...
		affinity:
			nodeAffinity:
				requiredDuringSchedulingIgnoredDuringExecution:
					nodeSelectorTerms:
						- matchExpressions:
							- key: size
								operator: Exists
...
```


## NOTE:
- The operator is case sensitive. `Exists` 
