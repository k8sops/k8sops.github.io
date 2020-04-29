---
layout: page
title: Roles and Collection
category: ansible
permalink: /ansible/roles
chapter: 6
---

Lower level blocks a.k.a Plugins:
![](images/plugins.png)

Composing using roles & collections:

![](images/composition.png)

***Roles & collections are concept higher than modules and lower than playbooks. Kind of pre-baked libraries achieving very specific set of tasks.***

### Ansible Galaxy

![](https://galaxy.ansible.com/search?keywords=git_config&order_by=-relevance&page=1&deprecated=false&type=collection)

```
ansible-galaxy -h
ansible-galaxy role -h
ansible-galaxy role search git_config
ansible-galaxy collection -h
```
* Installing a role:

```
ansible-galaxy info geerlingguy.jenkins
ansible-galaxy install geerlingguy.jenkins
// default location where role is installed
ls -la ~/.ansible/roles/geerlingguy.jenkins
ansible-galaxy remove geerlingguy.jenkins -v
```
* Side note on /tmp folder and default setting for keep temporary files when executing on remote server. Can be overwritten in the inventory file as well.

![](ansible/images/default_keep_remote_files.png)

* Galaxy docs:

[](https://galaxy.ansible.com/docs/index.html)

* Docs about creating roles:

[](https://galaxy.ansible.com/docs/contributing/creating_role.html)

* Docs for collections, still under devel branch:

[](https://docs.ansible.com/ansible/devel/dev_guide/developing_collections.html#)

## Testing a role in a container:

```
docker container run --rm -it python bash
pip install ansible
```

```
cat playbook.yaml 
---
- name: include jenkins role
  hosts: localhost
  roles:
        - geerlingguy.jenkins

ansible-playbook playbook.yaml
```