# gitlab_on_docker
## This is an example of how to use gitlab in docker with mounted volumes for backups of repo and configs.

### GITLAB install and run in Docker

Written By Joshua Loscar
May 2, 2018

#### Install docker

If you do not have docker installed,
You will need to install it on your host machine.

We will be using the Docker Community Edition (Docker CE).

You can learn how to install it from this page
https://www.docker.com/get-docker

I'll be installing this on a Debian linux box

```
apt-get install docker.io
```

Next we will pull the GITLAB/GITLAB-CE image from the docker repo
https://hub.docker.com/r/gitlab/gitlab-ce/

```
docker pull gitlab/gitlab-ce
```

To learn more about GITLAB you can go to this site
https://about.gitlab.com/installation/

Before we start the Docker Image, we want to design what volumes we will connect that.

Here is the documentation on backup and restore from Gitlab,

https://docs.gitlab.com/omnibus/settings/backups.html

We will want to backup "/etc/gitlab" to backup our configuration.

We will want to backup "/etc/ssh/" to backup the ssh keys of the users/developers.

We want to separate the back up for the Gitlab System (repos and metadata).

Depending on how you installed GITLAB, you will need to follow the instruction for your use case.

https://docs.gitlab.com/ce/raketasks/backup_restore.html#creating-a-backup-of-the-gitlab-system

I will be using the Docker instructions

```
docker exec -t <container name> gitlab-rake gitlab:backup:create

```
As an example here is a way to backup the configuration and secrets.

```
docker exec -t <your container name> /bin/sh -c 'umask 0077; tar cfz /secret/gitlab/backups/$(date "+etc-gitlab-\%s.tgz") -C / etc/gitlab'

```

So knowing this we need to have volumes mounted at /secret/gitlab/backups and /var/opt/gitlab in order to have these backups persisted outside the container.

There is also great documentation on how to restore gitlab to a cloud instance.
https://docs.gitlab.com/ce/raketasks/backup_restore.html#uploading-backups-to-a-remote-cloud-storage

So to start up a docker image with the volumes we use this command

```
sudo docker run -it --name gitlab -v /docker-volumes/gitlab/secret:/secret/gitlab/backups -v /docker-volumes/gitlab:/var/opt/gitlab gitlab/gitlab-ce /bin/bash
```

