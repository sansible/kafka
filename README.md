# Kafka

Master: [![Build Status](https://travis-ci.org/ansible-city/kafka.svg?branch=master)](https://travis-ci.org/ansible-city/kafka)  
Develop: [![Build Status](https://travis-ci.org/ansible-city/kafka.svg?branch=develop)](https://travis-ci.org/ansible-city/kafka)

* [ansible.cfg](#ansible-cfg)
* [Installation and Dependencies](#installation-and-dependencies)
* [Tags](#tags)
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

This role will install `ansible-city.users_and_groups` for managing `kafka`
user.

To install run `ansible-galaxy install ansible-city.kafka` or add this to your
`roles.yml`

```YAML
- name: ansible-city.kafka
  version: v1.0
```

and run `ansible-galaxy install -p ./roles -r roles.yml`





## Tags

This role uses two tags: **build** and **configure**

* `build` - Installs Kafka server and all it's dependencies.
* `configure` - Configure and ensures that the Kafka service is running.




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
    - name: ansible-city.kafka
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
    - name: ansible-city.zookeeper
    - name: ansible-city.kafka
```
