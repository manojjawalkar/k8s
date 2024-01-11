## Checking if encryption at rest is enabled
- The kube-apiserver process accepts an argument --encryption-provider-config that controls how API data is encrypted in etcd. The configuration is provided as an API named EncryptionConfiguration
- ![5a21726a1d48ea6231070b8620c70a3e.png](../_resources/5a21726a1d48ea6231070b8620c70a3e.png)
```
controlplane $ ps -aux | grep kube-apiserver | grep encryption-provider-config
```

- In the above image you can see the option is not enabled.
- Another way is to check here 
```
controlplane $ cat /etc/kubernetes/manifests/kube-apiserver.yaml  | grep encryption-provider-config 
```

## Enabling encryption at rest
1. Create a configuration file
2. Pass the option as a parameter in that file

### Creating configuration file
[Definition file](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)
```yaml
apiVersion: v1
kind: EncryptioConfiguration
metadata:
	name: myEC
resources:
	- resources:
		- secrets
	  providers:
	  	- aesgcm:
			keys:
				- name: key1
				  secret: XUn3iq4F+eHLfdaKyCR6/QtqJCvYqMwt1hf/wC1OV3k=
				- name: key2
				  secret: ibSkuumCZikoIfVCd1BnAgN34dU5rtBguxyruqw421c=
		- identity: {} # this fallback allows reading unencrypted secrets;
                     # for example, during initial migration
```
				  
- The providers array is an ordered list of the possible encryption providers to use for the APIs that you listed. Each provider supports multiple keys - the keys are tried in order for decryption, and if the provider is the first provider, the first key is used for encryption.
- Check the [list of available providers](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#providers) 
- Keys here are not your environment variables, they are the encryption keys that you want to use to decrypt the data that is going to be encrypted using the same keys.
- To generate keys (usually 32 bits or more) use the below comand if on linux/mac
	- `% head -c 32 /dev/urandom | base64`
```bash
% head -c 32 /dev/urandom | base64
XUn3iq4F+eHLfdaKyCR6/QtqJCvYqMwt1hf/wC1OV3k=
```
- Now that you have the EncryptionConfiguration file ready, Set the `--encryption-provider-config` flag on the kube-apiserver to point to the location of the config file.
	- Save the EncryptionConfiguration file at `/etc/kubernetes/enc/myEncConf-definition.yaml` on the control plane node
	- Edit the manifest for the kube-apiserver static pod to add the `--encryption-provider-config flag`
		- `/etc/kubernetes/manifest/kube-apiserver.yaml`
```yaml
...
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # add this line
    volumeMounts:
    ...
```

- Restart your API server
* * *
## Ensure all Secrets are encrypted
Since Secrets are encrypted on write, performing an update on a Secret will encrypt that content.

```
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

The command above reads all Secrets and then updates them to apply server side encryption.



