# Introduction
There several possibilities of configuring a Solace message router via the cli:

1. interactively at the solace cli prompt
2. by using the command source <script-name> at the cli prompt
3. by using the cli options. 


For Using Ansible we need the hidden options of the cli.
The sample code configures two syslog objects by using Ansible and the solace Cli with its hidden options.
The big advantage of this cli approach is that you are able to do a complete router configuration. 
Using the SEMPv2 API - together with your prefered configuration management tool - allows only a partial configuration of the message router.
