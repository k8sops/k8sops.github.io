---
layout: page
title: SSH Alternative
category: ansible
permalink: /ansible/ssh_alternative
chapter: 5
---

## Connections available

```
ansible-doc -t connection --list
```

* When used ansible against localhost it established communication via SSH. Can be verified using the script below:
[](https://github.com/ansible/ansible/blob/devel/lib/ansible/plugins/connection/local.py)


* ***Docker Connection*** Plugin:

```
ansible-doc -t connection docker
```

* Creating containers using playbook:

```
- name: Ensure docker containers created
  hosts: localhost
  tasks:
  - name: Ensure docker container started 
    docker_container:
      image: python
      #command: bash
      interactive: yes
      name: "{{ item }}" 
      state: started
    loop: "{{ query('inventory_hostnames', 'containers') }}"
```

* Deleting containers:

```
- name: Ensure containers are gone 
  hosts: localhost
  tasks:
  - name: Ensure container is absent 
    docker_container:
      name: "{{ item }}" 
      force_kill: yes
      state: absent 
    loop: "{{ query('inventory_hostnames', 'containers') }}"
```

```
ansible-console containers // containers is the group from inventory
user@containers (3)[f:5]$ git_config list_all=yes scope=global

Alternative to that would be:

docker container exec -it ansible_container_test1 bash
```


