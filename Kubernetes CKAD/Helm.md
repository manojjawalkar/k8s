- Helm is like a package manager for k8s
![492473b7c6daf7d94b3ed42dcbc32586.png](../_resources/492473b7c6daf7d94b3ed42dcbc32586.png)

## Helm concept
- Helm is used as package manager for k8s
- It knows what all objects belong to a project and can help us in managing (In case you want to delete the project you would have to manually go check and delete every related object. This can be tedious and some objects can be left unnoticed)

## Helm chart
- Inside any definition file we can add jinja templates so that we can use the same definition file for multiple projects
- Helm chart = values.yaml + templates
- It also has cart.yaml which contains metadata about the chart itself

## Helm commands
- helm list
	- Find out how many helm repo are installed
		- `helm repo list`
- helm uninstall my-release
- `helm pull --untar bitnami/wordpress` ( only to download and not install)
- Command to search specific charts on Artifact Hub.
	- 	 `helm search hub chart-name` 
	- 	 Note: Replace the chart-name with the original name.
	- 	 `helm search repo joomla`

![25a91c4460fe4bcd3fed5a348266164e.png](../_resources/25a91c4460fe4bcd3fed5a348266164e.png)
![2d5cae1d8056ae550e7a5d8e42db47de.png](../_resources/2d5cae1d8056ae550e7a5d8e42db47de.png)
![76ea205ade594c6d7d46c618dd0f6154.png](../_resources/76ea205ade594c6d7d46c618dd0f6154.png)

## download a package and install afterwards
![4341de3b48b201ed232974b85d769cfc.png](../_resources/4341de3b48b201ed232974b85d769cfc.png)
![ac74cb17dc534333777697fc77a45600.png](../_resources/ac74cb17dc534333777697fc77a45600.png)
