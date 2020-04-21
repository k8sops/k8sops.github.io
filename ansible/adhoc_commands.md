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

