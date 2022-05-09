Installing the operating system on HPE Proliant Servers with MAAS
=====================

![Visits Badge](https://badges.pufler.dev/visits/emrbaykal/maas-hpe)
![GitHub last commit](https://img.shields.io/github/last-commit/emrbaykal/maas-hpe)
[![GitHub license](https://img.shields.io/github/license/emrbaykal/maas-hpe)](https://github.com/emrbaykal/maas-hpe/blob/master/LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/emrbaykal/maas-hpe)](https://github.com/emrbaykal/maas-hpe/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/emrbaykal/maas-hpe)](https://github.com/emrbaykal/maas-hpe/network)
[![GitHub issues](https://img.shields.io/github/issues/emrbaykal/maas-hpe)](https://github.com/emrbaykal/maas-hpe/issues)

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
 
 Take In Action
---------------

- Before start working, infrastructure-specific and host-spesific variables must be filled .
   
   ## Infrastructure-Spesific Variables -- group_vars/all.yml 
      - The variables defined under this directory contain the connection and protocol information of the systems in the infrastructure.

   ## MAAS CLI Connection Parameters
   The following variable define Connection protocol, username and password information that ansible will use to access the server where maas cli is installed.
   ```yaml
   ansible_connection: ssh
   ansible_user: hpe
   ansible_ssh_pass: xxxxxx
   ```
   ## MAAS Server API & MAAS CLI Connection Parameters
   The following variable define maas cli ip address & maas server api url, username and api key information that ansible will use to access the server where maas rack controller and cli is installed.
   ```yaml
   # maas user
   maas_user: admin
   # maas api url
   maas_url: http://00:00:00:00:5240/MAAS
   # maas API Key
   maas_apikey: admin-api-key-sample
   # maas cli delegation server
   maas_cli_srv: maas-cli-server
   maas_cli_srv_ip: 00:00:00:00
   ```
  ## VMware Virtual Center Connection Parameters
   If there is vmware vcenter in your environment and virtual host deployment will be made on vmware, 
   the following VMware virtual center connection information must be filled.
   The following variable define VMware vcenter hostname, username , password and datacenter name informations.
   ```yaml
   # Vmware Virtual Center Info
   vcenter_hostname: 00:00:00:00
   vcenter_username: administrator@vsphere.local
   vcenter_password: xxxxxxx
   datacenter_name: VMware-Datacenter
   ```
  ## VMware Host Connection Parameters
   If there is vmware vcenter in your environment and virtual host deployment will be made on vmware, 
   the following esxi host whose deployment is planned host connection information must be filled.
   The following variable esxi username , password  informations.
   ```yaml
   # Planing to Deploy ESXI Host username & password Informations
   esxi_username: root
   esxi_password: xxxxx
   ```

  ## - Host-Spesific Variables -- host-vms-csv/hosts.csv
     - fgf


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
   - Pretask is a conditional execution block that runs before running main plays. These tasks do some prerequisite checks and validations.

```yaml
- Login to MAAS Server via using Maas CLI
- Checking whether the server is registered to maas.
- If the server is registered to maas, the status of the server is checked.
- If the server is registered to maas and its state is Deployed, 
  main plays will not be run for that server.

 ```

- Main Role Tasks:
  - HPE Proliant Server Preconfigure Role Tasks {{ maas-configure-physical-server }}
    - If the target server is HPE proliant Gen10 server, the following tasks are run before the operating system installation.

```yaml
- Check Proliant Server power state, powered on server if needed.
  - 01-power-state.yml
- Disk raid configuration is made to the server in the desired config.  
  - 02-logical-drives.yml
- Bios settings are made according to the application to be installed on the server.
  - 03-bios-settings.yml

```

  - Tasks used to register target servers to the maas server {{ maas-add-device }}
    - It registers the target server to the maas server, then the commissioning process is started.

```yaml
- If the target server is a virtual server running in vmware environment, 
  connect to Vmware Vcenter and  UUID information of the related virtual server is pulled.
  - 01-query-vm.yml
- If the target server is a virtual server running in vmware environment, 
  it is registered to the maas server together with the UUID information obtained in the previous task.  
  After the registration process, the commissioning state is started automatically.  
  - 02-add-virtual-device.yml
- if target server is HPE Proliant Server, it is registered to the maas server together with the server ILO informations.
  After the registration process, the commissioning state is started automatically.
  - 03-add-physical-device.yml

```

  - Network configuration of target server. {{ maas-configure-physical-server }}
    - Network configuration is done in line with the information filled in the csv file of the target server.

```yaml
- Checking whether the server is registered to maas.
- If the server is registered to maas, the status of the server is checked.
  - 01-query-machine-id-net.yml
- The settings of the public network interface are made.  
  - 02-conf-public-insterface.yml

```

  - Storage configuration of target server. {{ maas-configure-storage-layout }}
    - Storage configuration is done in line with the information filled in the csv file of the target server.
   

```yaml
- Checking whether the server is registered to maas.
- If the server is registered to maas, the status of the server is checked.
  - 01-query-machine-storage.yml
- Clean current storage layout
  - 02-clean-storage-layout.yml
- If the target server is desired to be set up as vmware host, to create vmfs storage layout.
  - 03-set-vmfs-storage-layout.yml
- If the target server is desired to be set up as Redhat Linux OS, to crate boot partition.
  - 04-create-boot-partition.yml
- If the target server is desired to be set up as Redhat Linux OS, to crate LVM Layout 
  - 05-create-volume-group.yml
- If the target server is desired to be set up as Redhat Linux OS, to crate SWAP Partition
  - 06-create-swap-logical-volume.yml
- If the target server is desired to be set up as Redhat Linux OS, to crate / Partition
  - 07-create-root-logical-volume.yml
- If the target server is desired to be set up as Redhat Linux OS, to crate /var Partition
  - 08-create-var-logical-volume.yml
- If the target server is desired to be set up as Redhat Linux OS, to crate /home Partition
  - 09-create-home-logical-volume.yml
- If the target server is desired to be set up as Redhat Linux OS, to crate /tmp Partition
  - 10-create-tmp-logical-volume.yml
```

  - Vmware Host Vcenter Registration. {{ maas-add-esxi-vcenter }}
    - If the target server is desired to be set up as vmware host, after the operating system installation, 
      the registration process is performed in the Vmware Virtual Center.

```yaml
- Add ESXi Host to vCenter.
  - 01-add-host-to-vcenter.yml

```
- Post Tasks 
   - Post is a conditional execution block that runs after running main plays. 

```yaml
- Logout to MAAS Server via using Maas CLI

 