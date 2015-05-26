# docker 

Role to install and configure docker and docker-compose in centos 6.5


## Requirements

None.


## Role Variables

### Defined variables in vars
* epel_mirror
* compose_url
* compose_dest


## Dependencies

### Variable
* docker_user

## Example Playbook

    - hosts: servers
      roles:
         - { role: docker, sudo: yes }
