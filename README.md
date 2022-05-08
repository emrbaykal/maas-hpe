Installing the operating system on HPE Proliant Servers with MAAS
=====================

The purpose of this project is to automatically install the desired operating system on HPE Proliant servers using ansible and maas software, without human intervention.

Canonical Maas software was used for operating system installation in the project. Ansible software was used to run the installation processes in a specific order and to make the necessary settings on the HPE Server and MAAS software for the operating system installation.

Requirements
------------

The following system requirements are needed in order for the written codes to work properly.

 - Maas Server (Region & Rack Controller)
   - You can find detailed information about Maas server setup from the link below.
      - [How to install MAAS](https://maas.io/docs/how-to-install-maas)
 - Ansible Core
   - You can find detailed information installing ansible core from the link below. You can install ansible where located on Maas servers operating system.
      - [Installing Ansible on Ubuntu](https://docs.ansible.com/ansible/2.9/installation_guide/intro_installation.html#installing-ansible-on-ubuntu) 
 - HPE Proliant Servers Gen10 & Gen10+ 
 -  You can download the github repository to the server with ansible installed with the following command.
    - git clone git@github.com:emrbaykal/maas-hpe.git
 

Task & Roles
------------

- Host Variable Role Tasks {{ role: host-variable }}
   - Using this role, the host variables are created by reading the data on the csv file created by the user.

```yaml
- Create host variable file
  - 01-host-var.yml
- Convigure ansible hosts file 
  - 02-hosts.yml

```

- Pre Tasks 
   - Pretask is a conditional execution block that runs before running main the plays. These tasks do some prerequisite checks and validations.

```yaml
- Login to MAAS Server via using Maas CLI
- Checking whether the server is registered to maas.
- If the server is registered to maas, the status of the server is checked.
- If the server is registered to maas and its state is Deployed, main plays will not be run for that server.

 ```
 Take In Action
 -----------

 