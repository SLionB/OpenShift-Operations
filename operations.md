Red Hat OpenShift Container Platform Operations
Introduction

Module Topics
-----------
Course Goals

Course Overview

Course Activities

Prerequisites

Class Environment

Course Goals
-------------
Build on knowledge of OpenShift Implementation

Deepen technical understanding of OpenShift operation

Describe form and use of certificates in OpenShift

Enable management of certificates

Describe monitoring and alerting in OpenShift context

Describe different monitoring solutions commonly used with OpenShift

Zabbix

Prometheus/Grafana

Describe different types of upgrades and how they are done

Course Overview
----------------
Module 01: Course Introduction

Module 02: Certificates

Module 03: Monitoring

Module 04: Upgrades

Course Activities
--------------------
Certificates Activity Group
--------------------------
Describe structure and functions of different types of certificates

Describe common tools and libraries for certificate management

Describe which components use certificates in OpenShift

Describe major features of OpenShift Ansible Playbooks with regard to certificates

Describe OpenShift default key and certificate deployment strategy

Describe how certificates and OAuth interrelate

Describe certificate redeployment use cases

Create certificate expiration reports with Ansible Playbooks

Describe certificate redeployment features of OpenShfit Ansible Playbooks

Deploy and redeploy certificate authorities in OpenShift with Ansible Playbooks

Redeploy client certificates for OpenShift users with Ansible Playbooks

Deploy and redeploy server certificates in OpenShift with Ansible Playbooks

Manage custom-named certificates for OpenShift master with OpenShift Ansible Playbooks

Redeploy new etcd certificate authority

Redeploy default registry and router certificates

Describe default registry and router certificates vs. custom registry and router certificates

Examine security level of OpenShift private keys

Describe OpenShift built-in command line tools for managing certificates

Monitoring Activity Group
--------------------------
Describe typical scenarios where monitoring is helpful in OpenShift

Describe difference between infrastructure monitoring and application monitoring

Describe in detail Docker monitoring for OpenShift

Describe in detail etcd, master, node monitoring

Describe how to monitor routers, registry, dnsmasq, logging, metrics

Describe purpose of cluster metrics

Compare/contrast Zabbix and Prometheus monitoring features

Create Prometheus query language expressions

Describe purpose of Prometheus Node Exporter

Describe use of Grafana with Prometheus in OpenShift

Describe alerting as available in Zabbix, Prometheus, Grafana


Upgrade Activity Group
--------------------------
Describe different types of release upgrades

Describe risks of different types of upgrades

Describe upgrading control plane

Describe upgrading nodes

Describe difference between in-place and blue-green upgrades

Prepare cluster for upgrade

Describe in-place, phased upgrade steps

Utilize upgrade hooks to perform custom tasks during in-place upgrade

Execute in-place, phased upgrades of groups of nodes

Describe in detail process of blue-green upgrade

Execute manual blue-green upgrade steps

Describe cluster blue-green upgrades

Execute manual cluster blue-green upgrade steps

Execute post-upgrade steps for cluster logging, metrics

Verify success of upgrade

Prerequisites
----------------
Knowledge of Red Hat Enterprise Linux 7

Hands-on experience with OpenShift 3.x

Ability to read and modify code

Workstation with SSH client and web browser installed available for training


Red Hat OpenShift Operations
============================
Certificates in OpenShift
--------------------------


Module Topics
--------------
Background

Certificates in OpenShift

Managing Certificates


Background
-----------
Certificate = file or byte stream composed of:

Identifying information (RDN) of issuer

Public key of private/public cryptographic keypair owned by issuer

Cryptographic signature of identifying information and cryptographic keypair

Identifying information, public keys, and cryptographic signatures of third-party "CA" signers

Certificates can be cryptographically signed by another certificate

Creates "chain of authority"

X.509: Standard for certificate format and contents

IETF PKIX Working Group leads standardization of X.509 certificate definition and use on Internet

Current format version: X.509v3

