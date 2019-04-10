# Table of contents
* [Introduction](#introduction)
* [Prerequisites](#prerequisites)
   * [Message Router](#message-router)
   * [Ansible Server](#ansible-server)
* [Ansible playbooks - a short introduction](#ansible-playbooks---a-short-introduction)
* [Overview of files](#overview-of-files)
   * [syslog.yml](#syslogyml)
   * [hosts.yml](#hostsyml)
   * [vars/syslog-test.yml](#varssyslog-testyml)
   * [files/show_current_config.cli](#filesshow_current_configcli)
   * [templates/syslog.tp.cli](#templatessyslogtpcli)
   * [roles/handle-syslogs/tasks/main.yml](#roleshandle-syslogstasksmainyml)
* [Running and testing the playbook](#running-and-testing-the-playbook)


# Introduction
There several possibilities of configuring a Solace message router via the cli:

1. interactively at the Solace cli prompt
2. by using the command source <script-name> at the cli prompt
3. by using the cli options. 


For Using Ansible we need the hidden options of the cli.
The sample code configures two syslog objects by using Ansible and the Solace Cli with its hidden options.
The big advantage of this cli approach is that you are able to do a complete router configuration. 
Using the SEMPv2 API - together with your prefered configuration management tool - allows only a partial configuration of the message router.


# Prerequisites
## Message Router
Before you can start applying the sample code you have to create a user on the Solace router, e.g. ansible.
This user has to be member of the sysadmin group.
Unlike the default admin user - which has the cli has the login program - the user ansible has /bin/bash as login program.
Furthermore the user should have a ~ansible/.ssh/authorized_keys file. 
You also need a SSH public key which fits to SSH private key you will use on the Ansible server. This public key has to be pasted into the authorized_keys file.
If you are using the Solace docker image you can create a Dockerfile that will do these adjustments.
Code was tested with the default solace Docker image release 9.0.0.17.

Docker hostname in the sample code is mysrv001. The Solace SSH port 2222 is only published to the service LAN interface of the docker host. 
We use the name mysrv001sv for this interface.
## Ansible Server
The sample configuration definitions have to reside on the Ansible management server. All code was tested on RHEL 7.3 and Ansible 2.7.9. The user on the  Ansible server should be able to contact the message router via ssh using port 2222. 
The following command must work
```ssh
ssh -p 2222 ansible@mysrv001sv
```
# Ansible playbooks - a short introduction
With Ansible playbook you can send commands to a solace message router using ssh.
Playbooks are written in YAML data serialization format. 
Each playbook is build of one or several tasks.
For configuring Solace objects we normally have to run some Ansible tasks, e.g.


* Transfer, Run and check current config 
* Build the cli script by using a template cli script
* Transfer the cli script to the message router
* ...

# Overview of files

## syslog.yml
This file ist the playbook. 
**_hosts: {{env}}_** defines the group of routers we want to configure. A group is defined in the ini file. In our example the ini file is **_hosts.yml_**.
We pass the value for the host key as an extra variable **_env_** with the ansible-playbook command using the **_-e option_**.
**_become: true_** means that all commands will be run through sudo, so the commands will be run as the root user.
We include the file **_vars/syslog-{{env}}.yml_** which contains all variables related to the syslog configuration for test group. 
Infos from the managed Solace router aren't necessary so we set **_gather_facts: false_**
We find one task in our playbook. But by using **_include_role_** further tasks inside the appropriate role subdirectory will run.
By using **_with_items_** we are looping the main task and we are able to two several syslog configurations.

## hosts.yml
Ansible works against multiple systems in your infrastructure at the same time.
It does this by selecting portions of systems listed in the inventory. In our example the inventory file is **_hosts.yml_**.
In this file we define the group *_test_** of message routers.
We have one router **_mysrv001sv_** in this group. Also we define 2 group specific vars.
**_ansible_port_** defines the SSH port we want to connect to.
The **_ansible_user_** variable conatins the name of the user we defined on the router

## vars/syslog-test.yml
In this file all variables for the syslog configuration are set. 
In this file also the variables **_status_** and **_overwrite_** are set. With their values you can define if an object is present or absent and how to handle an already existing syslog object.

## files/show_current_config.cli
This is the cliscript to get the current-config of the message router.

## templates/syslog.tp.cli
This ist the template for building the syslog cli-scripts. The jinja code and the variables allow a flexible build of the appropriate cli script.

## roles/handle-syslogs/tasks/main.yml
The tasks defined in this file are doing the syslog configuration:
* Copy the show_current_config.cli to the cliscripts directory if it not exists and execute it.
* Depending on the current configuration and the value of the status respective the overwrite variable the fact cliaction will be set.
* The cliscript based on the template syslog.tp.cli will be transfered to the message router. The name of the syslog object is part of the cliscript name.
* Finally the transfered cliscript will be excuted and the output will be checked.

# Running and testing the playbook
If you want to test the sample code just do the following steps:
* Replace the name mysrv001sv in hosts.yml
* Adjust the syslog configuration in vars/syslog-test.yml

Afterwards you can run the following command:
```playbook
ansible-playbook -e env=test  syslog.yml  -i hosts.yml
```
If the syslog objects defined in the syslog-test.yml do not exist they will be created if you set **_status: present_**
You can verify the creation of the syslog object at the solace prompt:

```show_syslog
show syslog * 
```
Afterwards you can change one port for mysyslog1
If **_overwrite: no_** is set the configuration will stay the same on the message router. Because mysyslog1 already exists and you defined the object will not be overwritten.
If you set **_overwrite: yes_** the old config will be updated by deleting the old object and rebuilding it with the same name.
If you run the playbook a second time with **_overwrite: yes_** without changing any value also the complete config will be deleted and rebuild. Updating an object this manner is not optimal. But it makes things much easier.
Checking all values of the current configuration against the desired values defined on you Ansible server is much more complicated
If you want to avoid overwriting an existing config than just set **_overwrite: no_**.
If you want to delete an object then set **_status: absent_**
