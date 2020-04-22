---
layout: page
title: Adhoc Commands
category: ansible
permalink: /ansible/adhoc
chapter: 1
---

## Git Config use case

Setting user name and email in git config for making commits.

```
git config --global user.name username
git config --global --add user.email username@no-reply.com
```

* Very common user case and any system we start using. This has to be done.

* Can be automated using a shell script.

```
git-config.sh

git config --global user.name username
git config --global --add user.email username@no-reply.com

sh git-config.sh // does the job
```

## Idempotence

* This issue with using the shell script is that it doesn't take care of idempotence.

* we run it multiple times, git add will add duplicate entrys in git config file.

* Now we can make it idempotennt, but that would be a ***difficult undertaking***

```
// lame attempt to idempontence

if [[ -f ~/.gitconfig ]]; then
    isName = `cat ~/.gitconfig | grep 'user.name'`
    if [[ isName ]]; then
        // add config
        git config --global --add user.name "testuser"
    fi
fi
// see the difficulty???????????????
```

## Declarative & Re-conciliation

* Focusing on ***WHAT*** instead of ***HOW***

![alt text](images/reconciliation.png "declarative")


## Installing Ansible

```
pip install -U pip
pip install ansible
ansible --version

pip list --outdated
pip list --outdated --pre // releases like RC. not stable

pip install -U ansible // upgrading
```

## Porting guides
[porting docs](https://docs.ansible.com/ansible/latest/porting_guides/porting_guides.html)

## Ansible Architecture

![alt text](images/ansible_architecture.png "ansible architecture")

* Most common communication between control node & managed nodes happen over ***SSH***

But there are other options as well.

```
ansible-doc -t connection -l
```

![](images/different_managed_nodes.png "managed nodes")

## Ansible Doc command

```
ansible-doc -t connection -l
ansible-doc <plugin type flag> <name of plugin>  <list flag>
```

## Using Adhoc commands

* Typing ansible tab, gives following commands

```
ansible tab
ansible             ansible-connection  ansible-doc         ansible-inventory   ansible-pull        
ansible-config      ansible-console     ansible-galaxy      ansible-playbook    ansible-vault
```

* Adhoc commands are for ***one off*** tasks.

```
ansible -h / --help
```

* Module Index
[](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html)

* Git config use case with Ansible

![](images/git_config.png "git config use case" ) 
![](images/configuring_localhost.png =500x500 "configguring localhost")

* Copy Module

```
ansible -m copy -a "src=master.gitconfig dest=~/.gitconfig" localhost
localhost | CHANGED => {
    "changed": true,
    "checksum": "d0220965edf7c588257bc8a1b0bfe5c6cf758ab5",
    "dest": "/Users/someuser/.gitconfig",
    "gid": 20,
    "group": "staff",
    "md5sum": "cd67fef27efe2ac03fe4a77ce48a0f0f",
    "mode": "0644",
    "owner": "someuser",
    "size": 54,
    "src": "/Users/someuser/.ansible/tmp/ansible-tmp-1587554263.341763-97437170343070/source",
    "state": "file",
    "uid": 501
}
```

* Idempotence, with running the same command again

```
ansible -m copy -a "src=master.gitconfig dest=~/.gitconfig" localhost
localhost | SUCCESS => {
    "changed": false,
    "checksum": "d0220965edf7c588257bc8a1b0bfe5c6cf758ab5",
    "dest": "/Users/someuser/.gitconfig",
    "gid": 20,
    "group": "staff",
    "mode": "0644",
    "owner": "someuser",
    "path": "/Users/someuser/.gitconfig",
    "size": 54,
    "state": "file",
    "uid": 501
}
```

* Incases of ***drift*** where the file is present but contents has changed.
*** Ansible will correct it ****

* Check flag ***--check***
```
ansible -m copy -a "src=master.gitconfig dest=~/.gitconfig" localhost --check
localhost | CHANGED => {
    "changed": true
}
```

* Diff flag:

```
ansible -m copy -a "src=master.gitconfig dest=~/.gitconfig" localhost --diff
--- before: /Users/someuser/.gitconfig
+++ after: /Users/someuser/Documents/source/ansible/someuser/master.gitconfig
@@ -1,4 +1,4 @@
 [user]
-name = someuser123
+name = someuser
 email = someuser@gmail.com
```

* Can use both to check if a change is made & what change.

```
ansible -m copy -a "src=master.gitconfig dest=~/.gitconfig" localhost --check --diff
```