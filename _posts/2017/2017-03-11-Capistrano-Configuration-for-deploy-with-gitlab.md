---
layout: post	
title: Capistrano configuration for deploy with gitlab
published: true
categories: [Capistrano]
date: 2017-03-11 10:45:35
excerpt: | 
    Capistrano is written in Ruby, but it can easily be used to deploy any language.

    If your language or framework has special deployment requirements, Capistrano can easily be extended to support them.
---

## Capistano

Capistrano is written in Ruby, but it can easily be used to deploy any language.

If your language or framework has special deployment requirements, Capistrano can easily be extended to support them.


## Configure Server

- configure **deploy** user
 
```ssh 
sudo groupadd deployers
sudo adduser deploy
sudo usermod -a -G deployers deploy
```
- to give the deployers group the permissions, run the following and edit the /etc/sudoers file: `visudo`
- Add the following line to after the groups:

```ssh 
..
## Allows people in group wheel to run all commands
%deployers      ALL=(ALL)NOPASSWD:/bin/chown, /bin/chmod

..
```

- generate ssh-key for deploy user

```ssh
sudo su - deploy
ssh-keygen -t rsa -b 4096 -C "deploy@example.com"
```

- add new deploy key to gitlab on **http://gitlabserver/admin/deploy_keys** from **new server** `cat ~/.ssh/id_rsa.pub`
- enable new deploy-key on gitlab project
- copy **gitlab-runner** key from gitlab-machine `su gitlab-runner -c "cat ~/.ssh/id_rsa.pub"`
- paste gitlab ssh-key to new server via **deploy** user `vi ~/.ssh/authorized_keys`
- change file mode `chmod 644 ~/.ssh/authorized_keys`
- install git on new server `yum -y install git`
- change project installation folder group `chgrp -hR deployers /var/www`
- change project installation folder permission `chmod 775 -R /var/www`
- optional `visudo` add `%deploy   ALL=(ALL)NOPASSWD:/bin/chown, /bin/chmod`

> first create capistrano config file with `cap install` command on project root folder.

