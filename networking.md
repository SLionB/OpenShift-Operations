# Networking

## Viewing pod network information
Get information about the pod network, including the IP ranges allocated to each node:
```bash
$ oc get hostsubnet
NAME      HOST      HOST IP           SUBNET
infra     infra     192.168.115.234   10.128.0.0/23
master    master    192.168.115.230   10.128.2.0/23
node1     node1     192.168.115.231   10.130.0.0/23
node2     node2     192.168.115.232   10.129.0.0/23
node3     node3     192.168.115.233   10.131.0.0/23
```

## Checking OVS service status
Check the status of the OVS service on each node in the cluster:
```
$ systemctl status ovs-vswitchd
```

## Viewing OVS veth interfaces linked to pods
View pods' virtual Ethernet (veth) interfaces that’s linked to the eth0 interface in the pods from the application node:
```
# ip a | egrep '^[0-9].*:' | awk '{ print $1 $2}'
1:lo:
2:ens192:
3:ovs-system:
6:br0:
7:docker0:
8:vxlan_sys_4789:
9:tun0:
11:vethab9703b2@if3:
12:veth0c107a4c@if3:
13:vetha2d62f98@if3:
127:veth2983d232@if3:
135:vethb26e300c@if3:
142:vethfd5087e3@if3:
144:veth11ce49e3@if3:
145:veth8cbcd592@if3:
153:vethed403796@if3:
156:vethe07c877b@if3:
163:vethe8861d2a@if3:
166:vetha3822557@if3:
169:veth3d41f6c4@if3:
172:vethe3d4abb0@if3:
```
## Tracing pod network traffic from host
Find pod name and node name where a specific application is running
```
$ oc get pods -o wide -n image-uploader --show-all=false
NAME              READY     STATUS    RESTARTS   AGE       IP             NODE
app-cli-1-r9xc1   1/1       Running   0          10d       10.131.0.225   node3
app-gui-1-3j2gb   1/1       Running   0          10d       10.131.0.228   node3
ruby-ex-1-v1287   1/1       Running   0          10d       10.131.0.222   node3
```
Find the linked interface index for the app-cli container eth0 interface
```
$ oc exec app-cli-1-r9xc1 cat /sys/class/net/eth0/iflink
169
```
Find the veth interface name that app-cli is associated using the linked interface index.
```
$ ssh node3
# ip a | egrep -A 3 '^169.*:'
169: veth3d41f6c4@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue master ovs-system state UP
    link/ether 76:c2:65:55:d8:d4 brd ff:ff:ff:ff:ff:ff link-netnsid 15
    inet6 fe80::74c2:65ff:fe55:d8d4/64 scope link
       valid_lft forever preferred_lft forever
```
Print network traffic(HTTP packets to and from port 80) on veth inteface
```
tcpdump -i veth3d41f6c4@if3 'tcp port 80'
```

## Working with OVS
List OVS bridge
```
# ovs-vsctl list-br
br0
```
List the interfaces connected to br0
```
# ovs-vsctl list-ifaces br0
tun0
veth02a98eb8
veth0c107a4c
veth0e4698cb
vxlan0
```

## Working with Router
Find that haproxy service is listening on port 80 (on the infra node):
```
# netstat -tpl --numeric-ports | grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      108765/haproxy
```
Display the namespaces associated with the haproxy process 
```
# lsns -p 108765
        NS TYPE  NPROCS   PID USER          COMMAND
4026531837 user     438     1 root          /usr/lib/systemd/systemd --switched-root --system --deserialize 21
4026531838 uts      412     1 root          /usr/lib/systemd/systemd --switched-root --system --deserialize 21
4026531956 net      412     1 root          /usr/lib/systemd/systemd --switched-root --system --deserialize 21
4026532921 ipc        3  3829 ansibleremote /usr/bin/pod
4026533229 mnt        2 16533 1000000000    /usr/bin/openshift-router
4026533230 pid        2 16533 1000000000    /usr/bin/openshift-router

```
Verify that the router pod uses the UTS and network namespaces from the host system
```
[root@infra ~]# lsns -p 108765 | grep 4026531838
4026531838 uts      414     1 root          /usr/lib/systemd/systemd --switched-root --system --deserialize 21
# lsns -p 108765 | grep 4026531956
4026531956 net      415     1 root          /usr/lib/systemd/systemd --switched-root --system --deserialize 21
```
Get the router pod name:
```
# oc get pods -n default | grep router
router-2-tr7xb             1/1       Running   9          138d
```
Enter the router pod:
```
$ oc project default
$ oc rsh router-2-tr7xb 
sh-4.2$
```
Verify that the router pod can see the host’s network
```
ip a
```

## Search for an application HAProxy configuration in the router pod
Grep haproxy configuration for application name
```
$ grep app-cli /var/lib/haproxy/conf/haproxy.config
```

## View the services for a project
```
$ oc get services -n image-uploader
```
## Access a service from an application node using DNS
```
# curl app-cli.image-uploader.svc.cluster.local:8080
```

## Enabling the ovs-multitenant plugin
1. Open the master configuration file located at /etc/origin/master/masterconfig.
yaml.
2. Locate the networkPluginName parameter in the file. The default value that
enables the ovs-subnet plugin is redhat/openshift-ovs-subnet. Edit this line
as shown next.
...
networkPluginName: redhat/openshift-ovs-multitenant
...
3. Restart the origin-master service:
```
# systemctl restart origin-master
```


