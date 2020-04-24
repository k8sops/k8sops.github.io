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