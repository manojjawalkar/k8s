# Policies
- Network policies are linked with one or more pods

## Use case
- By default, in k8s, all pods and services can communicate with each other.
- Considering our architeture in [Ingress and Egress traffic](../Kubernetes%20CKAD/Ingress%20and%20Egress%20traffic.md) we would like to restrict communication between web server and database server directly. 
- For this, we will use network policy on DB server pod to restrict communication fromm anything except traffic from the API server on port 3200.
	- Rule would look like - ***Only allow ingress traffic from API pod on 3200***

## Applying network policies to a pod
- We use labels and selectors
- We label a pod say `labels:\n\t role: db`
- We use the same label in network policy
	- There is a `podSelector.matchLabels` field in network policy under which the labels are defined.
- In our case we want to ***allow ingress traffic from API pod on port 3200***
- To do so, we specify `policyTypes: - Ingress`
- Next we specify more details like pods name and protocol and port.

networkPolicyDefinition.yaml
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
	    role: db
  policyTpyes:
    - Ingress
  ingress:
  - from: 
    - podSelector:
	    matchLabels:
		    name: api-pod
      namespaceSelector:
	    matchLabels:
		    env: production
	  ports:
	    - protocol: TCP
		  port: 3200
		

```

## Create a policy using create command
`kc create -f networkPolicyDefinition.yaml`


# NOTE: 
### Always refere to the network solutions documentation to see the support for the network policies.