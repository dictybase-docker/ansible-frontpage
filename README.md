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

For example, if the `Vagrantfile` is in `/home/caboose/vms`, then the
private key would be in

`/home/caboose/vms/.vagrant/machines/default/virtualbox/private_key`

* Setup one global variables in a file inside `group_vars` folder. The filename
  should match the group name inside the hosts file.

```
mkdir group_vars
```

So, for `dicty-local` group, create a file `group_vars/dicty-local` and setup `docker_user` variable.

```touch group_vars/dicty-local```

For local vagrant vm, use the following value

```
docker_user: vagrant
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

### Docker with https
#### Generating certificates
You need the following files to get it working

1. A certificate authority private key
2. Generate a certificate authority(CA) using the above private key 
3. A private key for server
4. A certificate signing request for server
5. Generate a signed certificate for server using the previous signing request 
   and the CA from step 2.
   Make sure you use a config file and add the IP address of your server to get
   included in the certificate. For a vagrant host as an example, use 127.0.0.1(localhost).
   This is very important.
6. A private key for client.
7. A certificate signing request for client.
8. Generate a signed certificate for client using the previous(7) signing request
   and the CA from step 2.

The [documentation](https://docs.docker.com/articles/https/) for setting up docker with HTTPS 
shows how to generate all of the above files. However, only skip the step for setting up the docker daemon,
the ansible playbook below takes care of it.

#### Ansible and deploy
Setup the following variables in your `group_vars` files. For vagrant, use the 'dicty-local` file. Use the full. 
path for the location of the files that were generated in the certificate generation steps.

```
# certificate authority
tlscacert: # full path
# signed certificate for server
tlscert: # full path
# private key for server
tlskey: # full path

# remote folder under /etc/ssl where the tls files will be copied
docker_tls_path: /etc/ssl/docker_localhost
# The above is for vagrant localhost. Change for other hosts accordingly.
```

Run the playbook `docker_install_tls.yml`. For local vagrant host,

```
ansible-playbook -v -i hosts -l dicty-local docker_install_tls.yml
```

The above will setup vagrant with docker daemon with HTTPS running on port 2376. The daemon
is available on port 9998 on the host. Check the daemon with docker command

```
DOCKER_HOST=tcp://127.0.0.1:9998 DOCKER_TLS_VERIFY=1 docker version
```

It expects the client certificates are in `$HOME/.docker`. In case in different location, use
`DOCKER_CERT_PATH` to change the default.

Then clone [this](https://github.com/dictybase-docker/frontpage) repo, install docker-compose
and run docker-compose on the remote host. For local vagrant vm

```
DOCKER_HOST=tcp://127.0.0.1:9998 DOCKER_TLS_VERIFY=1 docker-compose -f frontpage.yml up -d
```

