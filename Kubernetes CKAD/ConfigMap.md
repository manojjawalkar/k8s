## Config Map definition
```yaml
apiVersion: v1
kind: ConfigMap
metdata: 
	name: game_demo
data:
	# property like keys
	initial_player_lives: "3"
	initial_weapon: "Maverick"
	ui_properties_file_name: "user-interface.properties"
	# File like keys
	game.properties: |
		enemy.types=aliens,monsters
		player.max.lives=5
	user-interface.properties: |
		color.good=green
		color.bad=red
		allow.textmode=true
```
		
* * *
## Inserting values into pod definition from config maps

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: myapp-pod
spec:
  containers:
  - name: weapp-color
    image: kodekloud/webapp-color 
  	 env:
	 		- name: PLAYER_INITIAL_LIVES
			 	valueFrom:
					configMapKeyRef:
						- name: game_demo
							key: initial_player_lives
			- name: INITIAL_WEAPON
				valueFrom:
					configMapKeyRef:
						- name: game_demo
							key: initial_weapon
```

* * *

## Commands 
```
kc get configmap
kc get cm
```

```
# kubectl describe cm db-config
Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_HOST:
----
SQL01.example.com
DB_NAME:
----
SQL01
DB_PORT:
----
3306

BinaryData
====

Events:  <none>
```

