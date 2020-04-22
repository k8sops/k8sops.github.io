---
layout: page
title: Playbooks
category: ansible
permalink: /ansible/playbooks
chapter: 2
---

* Can we just write all the adhoc commands in one file and call it a day!!
![](images/scripted_adhoc.png)

* ***Not a GOOD*** option since every ansible call does the fact check and repeating that is inefficient.

## Two styles to define Tasks

1. Euqal To delimeted
![](ansible/images/equal_separated.png "equal to separated")

2. Colon separated
![](images/colon_separated.png "colon separated")

## Working with Playbooks
["working with playbooks"](https://docs.ansible.com/ansible/latest/user_guide/playbooks.html)