Reference:

1.  [Basics of ctr command](https://labs.iximiuz.com/courses/containerd-cli/ctr/container-management#recap-what-is-ctr)
2.  Use the [Pod definition file](https://adil.medium.com/how-do-containers-communicate-via-localhost-in-a-kubernetes-pod-d9e193844b9d) here

# Pause Container

- A special container that **exists in every pod** that is created by containerD
- Not visible to kubectl
- **Minimal resource usage**
    - The pause container does not run any application code.
    - It only servs as a placeholder for namespaces (Network and IPC)

## Purpose

- Shares Inter process communication namespace between containers
- Provides shared network namespace
- Network isolation between pods
- Responsible for routing traffic between the pod and the outside world
- Along with network, ports are shared too
- Responsible for managing the lifecycle of a pod
    - When all other containers within the pod have completed their tasks and exited, the pause container remains running, effectively keeping the pod alive. This ensures that the resources allocated to the pod, such as network namespaces, are not prematurely released.
- Portability
    - The pause container is a standard component of Kubernetes, so it is available on all Kubernetes platforms. This means that you can deploy your applications to any Kubernetes cluster without having to worry about configuring networking

## Working

- Pause container creates a network namespace and shares it with all other containers (inside that pod only)
- Because of this, since the network stack is shared, containers can communicate with each others
- Since it also shares the IPC namespace, it allows all other containers to communicate with each other.
- The pause container starts, then goes to “sleep"

## Checking the pause container

- ssh into any node then run below command

```
node01 $ ctr -n k8s.io c ls | grep pause
3325bef282b6555ed86811f22b08199824c8a305d55f28480a5403e03fe8ad1d    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
646f0099701ceafabf9c0eb31b5845650e0aea60fd9df951722ce2b2d64bba60    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
88cebeb5db8ba95f16dd97ec428a809cbf57981f4bac946053ab184191ee54fb    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
9a42c73bf8a3e567dc100cef216e3842c01b4ff683c7990d6dea8e83e9831c8d    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
a82b4110822e1427e7a07fea354f5fa3686d070285dcdcbfc81f150c8c30843d    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
c7d6f76e09fbe7f8c153fb74d6197220b9a00d848003f11cd9d81b449ff2660c    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
e17c77fcc66c98abc895d7b6ed72f7774b72584c6b2ada47b3d3e3987e4ffb22    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
f81155092207c6198de8261843dd0dc661b64fec17ca1dd13201ef59973d7cff    registry.k8s.io/pause:3.5                  io.containerd.runc.v2    
fc9b56cf076facb41e347dd4a7f942faecb6bc83877686eb85f2993cb072b733    registry.k8s.io/pause:3.5                  io.containerd.runc.v2 
```

- When you create a new pod on this node, a new pause container is created. Identify the ID of that pause container and then run this command ( You might need to install jq command).

```
node01 $ ctr -n k8s.io c info c7d6f76e09fbe7f8c153fb74d6197220b9a00d848003f11cd9d81b449ff2660c | jq .Spec.hostname  
"mypod"
```

## Checking namespaces of two containers spawn in a single pod (use the pod definition file in reference)

1.  Get the container IDs

```
node01 $ ctr -n k8s.io c ls | grep nodejs
8ffa2f39c9c37adf81e8bddeaa3e2e7d60889017dfcf3154f92288d55df15505    docker.io/webratio/nodejs-http-server:latest    io.containerd.runc.v2    
a3e5642359b3ee34a37aceee282b853ed25fc966f422a1afe89ffa60a66494b5    docker.io/webratio/nodejs-http-server:latest    io.containerd.runc.v2    
```

2.  Check the info for both the containers

```
node01 $ ctr -n k8s.io c info 8ffa2f39c9c37adf81e8bddeaa3e2e7d60889017dfcf3154f92288d55df15505 | jq .Spec.linux.namespaces
[
  {
    "type": "pid"
  },
  {
    "type": "ipc",
    "path": "/proc/12012/ns/ipc"
  },
  {
    "type": "uts",
    "path": "/proc/12012/ns/uts"
  },
  {
    "type": "mount"
  },
  {
    "type": "network",
    "path": "/proc/12012/ns/net"
  }
]
node01 $ ctr -n k8s.io c info a3e5642359b3ee34a37aceee282b853ed25fc966f422a1afe89ffa60a66494b5 | jq .Spec.linux.namespaces
[
  {
    "type": "pid"
  },
  {
    "type": "ipc",
    "path": "/proc/12012/ns/ipc"
  },
  {
    "type": "uts",
    "path": "/proc/12012/ns/uts"
  },
  {
    "type": "mount"
  },
  {
    "type": "network",
    "path": "/proc/12012/ns/net"
  }
]
```

3.  Notice that both use same namespace spec
4.  Under the path we can see the PID for ipc and uts namespaces - which is 12012 in our case
5.  Check what is PID 12012

```
node01 $ ps -ef | grep 12012
65535      12012   11982  0 12:21 ?        00:00:00 /pause
root       16023   13185  0 12:38 pts/0    00:00:00 grep --color=auto 12012
```

6.  Use below script to find the container (TBD)

```
#!/bin/bash

cpid=$1
while true; do
        ppid=$(ps -o ppid= -p $cpid)
        pname=$(ps -o comm= -p $ppid)
        if [ "$pname" == "docker" ]; then
                echo "$cpid parent $ppid ($pname)"
                break
        else
                echo "$cpid parent $ppid ($pname)"
                cpid=$ppid
        fi
done

docker ps -q | xargs docker inspect --format '{{.State.Pid}}, {{.Name}}' | grep $cpid
```

## UTS namespace

- The UTS namespace provides a layer for using the same hostname between containers.