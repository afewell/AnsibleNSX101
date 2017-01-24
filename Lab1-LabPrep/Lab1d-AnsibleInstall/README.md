# Ansible Installation and Configuration
This section will provide step-by-step instructions on the installation and configuration of Ansible, the NSX Modules for Ansible, and any required Dependencies.

## Before you begin
-  Please Be sure you have completed Labs 1a and 1b
-  You should be able to establish a remote desktop connection to the Main Console/Windows Server.
-  You should be able to establish an SSH session to the AnsibleCS server
-  The Windows and AnsibleCS servers should have internet access
-  You should have IP reachability between all hosts on the labs internal management (192.168.110.0/24) network
-  VMware Employees using OneCloud:
  -  You may load the vApp named "AnsibleNSX" to which is preconfigured to include all configurations provided in labs 1a-1c.
  -  If you would like to skip Ansible installation and proceed directly to using the NSX Ansible modules, another vApp called "AnsibleNSX_prepped" is available. If you load this vApp, you can proceed directly to Lab-2
  -  Both of these vApp Templates are located in the "US01-5 Sandbox-SDDC Internal" catalog
  - __Note:__ I shutdown each host including the vcenter server and iSCSI server before saving the vApp template. Once you start and power on the vApp, please give it roughly 10 minutes before starting the lab exercises.

## Ansible Installation

1. Open an SSH session to the AnsibleCS server
  - For OneCloud and vCloud Director users, to find the external IP address of the AnsibleCS server, open you vApp and go to the networking tab, right click on the network and select "Configure Services". Select the NAT tab to find the external IP address
-  Update Ubuntu
  - `sudo apt-get update -y && sudo apt-get upgrade -y`
-
