# Liveness probe
- Just because the container is running doesn't mean the application is up
- Sometimes the application might misbehave due to some reason like a bug in the application
- We need a method to make sure the application is up at all the time
- This is where Liveness probe helps
- It periodically tests the application to check if it is in a healthy state
- The app developer decides what it means for an application to be in a healthy state
- If the liveness test fails, the container is considered unhealthy and is destroyes and recreated
- There are other options like adding initial delay `initialDelaySeconds` to add a delay before the readiness probe can start
- To specify interval to probe, or how often to probe, we have `periodSeconds`
- Ideally the container is destroyed and recreated if the readiness probe fails after 3 attempts. We can configure this value using `failureThreshold`

The liveness is configured under spec section of a pod
```
...
spec:
	livenessProbe:
		httpGet:
			path: /api/healthy
			port: 8080
		initialDelaySeconds: 10
		periodSeconds: 5
		failureThreshold: 6
...
```

Here, `httpGet` is the option we get for testing APIs, `tcpSocket` for ports, and `exec` for commands.

```
...
spec:
	livenessProbe:
		tcpSocket:
			port: 4321
		initialDelaySeconds: 8
		periodSeconds: 5
		failureThreshold: 9
...
```

```
...
spec:
	livenessProbe:
		exec:
			command: 
				- cat
				- /app/isHealthy
...
```
* * *
# Readiness Probe
- It is use to make sure the pod is up and running as soon as it is created
- By default, k8s marks a pod in a ready state as soon as it is scheduled on a node
- But an app can take time before it is completely up and running 
- For this we use readiniess probe
- Similar to Liveness probe, we get different test options for testing applications 
	- httpGet
	- tcpSocket
	- command
- There are other options like adding initial delay `initialDelaySeconds` to add a delay before the readiness probe can start
- To specify interval to probe, or how often to probe, we have `periodSeconds`
- Ideally the container is destroyed and recreated if the readiness probe fails after 3 attempts. We can configure this value using `failureThreshold`
- Readiness probe is also configured under spec section

```
...
spec:
	containers:
		readinessProbe:
			httpGet:
				path: /api/healthy
				port: 8080
			initialDelaySeconds: 10
			periodSeconds: 5
			failureThreshold: 6
...
```

```
...
spec:
	readinessProbe:
		tcpSocket:
			port: 4321
...
```

```
...
spec:
	readinessProbe:
		exec:
			command: 
				- cat
				- /app/isHealthy
		initialDelaySeconds: 10
		periodSeconds: 5
...
```

