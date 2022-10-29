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
```bash
touch ansible.cfg
```
##### contains the following:
```
[defaults]
inventory = inventory

[inventory]
enabled_plugins = ini, aws_ec2
```
------
```bash
touch inventory/host
```
##### contains the following:
```
[local]
localhost ansible_connection=local
```
------
```bash
touch ./inventory/hosts.aws_ec2.yml
```
##### contains the following:
```
---
plugin: aws_ec2
regions:
  - us-west-2
```
## Display a graph of your inventory of managed hosts
```bash
ansible-inventory --graph
```
![alt text](https://github.com/nicasalazar/4640-pod1/blob/main/ansibele.JPG "Ansible Graph")

```YAML
config here
```
