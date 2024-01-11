# Formatting Output with kubectl
The default output format for all kubectl commands is the human-readable plain-text format.

The `-o` flag allows us to output the details in several different formats.



`kubectl [command] [TYPE] [NAME] -o <output_format>`
* * *
**Here are some of the commonly used formats:**



`-o json` Output a JSON formatted API object.

`-o name` Print only the resource name and nothing else.

`-o wide` Output in the plain-text format with any additional information.

`-o yaml` Output a YAML formatted API object.


**Examples**:
```
kubectl get deployment httpd-frontend -o yaml > http-deployment-definition.yaml
kubectl get replicaset httpd-frontend-d474dcb49 -o yaml > replicaset-definition.yaml
kubectl get replicaset httpd-frontend-d474dcb49 -o json
```
```bash
# kubectl get replicaset 
NAME                       DESIRED   CURRENT   READY   AGE
httpd-frontend-d474dcb49   3         3         3       12m

# kubectl get replicaset -o wide
NAME                       DESIRED   CURRENT   READY   AGE   CONTAINERS       IMAGES             SELECTOR
httpd-frontend-d474dcb49   3         3         3       11m   httpd-frontend   httpd:2.4-alpine   name=httpd-frontend,pod-template-hash=d474dcb49

# kubectl get deployment
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
httpd-frontend   3/3     3            3           13m

# kubectl get deployment -o wide
NAME             READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS       IMAGES             SELECTOR
httpd-frontend   3/3     3            3           13m   httpd-frontend   httpd:2.4-alpine   name=httpd-frontend

```