Certificate Types
```````````````
Type

Use

ca, root ca

Expressly created for signing CSRs from other certificates

intermediary

Signed by root ca certificate

Granted authority to sign other certificates

server, client, other end entity

Exchanged to validate identity and begin encrypted communications

Cannot sign other certificates

Common Tools and Libraries
``````````````````````````
openssl suite of tools and libraries

gnutls suite of tools, including certtool and libraries

Red Hat Identity Management and FreeIPA: Certificate Management, LDAP directory server, Kerberos, etc.

Certificates in OpenShift
--------------------------
Certificates stored in master servers

Major OpenShift components can either:

Share root CA

Indicate their own root CA certificate

Components include:

Masters (API server and controllers)

etcd

Nodes

Registry

Router

Ansible Playbooks
`````````````````
Ansible Playbooks provided with installer automate:

Deployment of custom certificates and keys at install time

Checking of expiration dates

Backups

Certificate redeployments

Certificates and Keys
`````````````````````
Default installation of OpenShift configures self-signed certificates

Production environments likely to require different root CA keys

Default private key length: 2048

Authenticating to OpenShift with Certificates
````````````````````````````````````````````
Authentication options: Certificates or OAuth tokens

kubeconfig configuration files contain certificates

Use to authenticate oc command for certain users

service accounts have keypairs in their configurations

Use to generate tokens for authenticating to OpenShift API

Managing Certificates
--------------------
Certificate Redeployment Use Cases
```````````````````````````````````
You have CA in organization, want to create certificates using it

Installer detected wrong host names, issue identified too late

Certificates expired, need to be updated

You have new applications, want to serve them securely

Checking Certificate Expirations
```````````````````````````````
Certificate expiry playbooks use Ansible role openshift_certificate_expiry

Certificates examined by role:

Master and node service certificates

Router and registry service certificates from etcd secrets

Master, node, router, registry, kubeconfig files for cluster-admin users

etcd certificates (including embedded)


Checking Certificate Expirations: Role Variables
`````````````````````````````````````````````````
openshift_certificate_expiry Ansible role uses following variables:

Core Variables

Variable Name Default Value Description

openshift_certificate_expiry_config_base

/etc/origin

Base OpenShift configuration directory

openshift_certificate_expiry_warning_days

30

Flag certificates that expire in this many days from now

openshift_certificate_expiry_show_all

no

Include healthy (non-expired and non-warning) certificates in results

Optional Variables
Variable Name Default Value Description

openshift_certificate_expiry_generate_html_report

no

Generate HTML report of expiry check results

openshift_certificate_expiry_html_report_path

/tmp/cert-expiry-report.html

Full path for saving HTML report

openshift_certificate_expiry_save_json_results

no

Save expiry check results as JSON file

