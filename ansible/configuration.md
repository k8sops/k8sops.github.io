---
layout: page
title: Configuration in Ansible
category: ansible
permalink: /ansible/configuration
chapter: 4
---

[](https://docs.ansible.com/ansible/latest/reference_appendices/config.html#ansible-configuration-settings)

```
Changes can be made and used in a configuration file which will be searched for in the following order:

ANSIBLE_CONFIG (environment variable if set)
ansible.cfg (in the current directory)
~/.ansible.cfg (in the home directory)
/etc/ansible/ansible.cfg
Ansible will process the above list and use the first file found, all others are ignored.
```

[](https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html#general-precedence-rules)
