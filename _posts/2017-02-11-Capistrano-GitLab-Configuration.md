---
layout: post
title: "Capistrano Configuration on centos with gitlab"
categories: [Capistrano]
---

# Capistrano Configuration

## Configure Server

- configure **deploy** user 

```sh

groupadd deployers
adduser deploy
usermod -a -G deployers deploy
```
- to give the deployers group the permissions, run the following and edit the /etc/sudoers file: `visudo`
- Add the following line to after the groups:
```ssh 
..
## Allows people in group wheel to run all commands
%deployers      ALL=(ALL) ALL

..

```
- generate ssh-key for deploy user
```sh

su deploy
ssh-keygen -t rsa -b 2048
```
- add new deploy key to gitlab on **http://gitlabserver/admin/deploy_keys** from **new server** `cat ~/.ssh/id_rsa.pub`
- enable new deploy-key on gitlab project
- copy **gitlab-runner** key from gitlab-machine `su gitlab-runner -c "cat ~/.ssh/id_rsa.pub"`
- paste gitlab ssh-key to new server via **deploy** user `vi ~/.ssh/authorized_keys`
- change file mode `chmod 700 ~/.ssh/authorized_keys`
- install git on new server `yum -y install git`
- change project installation folder group `chgrp -hR deployers /var/www`
- change project installation folder permission `chmod 775 -R /var/www`
 


- optional `visudo` add `%deployers   ALL=(ALL)NOPASSWD:/bin/chown, /bin/chmod`


> first create capistrano config file with `cap install` command on project root folder.