Running Certificate Expiration Report Playbooks
`````````````````````````````````````````````````
Playbooks must be used with inventory file that is representative of cluster

For best results: Run ansible-playbook with -v option

easy-mode.yaml example playbook offers play to begin with

Produces JSON and stylized HTML reports in /tmp/

Sets very large warning window, so you almost always get results back

Includes all certificates (healthy or not) in results


Configure easy-mode.yaml :

- name: Check cert expirys
  hosts: nodes:masters:etcd
  become: yes
  gather_facts: no
  vars:
    openshift_certificate_expiry_warning_days: 1500
    openshift_certificate_expiry_save_json_results: yes
    openshift_certificate_expiry_generate_html_report: yes
    openshift_certificate_expiry_show_all: yes
  roles:
    - role: openshift_certificate_expiry
Run easy-mode.yaml :

$ ansible-playbook -v -i <inventory_file> \
    /usr/share/ansible/openshift-ansible/playbooks/certificate_expiry/easy-mode.yaml

Other Example Expiry Playbooks
``````````````````````````
File Name Use

default.yaml

Produces default behavior of openshift_certificate_expiry role

html_and_json_default_paths.yaml

Generates HTML and JSON artifacts in their default paths

longer_warning_period.yaml

Changes expiration warning window to 1,500 days

longer-warning-period-json-results.yaml

Changes expiration warning window to 1,500 days

Saves results as JSON file

To run any example playbook:

$ ansible-playbook -v -i <inventory_file> \
    /usr/share/ansible/openshift-ansible/playbooks/certificate_expiry/<playbook>
	
Configuring Certificate Validity
``````````````````````````````
Add or modify variables in Ansible inventory file

Changes duration of validity for certificates when generated by OpenShift:

[OSEv3:vars]

openshift_hosted_registry_cert_expire_days=730
openshift_ca_cert_expire_days=1825
openshift_node_cert_expire_days=730
openshift_master_cert_expire_days=730
etcd_ca_default_days=1825

Redeploying Certificates
``````````````````````````
Use playbooks to redeploy master, etcd, node, registry, router certificates on all relevant hosts

Options:

Redeploy all at once using current CA

Redeploy certificates for specific components only

Redeploy newly generated or custom CA on its own

Playbooks must be run with inventory file that is representative of cluster

Examine variables to ensure they are correct for cluster:

openshift_hostname

openshift_public_hostname

openshift_ip

openshift_public_ip

openshift_master_cluster_hostname

openshift_master_cluster_public_hostname

Redeploying a New or Custom OpenShift CA
`````````````````````````````````````````
redeploy-openshift-ca.yml playbook:

Generates new CA certificate

Distributes updated bundle to all components, including client kubeconfig files and node’s database of trusted CAs (CA-trust)

Includes serial restarts of:

Master services

Node services

Docker

Custom CAs also supported

Registry and routers continue to communicate with master without change

To use custom CA, set variable in inventory file:

# Configure custom ca certificate
# NOTE: CA certificate will not be replaced with existing clusters.
# This option may only be specified when creating a new cluster or
# when redeploying cluster certificates with the redeploy-certificates
# playbook.
openshift_master_ca_certificate={'certfile': '</path/to/ca.crt>', 'keyfile': '</path/to/ca.key>'}
If you do not set variable, current CA regenerated in next step

Run redeploy-openshift-ca.yml playbook, specifying inventory file:

 $ ansible-playbook -i <inventory_file> \
    /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/redeploy-openshift-ca.yml
Use redeploy-certificates.yml when you want to redeploy certificates signed by new CA on all components

Redeploying All Certificates
````````````````````````````
redeploy-certificates.yml does not regenerate {product-title} CA certificate

New certificates created, signed by current CA certificates

Master, etcd, Node, Registry, Router

Restarts:

etcd

Master services

Node services

Run playbook specifying inventory file:

$ ansible-playbook -i <inventory_file> \
    /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/redeploy-certificates.yml
	

Custom Named Master Certificates
``````````````````````````````````
Configure custom serving certificates for public host names of API server and web console:

During initial installation

When redeploying

Custom CA supported

Custom serving certificates cached on masters

Not overwritten unless openshift_master_overwrite_named_certificates=True

To configure custom serving certificates, use openshift_master_named_certificates and openshift_master_overwrite_named_certificates Ansible variables

Configurable in inventory file

Example:

openshift_master_named_certificates=[{"certfile": "/path/on/host/to/custom1.crt", "keyfile": "/path/on/host/to/custom1.key", "cafile": "/path/on/host/to/custom-ca1.crt"}]
openshift_master_overwrite_named_certificates=True


Redeploying a New etcd CA
````````````````````````````
redeploy-etcd-ca.yml playbook redeploys etcd CA

Also serial restarts:

etcd

master services

To redeploy newly generated etcd CA, run redeploy-etcd-ca.yml, specifying inventory file:

$ ansible-playbook -i <inventory_file> \
    /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/redeploy-etcd-ca.yml
Run redeploy-etcd-certificates.yml at your discretion

Alternative: Use redeploy-certificates.yml to redeploy certificates for OpenShift components besides etcd peers and master clients

