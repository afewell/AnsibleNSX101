# Ansible Installation and Configuration
This section will provide step-by-step instructions on the installation and configuration of Ansible, the NSX Modules for Ansible, and any required Dependencies.

## Before you begin
-  Please Be sure you have completed Labs 1a and 1b
-  You should be able to establish a remote desktop connection to the Main Console/Windows Server.
-  You should be able to establish an SSH session to the AnsibleCS server
-  The Windows and AnsibleCS servers should have internet access
-  You should have IP reachability between all hosts on the labs internal management (192.168.110.0/24) network
-  VMware Employees using OneCloud:
  -  You may load the vApp named "AnsibleNSX" to which is preloaded with all configurations provided in labs 1a-1c.
  -  If you would like to skip Ansible installation and proceed directly to using the NSX Ansible modules, another vApp called "AnsibleNSX_prepped" is available. If you load this vApp, you can proceed directly to Lab-2
  -  Both of these vApp Templates are located in the "US01-5 Sandbox-SDDC Internal" catalog
  - __Note:__  After you power on the vApp, please wait ~10 minutes before starting the lab exercises for everything to boot and connect.

## Ansible Installation

1. Open an SSH or other terminal session to the AnsibleCS server
  - For OneCloud and vCloud Director users, to find the external IP address of the AnsibleCS server, open you vApp and go to the networking tab, right click on the network and select "Configure Services". Select the NAT tab to find the external IP address
-  Update Ubuntu
  - `sudo apt-get update -y && sudo apt-get upgrade -y`
-  Install required dependencies for Ansible & NSX Ansible Modules
  -  `sudo apt-get install python python-dev python-pip -y`
  -  `sudo apt-get install build-essential libssl-dev libffi-dev libxml2-dev libxslt-dev python-dev zlib1g-dev python-openssl`
-  Add Ansible Personal Package Archive and Install Ansible
  -  `sudo apt-get install software-properties-common`
  -  `sudo add-apt-repository ppa:ansible/ansible`
  -  `sudo apt-get update`
  -  `sudo apt-get install ansible`
-  Install NSX RAML client (Required for NSX Ansible Module)
  -  `sudo pip install nsxramlclient --upgrade`
-  Install Python Client for vCenter (Required for NSX Ansible Module)
  -  `sudo pip install pyvmomi --upgrade`
-  Install VMware OVF Tool (Required for NSX Ansible Module)
  -  Download VMware OVF Tool and install on AnsibleCS Server. The reference lab used the Windows server to download OVF Tool from vmware.com, and used IIS on the windows server to transfer the OVF Tool file to the AnsibleCS server. It does not matter exactly how you get the OVF tool file to the AnsibleCS server as long as you get it there.
  - The following instructions assume the OVF Tool installer file has been downloaded to the windows server and IIS has already been configured. The following steps show how to transfer OVF tool from the Windows Server to the AnsibleCS server and then install it. Instructions on getting the OVF tool file and configuring IIS are not provided.
  - OneCloud Users can follow the following instructions exactly, for others you may use this as a reference, be sure to change any variables for your own environment as needed.
  - Download OVF Tool from the Windows (IIS) Server to the home directory:
    -  `cd ~`
    -  `wget 192.168.110.10/ovftool41.bundle`
    -  `chmod +x ovftool41.bundle`
    -  `sudo ./ovftool41.bundle`
    -  Accept license and follow prompts to complete installation
-  Clone the NSX Ansible and RAML repositories into the Ansible Modules directory
  -  `sudo mkdir /usr/share/my_modules`
  -  `cd /usr/share/my_modules`
  -  `sudo git clone https://github.com/vmware/nsxraml`
  -  `sudo git clone https://github.com/vmware/nsxansible`
## Congratulations, you have completed Lab 1!
## [Click Here To Proceed To Lab-2](../../Lab2-NSXDeploy/)
