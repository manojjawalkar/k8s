## env
- Allows you to set environment variables for a container, specifying a value directly for each variable that you name.

## envFrom
- Allows you to set environment variables for a container by referencing either a ConfigMap or a Secret. When you use envFrom, all the key-value pairs in the referenced ConfigMap or Secret are set as environment variables for the container. 
- You can also specify a common prefix string.

**NOTE**: ***The environment variables set using the env or envFrom field override any environment variables specified in the container image**.*

## Using env in pod definition
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: envar-demo
  labels:
    purpose: demonstrate-envars
spec:
  containers:
  - name: envar-demo-container
    image: gcr.io/google-samples/node-hello:1.0
    env:
    - name: DEMO_GREETING
      value: "Hello from the environment"
    - name: DEMO_FAREWELL
      value: "Such a sweet sorrow"
```

### Create a Pod based on that manifest:

`kubectl apply -f https://k8s.io/examples/pods/inject/envars.yaml`

### List the Pod's container environment variables:

`kubectl exec envar-demo -- printenv`

The output is similar to this:
```
NODE_VERSION=4.4.2
EXAMPLE_SERVICE_PORT_8080_TCP_ADDR=10.3.245.237
HOSTNAME=envar-demo
...
DEMO_GREETING=Hello from the environment
DEMO_FAREWELL=Such a sweet sorrow
```

