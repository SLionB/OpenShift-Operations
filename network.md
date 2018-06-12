# Networking

Get information about the pod network, including the IP ranges allocated to each node
```bash
oc get hostsubnet
NAME      HOST      HOST IP           SUBNET
infra     infra     192.168.115.234   10.128.0.0/23
master    master    192.168.115.230   10.128.2.0/23
node1     node1     192.168.115.231   10.130.0.0/23
node2     node2     192.168.115.232   10.129.0.0/23
node3     node3     192.168.115.233   10.131.0.0/23

```
Check the status of the OVS service on each node in the cluster
```
systemctl status ovs-vswitchd
```

