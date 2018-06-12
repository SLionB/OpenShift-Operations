# Networking

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
Check the status of the OVS service on each node in the cluster:
```
$ systemctl status ovs-vswitchd
```

View pods' virtual Ethernet (veth) interfaces that’s linked to the eth0 interface in the pods from the application node:
```
# ip a | egrep '^[0-9].*:' | awk '{ print $1 $2}'
1:lo:
2:eth0:
3:ovs-system:
6:br0:
7:docker0:
8:vxlan_sys_4789:
9:tun0:
10:veth68d047ad@if3:
11:veth875e3121@if3:
12:vethb7bbb4d5@if3:
13:vethd7768410@if3:
14:veth8f8e1db6@if3:
15:veth334d0271@if3:
```

Find pod name where a specific application is running
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

