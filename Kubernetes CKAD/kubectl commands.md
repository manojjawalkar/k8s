# Get all k8s objects 
`kubectl get all`
# Create a new POD with an image

`kubectl run nginx --image=nginx`

# Get list of pods

`kubectl get pods`

# Get metadata about a pod

```
1. kubectl describe pod <POD_NAME>
2. kubectl describe pod
```

# Check which nodes the pods are placed on

```bash
1. kubectl describe pod <POD_NAME> | egrep "Name|Node"
2. kubectl describe pod newpods | egrep "Name|Node"
3. kubectl get pods -o wide
```

# Check number of containers inside a pod

```bash
Run kubectl describe pod <POD_NAME> and look under the Containers section
1. kubectl describe pod <POD_NAME>
```

## Sample output snippet

```
.
.
Containers:
  nginx:
    Container ID:   containerd://b5c3f7a35825f1fa91d6e85e769321b585d3d1f34a7a37adedcc88bffb1acf73
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:2bdc49f2f8ae8d8dc50ed00f2ee56d00385c6f8bc8a8b320d0a294d9e3b49026
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 03 Jan 2024 09:25:00 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-n8l8h (ro)
  agentx:
    Container ID:   
    Image:          agentx
    Image ID:       
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
.
.

```

We can check image the container is built from, state of the container, and many other things. Further below at the end we see events which tells you the logs of the containers.

## Check the status of each container indside pod

```
➜  kubectl get pods
NAME            READY   STATUS             RESTARTS        AGE
nginx           1/1     Running            0               53m
newpods-9qvcp   1/1     Running            3 (3m20s ago)   53m
newpods-8pmqt   1/1     Running            3 (3m20s ago)   53m
newpods-7tt77   1/1     Running            3 (3m20s ago)   53m
webapp          1/2     ImagePullBackOff   0               38m
```

The pod webapp has 2 containers but only 1 is in running state. So READY tells us information about running_containers_in_pod/total_containers_in_pod

# Create an nginx pod and expose it to port 8080
```
kc run custom-nginx --image=nginx --port=8080
```
# Delete a pod

```
1. kubectl delete pod webapp
```

# Use the kubectl run command to create a pod definition YAML file

```
~ kubectl run redis --image=redis123 --dry-run=client -o yaml > redis-definition.yaml

~ ls
redis-definition.yaml  sample.yaml

~ cat redis-definition.yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: redis
  name: redis
spec:
  containers:
  - image: redis123
    name: redis
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

# Create a resource from the manifest file

```
1. kubectl create -f redis-definition.yaml 
```

# Edit pod definition and re-run

```
1. kubectl edit pod redis
2. kubectl apply -f redis-definition.yaml
```

**A Note on Editing Existing Pods**  
In any of the practical quizzes, if you are asked to edit an existing POD, please note the following:

- If you are given a pod definition file, edit that file and use it to create a new pod.
- If you are not given a pod definition file, you may extract the definition to a file using the below command:
    - kubectl get pod <pod-name>\-o yaml > pod-definition.yaml</pod-name>
    - Then edit the file to make the necessary changes, delete, and re-create the pod.
- To modify the properties of the pod, you can utilize the kubectl edit pod <pod-name>command. Please note that only the properties listed below are editable.
    
    1.  spec.containers\[\*\].image
    2.  spec.initContainers\[\*\].image
    3.  spec.activeDeadlineSeconds
    4.  spec.tolerations
    5.  spec.terminationGracePeriodSeconds
    
    </pod-name>

* * *

# ReplicaSet

```
1. kubectl create -f replicaset-definition.yaml
2. kubectl get replicaset
3. kubectl scale --replicas=6 -f replicaset-definition.yaml
4. kubectl replace -f replicaset-definition.yaml
5. kubectl delete replicaset myapp-replicaset
```

replace is use to update the replicaset

# Deployment

```
1. kubectl create -f deployment-definition.yaml
2. kubectl get deployment
```

## - Create a new deployment called redis-deploy in the dev-ns namespace with the redis image. It should have 2 replicas

 ```
 kc create deployment redis-deploy -n=dev-ns --image=redis --replicas=2
 ```
 
 ## Deploy a redis pod using the redis:alpine image with the labels set to tier=db
 ```
 kc run redis --image=redis:alpine --labels="tier=db"
 ```
 
 ## Check the user the Pod is configured with
 ```
 kc exec my-app -- whoami
 ```
 
 ## Get a shell to the running Container:
```
kubectl exec -it my-app -- sh
```

## Checking the status of Pod:
### The reason for failure
```
kc describe pod mypod | grep -C3 "Last State:"
```