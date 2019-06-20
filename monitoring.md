Monitoring Activity Group
--------------------------
Module Topics
-----------
What Is Monitoring?

What Needs Monitoring?

Monitoring Solutions

Zabbix

Prometheus

Grafana

Alerts

What Is Monitoring
----------------
OpenShift: Complex product with many components

Any number of components can fail

Common scenarios:

Disk/volume group full

Memory exhausted

Non-responsive components

Ideal: Early detection mechanism before things fail

Required: Alerting before/when things fail

Solution Needs
`````````````````
Monitoring solution requirements:

Regularly check system status

Alert operators of abnormal state

Other solution options:

Show historical usage data

Show graphical dashboards

Can set up solution externally or on OpenShift

Monitoring Types
````````````````
Infrastructure monitoring

OpenShift nodes, masters, etc.

OpenShift components

Application monitoring

Customer applications running on OpenShift

Outside scope for course


What Needs Monitoring
-------------------
Docker
````````
OpenShift relies heavily on Docker on all nodes

Docker items to monitor:

Docker service liveness

Space available in Docker volume group

Hidden (non-mounted) volume

Individual on nodes and aggregate over cluster

Metadata space available in Docker volume group

Volume storage on nodes (/var/lib/docker)

Where Docker stores volume data


Docker Monitoring Commands Examples
``````````````````````````````````
Check

Command

Docker daemon

systemctl is-active docker

Check that Docker volume group has adequate space

echo $(echo \"$(docker info 2>/dev/null | awk '/Data Space Available/ {print $4}') / $(docker info 2>/dev/null | awk '/Data Space Total/ {print $4}')\" | bc -l) '>' 0.3 | bc -l

Check that Docker volume group has adequate metadata space

echo $(echo \"$(docker info 2>/dev/null | awk '/Metadata Space Available/ {print $4}') / $(docker info 2>/dev/null | awk '/Metadata Space Total/ {print $4}')\" | bc -l) '>' 0.3 | bc -l


Nodes
`````
On:

Monitor:

etcd

Server process

API

Masters

atomic-openshift-master(-api) service

API endpoint (master:433/healthz)

All nodes (including Masters)

atomic-openshift-node service

Available/used memory

Available/used file system space (of type xfs)


etcd and Master Monitoring Commands Examples
Check

Command

etcd is active

systemctl is-active etcd

etcd volume is not too full

echo "$(lvs | awk '/etcd/ {print $5}') > 70" | bc

Master API service is active

systemctl is-active atomic-openshift-master

Master API service is active (multi-master)

systemctl is-active atomic-openshift-master-api

Master controller service is active (multi-master)

systemctl is-active atomic-openshift-master-controller


Node Monitoring Commands Examples
Check

Command

Node service is active

systemctl is-active atomic-openshift-node

Node’s local data storage volume is not too full

echo "$(lvs | awk '/origin/ {print $5}') > 70" | bc

openvswitch service is active

systemctl is-active openvswitch


