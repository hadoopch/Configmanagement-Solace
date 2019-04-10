# Introduction
There several possibilities of configuring a Solace message router via the cli:

1. interactively at the solace cli prompt
2. by using the command source <script-name> at the cli prompt
3. by using the cli options. 


For Using Ansible we need the hidden options of the cli.
The sample code configures two syslog objects by using Ansible and the solace Cli with its hidden options.
The big advantage of this cli approach is that you are able to do a complete router configuration. 
Using the SEMPv2 API - together with your prefered configuration management tool - allows only a partial configuration of the message router.

# Prerequisites
## Message Router
Before you can start applying the sample code you have to create a user on the Solace router, e.g. ansible.
This user has to be member of the sysadmin group.
Unlike the default admin user - which has the cli has the login program - the ansible user has /bin/bash as login program.
Furthermore the user should have a ~ansible/.ssh/authorized_keys file. 
You also need a SSH public key which fits to SSH private key you will use on the Ansible server. This public key has to be pasted into the authorized_keys file.
If you are using the solace docker image you can create a Dockerfile that will do these adjustments.
Code was tested with the default solace Docker image release 9.0.0.17.

Docker hostname is mysrv001. The Solace SSH port 2222 is only published to the service LAN interface of the docker host. 
We use the name mysrv001sv for this interface.
## Ansible Server
The sample configuration definitions have to reside on the ansible management server. All code was tested on RHEL 7.3 and ansible 2.7.9. The user on the  Ansible server should be able to contact the message router via ssh using port 2222. 
The following must work
ssh -p 2222 ansible@mysrv001sv
# Ansible playbooks - a short introduction
With Ansible playbook you can send commands to a solace message router using ssh.
Playbooks are written in YAML data serialization format. 
Each playbook is build of one or several tasks.
For configuring Solace objects we normally have to run some Ansible tasks, e.g.


* Transfer, Run and check current config 
* Build the cli script by using a template cli script
* Transfer the cli script to the message router
* ...

## Overview of files

## syslog.yml

This file ist the playbook. 
The hosts key define the group of routers we want to configure. A group defined in the ini file is used.
We pass the value for the host key as an extra variable with the ansible-playbook command
**_become: true_** means that all commands will be run through sudo, so the commands will be run as the root user.

we include a yaml file vars/syslog-{{env}}.yml which contains all variables related to the syslog configuration for this test 
We don't need the infos from the managed solace router so we set gather_facts to false
We find one task in our playbook. But with include_role further tasks inside the appropriate role subdirectory will run.
By using  with_items we are looping the main task and we are able to two several syslog configurations
