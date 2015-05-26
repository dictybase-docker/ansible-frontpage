# Frontpage deployment with ansible
[Ansible](http://www.ansible.com) tasks for deploying dictybase
[frontpage](https://github.com/dictyBase/frontpage-dictybase) with
[docker](https://github.com/dictybase-docker/frontpage)

## Installation
Install ansible(>=1.9.1)

## Deployment

### Setup

* Create your `hosts` file. 

To start, copy the sample file.
```
cp hosts.sample hosts
```
Then edit accordingly. For a local vagrant machine use the `dicty-local` group
and fill up the `ansible_ssh_private_key_file` option. Generally it should be
found inside the `.vagrant` subfolder of folder where the `Vagrantfile` is
located.

* Setup two global variables in a file inside `group_vars` folder. The filename
  should match the group name inside the hosts file.

```
mkdir group_vars
```

So, for `dicty-local` group, create a file `group_vars/dicty-local` and setup `docker_user` and `image` variables.

```touch group_vars/dicty-local```

For local vagrant vm, use the following values

```
docker_user: vagrant
image: centos7
```

### End to End
It will a do a complete deployment from docker to running the final application, no other steps are needed.
Use the `frontpage.yml` playbook. For running on local vagrant vm

```
ansible-playbook -v -i hosts -l dicty-local frontpage.yml
```

For other hosts change the command line as neccessary.

### Docker only
It will do the basic setup and install docker only.
For running on local vagrant vm

```
ansible-playbook -v -i hosts -l dicty-local docker_install.yml
```

After that setup docker daemon to run through HTTPS as descrbied
[here](https://docs.docker.com/articles/https/).

Then clone [this](https://github.com/dictybase-docker/frontpage) repo, install docker-compose
and run docker-compose on the remote host. For local vagrant vm

```
DOCKER_HOST=tcp://127.0.0.1:9998 DOCKER_TLS_VERIFY=1 DOCKER_CERT_PATH=${HOME}/.docker docker-compose -f frontpage.yml up -d
```