OpenShift Components
```````````````
Routers

Registry

DNSmasq

Logging

Elasticsearch

Kibana

Metrics

Hawkular

Heapster

Cassandra


API EndPoints
``````````````
Check

Command

Health of master API endpoint

curl -H "Authorization: Bearer $(oc whoami -t)" https://<my_cluster_api>:8443/healthz | grep ok

Health of router

curl http://router.default.svc.cluster.local:1936/healthz | grep 200

Health of registry

curl -I https://docker-registry.default.svc.cluster.local:5000/healthz | grep 200

Health of EFK logging stack

See OpenShift Logging health check script for an example

Health of metrics stack

See OpenShift Metrics health check script for an example


OpenShift Cluster Metrics
``````````````````````````
Aggregate metrics on cluster level

CPU usage

Requests

Load

Memory usage

Requests

Used

Memory reserved

Total in cluster (aggregate over all nodes)


Monitoring Solutions
---------------------
Most customers already have monitoring solution

Any solution can be integrated with OpenShift

OpenShift Online uses Zabbix

Prometheus is strategic direction for Kubernetes (and therefore OpenShift)

Every Kubernetes/OpenShift component to expose metrics endpoint for Prometheus to collect data from

Planned for Kubernetes 1.7/OpenShift 3.7, subject to change


Zabbix
---------
Available from "zabbix.com^"

Requires agents on all monitored machines

Agents listening on port 10050

Needs to be accessible through firewall

Can be configured for agents to self-register when deployed

Uses pull model to fetch updated metrics from agents


Zabbix
Configuration
Zabbix can group similar hosts (e.g., OCP Host)

Each host can be member of one or more templates

Templates determine metrics to monitor

Applications

Specific items (e.g., available memory)

Triggers to set thresholds for alerting

Graph definitions for dashboards

Zabbix comes with default templates

Must be customized for OpenShift


Prometheus
------------
Available from "prometheus.io^"

Part of Cloud Native Computing Foundation

Open source

Multiple small components

Prometheus

Alert Manager

Node Exporter

Many optional components


Prometheus
Prometheus
Time series database

Multi-dimensional data model

Relies heavily on labels

Powerful queries

Efficient storage

No requirement for distributed storage

Local host storage preferred

No remote storage—e.g., no NFS

Time series collection via pull model over HTTP

Monitors itself

Prometheus
Considerations When Using Prometheus
No authorization mechanism

Can be configured with sidecar container for OAuth authentication via OpenShift

May make integration with Grafana and other tools difficult

Powerful query language

Aggregate metrics over time

Calculate average, sum, expressions

Group metrics by labels


Prometheus
OpenShift Service Monitoring
Prometheus can discover OpenShift services to monitor

Application needs to expose metrics in Prometheus format on /metrics endpoint

Service definition needs annotations

prometheus.io/scrape: "true"

prometheus.io/scheme: http

Set via oc annotate service <ServiceName> prometheus.io/scrape=true

No additional configuration necessary

Status of discovered services available on Targets page in Prometheus



Prometheus
OpenShift Service Monitoring
Prometheus can discover OpenShift services to monitor

Application needs to expose metrics in Prometheus format on /metrics endpoint

Service definition needs annotations

prometheus.io/scrape: "true"

prometheus.io/scheme: http

Set via oc annotate service <ServiceName> prometheus.io/scrape=true

No additional configuration necessary

Status of discovered services available on Targets page in Prometheus



PromQL Query Language Complex Examples
Query

Command

Number of containers that start or restart over previous 10 minutes

sum(changes(container_start_time_seconds[10m]))

Number of mutating API requests being made to control plane

sort_desc(drop_common_labels(sum without (instance,type,code) (rate(apiserver_request_count{verb=~"POST|PUT|DELETE|PATCH"}[5m]))))

Number of non-mutating API requests being made to control plane

sort_desc(drop_common_labels(sum without (instance,type,code) (rate(apiserver_request_count{verb=~"GET|LIST|WATCH"}[5m]))))

Top 10 pods doing most receive network traffic

topk(10, (sum by (pod_name) (rate(container_network_receive_bytes_total[5m]))))



Node Exporter
Collects system data from machines

Useful for nodes without Prometheus endpoints

Also collects custom metrics via text file collector

Metrics that Prometheus cannot collect on its own—for example:

Docker status

Docker volume group status

Custom script writes metrics into text file

Node Exporter collects metrics



Prometheus
Node Exporter Configuration
Can be run natively

Use Ansible Playbook to distribute to all nodes

Can be run containerized

Distribute as DaemonSet

Needs hostaccess SCC to bind to port 9100

Both need port 9100 open on firewall



Grafana
----------
Dashboards for various data sources

Includes Prometheus

Highly configurable

Plug-ins

Custom widgets

Sample dashboards available for many scenarios

Example: Docker or Kubernetes monitoring via Prometheus

Alerting
=============
Monitoring without alerting is useless

People cannot be expected to watch dashboard all day

Need alerts when things go wrong

Alerts based on metric thresholds—for example:

Node unreachable for more than 30 seconds

File system > 90% full

CPU > 80% for more than 5 minutes


Alerting Sources
Good monitoring solutions have alerting

Zabbix: Alerting built in

Prometheus: Alertmanager component

Grafana: Alerting built in

Can choose either Alertmanager or Grafana for alerting with Prometheus


Alerting
Alerting Targets
Common targets for alerts:

Pagers

SMS (text messages)

Email

Chat (e.g., Slack)

Set different types of alerts to different recipients via different channels



Alerting
Prometheus Alertmanager
Optional component

Other alerting mechanisms possible (Grafana)

Alerting rules set up in Prometheus servers

When alert triggers, event sent to Alertmanager

Alertmanager manages alerts including deduplicating, grouping, routing to correct receiver

Notification

Email, webhooks, Slack, PagerDuty

Silencing, aggregation, inhibiting
