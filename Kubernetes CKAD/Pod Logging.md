# View logs of a pod
## Single container pod
```
k logs myapp
```

## view logs live stream
```
k logs myapp -f
```

## Multi container pod
```
k logs myapp <container_name> -f
```