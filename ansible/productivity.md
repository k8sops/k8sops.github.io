---
layout: page
title: Productivity
category: ansible
permalink: /ansible/productivity
chapter: 4
---

## Using the doc comamnd
```
ansible-doc -t become sudo

-t : is the plugin type. To find plugin type use ansible-doc -h

ansible-doc -h

-t TYPE, --type=TYPE  Choose which plugin type (defaults to "module").
                        Available plugin types are : ('become', 'cache',
                        'callback', 'cliconf', 'connection', 'httpapi',
                        'inventory', 'lookup', 'shell', 'module', 'strategy',
                        'vars')
```

## Reading the plugin code
* doc command returns the documentation right from the plugin source code on the system.
* We can always to the location and read the source code

```
ls /usr/local/Cellar/ansible/2.8.5_1/libexec/lib/python3.7/site-packages/ansible/plugins/shell/
__init__.py     cmd.py          fish.py         sh.py
__pycache__     csh.py          powershell.py
```

## Plugin code from GIT

* Git repo: [](https://github.com/ansible/ansible)
* press `t` and search is the file search
* git config can be done using git module: [](https://github.com/ansible/ansible/blob/devel/lib/ansible/modules/source_control/git.py)

```
ansible-doc git_config // note not passing -t, it is implicity added
```

## Non-Idempotent Modules
* Some modules are not idempotent like command & debug modules.

## VS Code Extension for Ansible by Microsoft

https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#ansible-command-shell-completion

## Ansible REPL Console

```
ansible-console
```

