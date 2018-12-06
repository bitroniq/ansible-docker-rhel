# ansible-docker 

> Ansible Playbook for automatic installation and configuration of Docker and Docker-compose

## Main features:
 
* ✔ Automatic intallation of requirements
* ✔ Automatic installation of missing packagesa
* ✔ Automatic update of system packages
* ✔ Ubuntu 18.04+ compatibility
* ✔ CentOS 7+ compatibility
* ✔ Automatic Docker and Docker Compose installation in latest version

## Prerequisites

To use the Ansible Playbook for setup of Docker and Docker-compose the following prerequisites must be installed:

- Make sure the Host OS has public IP
- Make sure it is possible to acces via SSH

## Usage

Having all Prerequisites set up please follow the next steps to configure environment.

### Clone repository
```
git clone https://github.com/bitroniq/ansible-docker.git
```

### Install Ansible Galaxy Roles

Before Ansible Playbook can be played on target host (Ubuntu 18.04+ or CentOS 7+), some roles from Ansible Galaxy must be installed.

```
ansible-galaxy install -r requirements.yml
```

### Edit inventory file

Edit the inventory file with favorite editor:
```
vim inventory.yml
```

Replace hostname with the IP address of target host (guest OS):
```
---
all:
  hosts:
    IP_ADDRESS_OF_GUEST_OS:
```

### Play the Ansible Playbook (site.yml) to configure target host

On the local host execute the command from the listing below. Ansible will ask for:
 - user password: `PASSWORD`
 - sudo password: `PASSWORD`

```
ansible-playbook -i inventory.yml site.yml --ask-pass --ask-su-pass -u ec2-user
```

After few minutes all packages should be installed and the system should be configured accordingly.

Installation logs:
```
$ ansible-playbook -i inventory.yml site.yml --ask-pass --ask-su-pass -u ec2-user
SSH password:
SUDO password[defaults to SSH password]:

PLAY [all] ****************************************************************************************

TASK [Gathering Facts] ****************************************************************************
ok: [172.16.75.19]

TASK [Update packages] ****************************************************************************
ok: [172.16.75.19]

TASK [Install python for Ansible] *****************************************************************
changed: [172.16.75.19]

TASK [Gathering Facts] ****************************************************************************
ok: [172.16.75.19]

[...]
[...]

PLAY RECAP ****************************************************************************************
172.16.75.19               : ok=36   changed=21   unreachable=0    failed=0

```

## Manage Docker as a non-root user

Ansible will take care of adding user `ec2-user` to group `docker`.
But it is needed to log out and log back in so that your group membership is re-evaluated.

If testing on a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.

On a desktop Linux environment such as X Windows, log out of your session completely and then log back in.

Verify that you can run docker commands without sudo.

`$ docker run hello-world`

This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.

If you initially ran Docker CLI commands using sudo before adding your user to the docker group, you may see the following error, which indicates that your `~/.docker/` directory was created with incorrect permissions due to the sudo commands.

```
WARNING: Error loading config file: /home/user/.docker/config.json -
stat /home/user/.docker/config.json: permission denied
To fix this problem, either remove the ~/.docker/ directory (it is recreated automatically, but any custom settings are lost), or change its ownership and permissions using the following commands:
$ sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
$ sudo chmod g+rwx "$HOME/.docker" -R
```

---
## How it works and the content of repository

> Information for developers

### Content

The repository contains the following files and directories:
```
├── docker-centos-7
│   └── Dockerfile
├── docker-ubuntu-18.04
│   ├── Dockerfile
│   ├── initctl_faker
│   └── README.md
├── common
│   └── tasks
│       ├── debian.yml
│       ├── main.yml
│       └── redhat.yml
├── inventory.yml
├── README.md
├── requirements.yml
├── site.yml
├── tests
│   ├── inventory
│   ├── test.yml
│   └── vagrant.yml
├── vagrant-multi
│   └── Vagrantfile
└── vagrant-single
    └── Vagrantfile
```

* `docker-centos-7` - used for building docker container with CentOS 7 to test the Playbook
* `docker-ubuntu-18.04` - used for building docker container with Ubuntu 18.04 to test the Playbook
* `common` - the main role for installing packages
* `inventory.yml` - inventory file where the target hosts should be defined
* `README.md` - This readme file
* `requirements.yml` - List of Ansible Galaxy roles
* `site.yml` - Main Playbook that lists the roles to be played on target host
* `tests` - Playbook that is used by Vagrant to test the playbooks on different Linux versions
* `vagrant-multi` - Vagrantfile that will run many different Linux versions to test the Playbook
* `vagrant-single` - Vagrantfile that will run single machine


### How it works

`site.yml` is the main Playbook that contains the list if variables and roles to be played on target host.

First the `pre_tasks` are played to update packages on target host(s) and to install Python needed by Ansible on both local host and target host.

Ansible will also gather facts about the target host(s):
```
    - name: Gathering Facts
      setup:
```
This is needed to determine the target OS version etc.

Then the roles will be played:
```
  roles:
    # Play role common with base packages
    - role: common

    # Play role geerlingguy.docker to install docker and docker-compose
    - role: geerlingguy.docker

    # Play role geerlingguy.java to install java for pycharm
    - role: geerlingguy.java
      when: "ansible_os_family == 'Debian'"
      java_packages:
        - openjdk-8-jdk

    # Play role oefenweb.pycharm to install pycharm
    - role: oefenweb.pycharm
```
* `common` will install the basic packages:
  - name: Ensure git is present
  - name: Ensure vim is present
  - name: Ensure mc is present
* `geerlingguy.docker` - Ansible Galaxy role to install latest Docker - maintained by the community
* `geerlingguy.java` - Ansible Galaxy role to install Java - maintained by the community

---
## Contributing

1. Fork it (<https://github.com/bitroniq/ansible-docker/fork>)
2. Create your feature branch (`git checkout -b feature/fooBar`)
3. Commit your changes (`git commit -am 'Add some fooBar'`)
4. Push to the branch (`git push origin feature/fooBar`)
5. Create a new Pull Request

## References:

* https://galaxy.ansible.com/geerlingguy/java
* https://galaxy.ansible.com/geerlingguy/docker
* https://galaxy.ansible.com/oefenweb/pycharm
* https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html