Redeploying Registry or Router Certificates
```````````````````````````````````````````
redeploy-registry-certificates.yml and redeploy-router-certificates.yml replace installer-created certificates for registry and router

Automated replacement of registry certificates with the OpenShift CA is explained later in course

Manual replacement is required if certificates use a different CA from OpenShift

Redeploying Default Registry and Router Certificates
```````````````````````````````````````````````````
Redeploy router certificates

Run following playbook

Specify inventory file:

$ ansible-playbook -i <inventory_file> \
    /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/redeploy-router-certificates.yml
Redeploy registry certificates

Run following playbook

Specify inventory file:

$ ansible-playbook -i <inventory_file> \
    /usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/redeploy-registry-certificates.yml
	
Redeploying Custom Registry or Router Certificates
`````````````````````````````````````````````````
Registry and router certificates must be redeployed when custom CA is redeployed

If not, will lose contact with masters

Playbooks for redeploying certificates cannot redeploy custom registry or router certificates

Workaround: Manually redeploy registry and router certificates

See OpenShift documentation on Redeploying Certificates for procedures


OpenShift Command Line Built-In Tools
``````````````````````````````````````
OpenShift client oc has built-in functions for creating and managing certificates and keys

Keys

Certificate signing requests

Certificate authorities

Signing of all manner of certificates

Command

Use

oc adm certificate approve

Approve certificate signing request

oc adm certificate deny

Deny certificate signing request

oc adm ca create-key-pair

Create public/private key pair

oc adm ca create-master-certs

Create certificates and keys for master

oc adm ca create-server-cert

Create signed server certificate and key

oc adm ca create-signer-cert

Create signer (certificate authority/CA) certificate and key

oc adm ca encrypt

Encrypt data with AES-256-CBC encryption

oc adm ca decrypt

Decrypt data encrypted with oc adm ca encrypt


Examining Private Keys in OpenShift
````````````````````````````````````
# ansible masters -m shell -a'for x in /etc/origin/master/*.key; do echo \"$x\"; openssl rsa -in ${x} -noout -text 2>&1 | head -1
; echo === ;done;' | less
master1.alb1.internal | SUCCESS | rc=0 >>
"/etc/origin/master/admin.key"
Private-Key: (2048 bit)
===
"/etc/origin/master/ca.key"
Private-Key: (2048 bit)
===
"/etc/origin/master/etcd.server.key"
Private-Key: (2048 bit)
===




Course Activities
--------------------
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


Module Topics
=============
Upgrade Basics

Upgrade Types

In-Place Upgrades

In-Place Blue-Green Upgrades

Cluster Blue-Green Upgrades

Post-Upgrade Steps



Upgrade Basics
Upgrades = necessary maintenance task

Two upgrade scenarios:

Scenario

Description

Upgrade entire cluster

Minor version upgrade (3.5.5.15 → 3.5.5.26)

Major version upgrade (3.5 → 3.6)

Requires cluster to be on latest minor version of previous major version

Upgrade individual components

For errata on individual components like router or registry

Instructions included in errata



Upgrade Basics
Control Plane vs. Nodes
Control plane components

etcd

Masters

Node services on masters

Docker on masters

Docker on standalone etcd hosts

Node components

Node service on non-master nodes

Docker on non-master nodes



Upgrade Types
Two upgrade types:

In-place upgrades

Blue-green upgrades

Possible to combine both

Manual intervention necessary



In-Place Upgrades
Fully automated via Ansible Playbooks

Original Ansible hosts file needs to be available

Upgrade control plane first

Single-master environment incurs downtime

Upgrade nodes second

No manual orchestration of external router necessary


In-Place Upgrades
Upgrade Preparation
Upgrade to latest minor version for currently installed major version

Change Red Hat Subscription Manager channels to latest version

Install latest playbooks:

yum update atomic-openshift-utils

Log in as cluster-admin user (system:admin) on master host(s)

Run oc get nodes

Verify all nodes are ready

Verify non-schedulable labels


In-Place Upgrades
Grouping Nodes During Upgrade
Can set openshift_upgrade_nodes_serial variable in Ansible hosts file

Specifies percentage of nodes to be upgraded at one time

Example: Upgrade 20% of total nodes at a time:

ansible-playbook -i <path/to/inventory/file> </path/to/upgrade/playbook> -e openshift_upgrade_nodes_serial="20%"

Example: Upgrade two nodes at a time with label region=group1:

ansible-playbook -i <path/to/inventory/file> </path/to/upgrade/playbook> -e openshift_upgrade_nodes_serial="2" -e openshift_upgrade_nodes_label="region=group1"




In-Place Upgrades
In-Place Automated Upgrade Steps
Apply latest configuration

Upgrade master and etcd components, restart services

Upgrade node components, restart services

Apply latest cluster policies

Update default router if one exists

Update default registry if one exists

Update default image streams and InstantApp templates


In-Place Upgrades
Upgrade Hooks
Use to execute custom tasks during upgrade

Example: Pause for approval before each node

Hooks defined in Ansible hosts file:

openshift_master_upgrade_pre_hook=/usr/share/custom/pre_master.yml
openshift_master_upgrade_hook=/usr/share/custom/master.yml
openshift_master_upgrade_post_hook=/usr/share/custom/post_master.yml


In-Place Upgrades
Running the Upgrade
Use same playbook for minor and major upgrades

Options:

Option

Playbook

Upgrade entire cluster at once

upgrade.yml

Upgrade control plane first, nodes second

upgrade_control_plane.yml first

upgrade_nodes.yml second


In-Place Blue-Green Upgrades
Next level of complexity

Mostly manual process

Requires cluster with multiple masters and load balancer

Temporarily requires additional nodes

Can share software entitlements

Detailed instructions available in OpenShift documentation



In-Place Blue-Green Upgrades
Upgrade Steps
Upgrade control plane

Same process as in-place upgrades

Label:

Masters: type=master

Nodes: type=node

Upgrade nodes



In-Place Blue-Green Upgrades
Moving Traffic
Make green nodes schedulable

Make blue nodes unschedulable

Update latest image streams and templates

Triggers builds on green nodes

Drain/evacuate old nodes

Decommission old nodes

Remove from OpenShift cluster

Remove software entitlement

Delete VMs

Repeat process until no more blue nodes in cluster



Cluster Blue-Green Upgrades
Ultimate in deployment sophistication

Requires two active clusters (double the hardware)

Deploy application to both clusters

Leverage external router to cut traffic from old cluster to new cluster

Manual process

Need to understand workloads and persistence requirements


Post-Upgrade Steps
Upgrading the EFK Logging Stack
Use provided Ansible Playbook

/usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/openshift-logging.yml

Use for both:

Deploying logging for first time on existing cluster

Upgrading existing logging deployment

Specify logging variables

Use latest major version as <tag>

openshift_logging_install_logging=true
openshift_logging_image_version=<tag>


Post-Upgrade Steps
Upgrading Cluster Metrics
Use provided Ansible Playbook

/usr/share/ansible/openshift-ansible/playbooks/byo/openshift-cluster/openshift-metrics.yml

Specify metrics variables

Use latest major version as <tag>

openshift_metrics_install_metrics=true
openshift_metrics_image_version=<tag>
openshift_metrics_hawkular_hostname=<fqdn>
openshift_metrics_cassandra_storage_type=(emptydir|pv|dynamic)



Post-Upgrade Steps
Upgrade Verification
Check that all nodes marked as Ready

oc get nodes

Verify version numbers for docker-registry and router images

Example: for v3.6.173.0.5

oc get -n default dc/docker-registry -o json | grep \"image\" "image": "openshift3/ose-docker-registry:v3.6.173.0.5"

oc get -n default dc/router -o json | grep \"image\" "image": "openshift3/ose-haproxy-router:v3.6.173.0.5"

Run oc adm diagnostics to look for common issues



