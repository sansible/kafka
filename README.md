# Kafka

Master: [![Build Status](https://travis-ci.org/sansible/kafka.svg?branch=master)](https://travis-ci.org/sansible/kafka)
Develop: [![Build Status](https://travis-ci.org/sansible/kafka.svg?branch=develop)](https://travis-ci.org/sansible/kafka)

* [ansible.cfg](#ansible-cfg)
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
* [Maintenance scripts](#maintenance-scripts)
* [Examples](#examples)

This roles installs Apache Kafka server.

For more information about Kafka please visit
[zookeeper.apache.org/](http://kafka.apache.org/).




## ansible.cfg

This role is designed to work with merge "hash_behaviour". Make sure your
ansible.cfg contains these settings

```INI
[defaults]
hash_behaviour = merge
```




## Installation and Dependencies

This role will install `sansible.users_and_groups` for managing `kafka`
user.

To install run `ansible-galaxy install sansible.kafka` or add this to your
`roles.yml`

```YAML
- name: sansible.kafka
  version: v1.2
```

and run `ansible-galaxy install -p ./roles -r roles.yml`

### AWS Setup

This role has AWS support built in, it supports two methods for deployment/discovery.

#### AWS Cluster Autodiscovery

This method is designed for use with a single ASG controlling a cluster of Kafka instances,
the idea being that instances can come and go without issue.

The [AWS Autodiscover script](/files/aws_cluster_autodiscover) allows machines to pick an
ID and hostname/Route53 entry from a predefined list, AWS tags are used to mark machines
that have claimed an ID/host.

This script allows for a static set of hostnames with consistent IDs to be maintained
accross a dynamic set of instances in an ASG.

```YAML
- role: sansible.kafka
  kafka:
    aws_cluster_autodiscover:
      enabled: true
      hosts:
        - 01.kafka.io.internal
        - 02.kafka.io.internal
        - 03.kafka.io.internal
      lookup_filter: "Name=tag:Environment,Values=dev Name=tag:Role,Values=kafka"
      r53_zone_id: xxxxxxxx
    # A ZK cluster behind an ELB
    zookeeper_hosts:
      - zookeeper.app.internal
```

#### AWS Tag Discovery

Designed for instances that are stacially defined either as direct EC2 instances or
via a single ASG per instance.

The broker.id is derived from a tag attached to the instance, you can turn on this
behaviour and specify the tag to lookup like so:

```YAML
- role: sansible.kafka
  kafka:
    aws_cluster_assigned_id:
      enabled: true
      id_tag_name: instanceindex
    # A ZK cluster behind an ELB
    zookeeper_hosts:
      - zookeeper.app.internal
```




## Tags

This role uses two tags: **build** and **configure**

* `build` - Installs Kafka server and all it's dependencies.
* `configure` - Configure and ensures that the Kafka service is running.




## Maintenance scripts

These scripts are used in conjunction with the
[AWS Cluster Autodiscovery](aws-cluster-autodiscovery) deployment method.

* kafka_maintenance_at_start

  Intention behind this script is to introduce a new node to the cluster and evenly
  redistribute data. It's included in Configure stage of Ansible role.
  The new node contacts Zookeeper (ZK) and requests all brokers IDs currenly holding data.
  Once information is received json file is generated and information provided to ZK.


* kafka_maintenance_at_stop

  Intention behind this script is to allow node to remove itself from cluster during shutdown
  and evenly redistribute data to remaining nodes. Script is triggerend by stop_kafka included
  in relevant runleves.
  Node contacts Zookeeper (ZK) and requests all brokers IDs currenly holding data.
  Once information is received json file is generated and information provided to ZK.

* remove_dns_record

  After kafka_maintenance_at_stop is executed during shutdown (stop_kafka) node removes itself
  from Route53 (AWS).

* TODO:
  Becaue kafka_maintenance_start/stop are almost identical they can be merged.
  Depends on use argument could be provided.
  Example:
  kafka_maintenance at_start

  To remove node from Route53 (AWS) Ansible module can be also used.
  This will require tests.




## Examples

```YAML
- name: Install Kafka Server
  hosts: sandbox

  pre_tasks:
    - name: Update apt
      become: yes
      apt:
        cache_valid_time: 1800
        update_cache: yes
      tags:
        - build

  roles:
    - name: sansible.kafka
      kafka:
        zookeeper_hosts:
          - my.zookeeper.host
```

If you just want to test Kafka service build both Zookeeper and Kafka on the
same machine.

```YAML
- name: Install Kafka Server
  hosts: sandbox

  pre_tasks:
    - name: Update apt
      become: yes
      apt:
        cache_valid_time: 1800
        update_cache: yes
      tags:
        - build

  roles:
    - name: sansible.zookeeper
    - name: sansible.kafka
```
