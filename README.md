# Creating 2 Virtual Machines on AWS
* Ubuntu 22.04
* Red Hat Enterprise Linux 8

##### SSH into both VMs
```bash
ssh -i /path/key-pair-name.pem instance-user-name@instance-private-IPv4-address
```

# Installing Ansible
```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible

# boto3 lib for aws
sudo apt install python3-boto3
```

# Configuring Ansible
### 1. Create an ansible configuration file
```bash
touch ansible.cfg
```
##### which contains the following:
```
[defaults]
inventory = inventory

[inventory]
enabled_plugins = ini, aws_ec2
```
### 2. Create a new inventory directory with a host file
```bash
touch inventory/host
```
##### which contains the following:
```
[local]
localhost ansible_connection=local
```
### 3. Create a host.aws_ec2.yml file in the inventory directory
```bash
touch ./inventory/hosts.aws_ec2.yml
```
##### which contains the following:
```
---
plugin: aws_ec2
regions:
  - us-west-2
```
##### Display a graph of your inventory of managed hosts
```bash
$ ansible-inventory --graph
@all:
  |--@aws_ec2:
  |  |--ip-10-42-10-168.us-west-2.compute.internal
  |  |--ip-10-42-10-4.us-west-2.compute.internal
  |--@local:
  |  |--localhost
  |--@ungrouped:
```


##### Check that all managed nodes can be reached

```bash
$ ansible -m ping local
localhost | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```


# Creating Ansible playbook

## Create User Playbook
Creates a new regular user on both the Ubuntu and Red Hat Enterprise Linux VM's with the following:
1. A home directory in /home
2. A bash login shell
3. The ability to use sudo

```bash
touch user.yml
```

```YAML
---
- name: task1
  hosts: all
  tasks:
    - name: create regular user in ubuntu
      become: true
      ansible.builtin.user:
        name: ubun
        password: ""
        groups: sudo
        append: yes
        state: "present"
        shell: "/bin/bash"
        system: false
        create_home: true
        home: "/home/ubun"
      when: ansible_distribution in ["Ubuntu", "Debian"]
    - name: create regular user in red hat enterprise linux
      become: true
      ansible.builtin.user:
        name: redhat
        password: ""
        groups: wheel
        append: yes
        state: "present"
        shell: "/bin/bash"
        system: false
        create_home: true
        home: "/home/redhat"
      when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    - name: grab ./ssh from current user and copy to new user
      fetch:
        src: "~/.ssh"
        dest: "/home/ubun/"
        flat: yes
        owner: "ubun"
    - name: disable connection to server via ssh as the root user
      lineinfile:
        dest: /etc/ssh/sshd_config
        regexp: '^PermitRootLogin'
        line: "PermitRootLogin no"
        state: present
        backup: yes
      become: yes
      notify:
       - restart ssh
    - name: Start service sshd, if not started
      ansible.builtin.service:
        name: sshd
        state: started
    - name: Restart service sshd, in all cases
      ansible.builtin.service:
        name: sshd
        state: restarted
```

## Install Podman Playbook
Installs podman an open source container engine which is used for developing, managing and running container images.

```bash
touch podman.yml
```

```YAML
---
- name: install podman
   hosts: all
   become: yes
  tasks:
    - name: installing podman on ubuntu
      apt:
        name: podman
       state: present
      when: ansible_distribution == 'Debian' or ansible_distribution == 'Ubuntu'
    - name: installing podman on red hat enterprise linux
      yum:
        name: podman
        state: present
      when: ansible_distribution == 'CentOS' or ansible_distribution == 'Red Hat Enterprise Linux'
    - name: Pull httpd:2-alpine image from dockerhub.
       podman_image:
         name: docker.io/httpd
         tag: 2-alpine
    - name: Running httpd image.
        containers.podman.podman_container:
          name: my-first-container
          image:  docker.io/httpd:2-alpine
          state: started
```
