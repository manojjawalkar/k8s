A way to store sensitive data

# Creating a secret
## Imperitive way
**From literal**
```
kc create secret generic <secret_name> --from-literal=DB_HOST=mysql --from-literal=DB_USER=root --from-literal=DB_PASSWORD=my_passwd
```
**From file**
```
kc create secret generic app-secret --from-file=app-secret.properties
```

## Declarative way
```yaml
apiVersion: v1
kind: Secret
metadata:
	name: myapp-secret
data:
	DB_HOST: mysql
	DB_USER: root 
	DB_PASSWORD: my_passwd
```
`kc create secret -f myap-secret-definition.yaml`

NOTE: But we don't store values in plain text in the definition file. We encode them in base64 
```yaml
apiVersion: v1
kind: Secret
metadata:
	name: myapp-secret
data:
	DB_HOST: bXlzcWw=
	DB_USER: dmFpc2huYXZp 
	DB_PASSWORD: bWFub2o=
```
* * *
## Commands
```
kc get secret myapp-secret
kc describe secret myapp-secret
# To view the values  
kc describe secret myapp-secret -o yaml
```

* * *
## Encoding and decoding base64
### Encoding
```
echo -n "mysql" | base64
```

### Decoding
```
echo -n "bXlzcWw=" | base64 --decode
```

# Injecting a Secret into a Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
	containers:
		- name: mysql-db
		  image: mysql
		  envFrom: 
		  	- secretRef:
					name: myapp-secret
```

`envFrom` is a list where we can pass multiple secrets. The above method creates a pod with all the data inside the secret available as environment variable.  

## Injecting a single env variable from a secret
```yaml
.
.
spec:
	containers:
		- name: myapp-backend
		  image: mysql
		  env:
		  	- name: myapp-secret
	  		  key: DB_HOST
.
.
```

## Injecting a secret as a volume
```yaml
.
.
spec:
	containers:
		- name: myapp-backend
		  image: mysql
volumes:
	- name: app-secret-volume
	  secret:
	  	secretName: myapp-secret
.
.
```
**NOTE**: *If you were to mount the Secret as a volume in the Pod, each attribute in the Secret is created as a file with the value of the Secret as its content. In this case, since we have three attributes in our Secret, three files are created. And if we look at the contents of the DB_Password file, we see the paswrd in it. So, here are some things to keep in mind when working with Secrets. First of all, note that Secrets are not encrypted. They're only encoded, meaning, anyone can look up the file that you created for Secrets, or get the Secret object and then decode it using the methods that we discussed before, to see the confidential data. So, remember, not to check-in your Secret definition files along with your code when you push to GitHub or something.*

# Viewing a secret 
**1. Using Yaml output**
	 You can check the value of the secret by various ways. One of them is using the yaml output flag
```
manojjawalkar ~ kc get secret my-secret
NAME        TYPE     DATA   AGE
my-secret   Opaque   2      2m35s

manojjawalkar ~ ➜  kc get secret my-secret -o yaml
apiVersion: v1
data:
  DB_Host: bXlzcWw=
  DB_Password: cGFzc3dk
kind: Secret
metadata:
  creationTimestamp: "2024-01-09T05:15:25Z"
  name: my-secret
  namespace: default
  resourceVersion: "910"
  uid: bb38802d-1479-4fdb-9287-2e62d7866814
type: Opaque

manojjawalkar ~ ➜  echo -n "bXlzcWw=" | base64 -d
mysql
manojjawalkar ~ ➜  echo -n "cGFzc3dk" | base64 -d
passwd
```

**2. Sneaking into ETCD server**
	So the secrets are stored onto the etcd server. The path to the secrets is `/registry/secrets/default/<YOUR_SECRET>`. The etcd is running as a Pod so you can either exec into the Pod and check the secrets or, if you are on the control plane, you need to install the `etcdctl` cmd line tool (`apt-get install etcd-client`). 
```
controlplane $ ETCDCTL_API=3 etcdctl \
>    --cacert=/etc/kubernetes/pki/etcd/ca.crt   \
>    --cert=/etc/kubernetes/pki/etcd/server.crt \
>    --key=/etc/kubernetes/pki/etcd/server.key  \
>    get /registry/secrets/default/my-secret
/registry/secrets/default/my-secret
k8s


v1Secret

        my-secretdefault"*$2cc22cd3-5744-42ac-805e-4b137096a1532wB
kubectl-createUpdatevFieldsV1:C
A{"f:data":{".":{},"f:DB_Host":{},"f:DB_Password":{}},"f:type":{}}B
DB_Hostmysql

DB_PasswordpasswdOpaque"
controlplane $
``` 
	

## The way kubernetes handles secrets 
1. A secret is only sent to a node if a pod on that node requires it.
2. Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
3. Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.