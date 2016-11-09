---
layout: pageshow
title: git to add remote origin in ubuntu and fix problems
category: git
description: a tutorial for git 
---

## preface

there is an error in my **ubuntu 16.04LTS** at input method, so before I fix it, I have to use Eng to write my blog temporarilly

## select git archive 

I am using [coding.net](coding.net) to git my source code. why not using [github](github.com), **github**has to pay for the private archives, and **coding.net** is free for that and also easy to use.

## add a SSH key 

below is refered to [coding.net help](https://coding.net/help/doc/git/ssh-key.html)

### generate a SSH key

1.the first step is to generate a SSH key locally, enter below in console

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

2.copy the content in `id_rsa.pub` to the website to add a SSH key

3.after SSH key is successfully added to the website, have a test to make the host website trust your instrument

```
ssh -T git@git.coding.net
```

if return the successful info, then you can add a remote origin to a specified git

```
git remote add origin https://git.coding.net/Username/ProjectName

git push
```

if encountered a problem, like:

```
sign_and_send_pubkey: signing failed: agent refused operation
```

this will failed our push, s solution is answered in [askubuntu](http://askubuntu.com/questions/762541/ubuntu-16-04-ssh-sign-and-send-pubkey-signing-failed-agent-refused-operation)
. It explains that `ssh-agent` is running already but it can not find any keys attached. Solve problem:

```
ssh-add
```

then, the problem is fixed.








