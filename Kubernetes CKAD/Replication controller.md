## Purpose

- Provides HA
- Ensures specified number of PODs in a running at all time
- Load balancing across nodes

## Replication Controller and Replica Set

Both the terms have same purpose. Replication controller is the older technology that is being replaced by replica set. Replica set is the new recommended way to set up replication

## Creating a Replication Controller

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
   name: myapp-rc
   lables:
      app: myapp
      type: front-end
spec:
   template:
      	metadata:
      	   name: myapp-pod
      	      lables:
      	        app: myapp
      	        type: front-end
      	spec:
      	   containers:
      	      - name: nginx-container
      	        image: nginx
	replicas: 2
	selector: 
		app: myapp
```

### Creating and checking pods with Replication Controller
	1. kubectl create -f replication-controller-definition.yaml
	2. kubectl get replicationcontroller
	3. kubectl get pods
	
## Creating a Replica Set

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
	name: myapp-replicaset
	lables:
		app: myapp
		type: front-end
spec:
	template:
		metadata: 
			name: myapp-pod
			lables: 
				app: myapp
				type: front-end
		spec:
			containers:
				- name: nginx-container
				  image: nginx
	replicas: 3
	selector:
		matchLables:
			type: front-end
```

### Creating and checking pods with ReplicaSet
	1. kubectl create -f replicaSet-definition.yaml
	2. kubectl get replicaset
	3. kubectl get pods
	
We use `selector` here because replicaset can not only manage pods created using the definition file but can also manage the existing pods if the label matches.
* * *
## Scaling Replicas
We have three options
1. Update the replicaSet-definition.yaml file and replace with below command
	1. `kubectl replace -f replicaSet-definition.yaml`
2. Set the number of replicas for the definition file
	1. `kubectl scale --replicas=6 replicaSet-definition.yaml`
3. Set the number of replicas using replicaset name
	1. `kubectl scale --replicas=6 replicaset myapp-replicaset`

**Note**: *The using option 2 the number of replicas is not updated in the definition file. It will still be showing as 3 in the file*

* * *
## Commands:
	1. kubectl create -f replicaset-definition.yaml
	2. kubectl get replicaset
	3. kubectl replace -f replicaset-definition.yaml
	4. kubectl delete replicaset myapp-replicaset
	5. kubectl scale --replicas=6 replicaset-definition.yaml