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


