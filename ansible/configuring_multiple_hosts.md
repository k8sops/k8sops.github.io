---
layout: page
title: Configuring Multiple Hosts
category: ansible
permalink: /ansible/multiple-hosts
chapter: 3
---

## File types for Inventory files

* YAML (.yaml / .yml)
* INI
* Dynamic Inventory using python files
* File extensions that would be ignored are *.orig and more

```
ansible-config list

Look for IGNORE ->

INVENTORY_IGNORE_EXTS:
  default: '{{(BLACKLIST_EXTS + ( ''.orig'', ''.ini'', ''.cfg'', ''.retry''))}}
  description: List of extensions to ignore when using a directory as an invent
    source
```

## Listing and Inventory Graph

```
ansible-inventory --graph

@all:
  |--@ungrouped:
  |  |--localhost
  |--@vagrant:
  |  |--@centos:
  |  |  |--centos20
  |  |  |--centos21
  |  |--@ubuntu:
  |  |  |--ubuntu10
  |  |  |--ubuntu11

ansible-inventory --graph --vars // list out variables

@all:
  |--@ungrouped:
  |  |--localhost
  |  |  |--{ansible_connection = local}
  |  |  |--{ansible_python_interpreter = /usr/local/bin/python3}
  |--@vagrant:
  |  |--@centos:
  |  |  |--centos20
  |  |  |  |--{ansible_host = 192.168.50.20}
  |  |  |  |--{ansible_port = 22}
  |  |  |  |--{ansible_private_key_file = .vagrant/machines/centos20/virtualbox/private_key}
```

```
ansible inventory --list
```

## Using a Inventory file

```
ansible -m command -a "git config --global --list" vagrant

Here -> vagrant is the group configured in Inventory

Output ->
centos21 | FAILED | rc=2 >>
[Errno 2] No such file or directory: 'git': 'git'

ubuntu11 | FAILED | rc=128 >>
fatal: unable to read config file '/home/vagrant/.gitconfig': No such file
 or directorynon-zero return code

centos20 | FAILED | rc=2 >>
[Errno 2] No such file or directory: 'git': 'git'

ubuntu10 | FAILED | rc=128 >>
fatal: unable to read config file '/home/vagrant/.gitconfig': No such file or directorynon-zero return code
```

* ubuntu system has git installed but no git config. centos do have git installed.

Sample Playbook:

```
---

- name: Ensure git installed
  hosts: centos 
  tags: [ 'install-git' ]
  tasks: 
  - package: name=git state=latest
    when: ansible_os_family == 'RedHat' # conditional task
    become: yes # https://docs.ansible.com/ansible/latest/user_guide/become.html
  # Note: since we have a group of centos hosts we don't really need the when condition 
  # but it is here to demo conditional tasks and in the event we have a mistake or 
  # a group that contains a host that isn't a RHEL derivative 

- name: Ensure ~/.gitconfig copied from master.gitconfig
  hosts: vagrant
  tasks:
  
  - name: git_config module simplifies listing configuration 
    git_config: list_all=yes scope=global

  - name: first show no config in targets
    command: git config --global --list
    ignore_errors: yes 
      # https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html#ignoring-failed-commands
      # we know git config lookup will fail so ensure we don't stop playbook execution with ignore_errors
    register: git_config_before # how to register variables, in this case we capture the module output 
  - name: show git config output always - verbosity 0 is default for debug module
    debug: var=git_config_before # how to use variables!
  
  - name: tried and true copy module with master.gitconfig from previously in the course 
    copy: src=master.gitconfig dest=~/.gitconfig
  
  - name: show newly added config
    command: git config --global --list
    ignore_errors: yes
    register: git_config_after
  - name: ensure to show git config after with debug - this time only show stdout_lines
    debug: var=git_config_after.stdout_lines

## Notes
# - This is a great read on error handling: https://docs.ansible.com/ansible/latest/user_guide/playbooks_error_handling.html
# - Valid variable names: https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html?highlight=register#creating-valid-variable-names

```

* Adhoc command to check what package manager would be used
```
ansible -m setup -a "filter=ansible_pkg_mgr" all
```

## Troubleshooting Inventory scripts
* If we have dynamic inventory, we can run the python script and very the output
* Example:

```
python3 centos.py 
{"_meta": {"hostvars": {"centos20": {"ansible_host": "192.168.50.20", "ansible_private_key_file": ".vagrant/machines/centos20/virtualbox/private_key"}, "centos21": {"ansible_host": "192.168.50.21", "ansible_private_key_file": ".vagrant/machines/centos21/virtualbox/private_key"}}}, "centos": {"hosts": ["centos20", "centos21"]}}
```

* For static inventory files we can use the ***graph** command and veiw the result.

```
ansible-inventory --graph --vars // list out variables
```

#### Listing all the types of inventory

```
ansible-doc -t inventory --list // about 31 of them using wc -l
```
