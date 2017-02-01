# Kafka

Master: [![Build Status](https://travis-ci.org/sansible/kafka.svg?branch=master)](https://travis-ci.org/sansible/kafka)  
Develop: [![Build Status](https://travis-ci.org/sansible/kafka.svg?branch=develop)](https://travis-ci.org/sansible/kafka)

* [ansible.cfg](#ansible-cfg)
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
* [Maintenance scripts] (#maintenance-scripts)
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
  version: v1.0
```

and run `ansible-galaxy install -p ./roles -r roles.yml`




## Tags

This role uses two tags: **build** and **configure**

* `build` - Installs Kafka server and all it's dependencies.
* `configure` - Configure and ensures that the Kafka service is running.




## Maintenance scripts

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
