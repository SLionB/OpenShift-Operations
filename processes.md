# Processes

## Finding Container processes from the host
Get container id
```
$ docker ps | grep app-cli
760dd19c663b        openshift3/ose-pod:v3.6.173.0.63                                                                                                                   "/usr/bin/pod"           10 days ago         Up 10 days                              k8s_POD_app-cli-1-r9xc1_image-uploader_548306a0-65db-11e8-81ff-0050568c65e9_0
```
Find the process id of this container
```
$ docker inspect --format '{{ .State.Pid }}' 760dd19c663b
40112
```
Get the processes 
```
pstree -p 40112
