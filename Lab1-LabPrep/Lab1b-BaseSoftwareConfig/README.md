# Base lab software configuration
This section provides configuration information for the base lab software. The primary focus on this course is to provide instruction on the usage of Ansible NSX Modules to install and configure VMware NSX. Before excercises can begin focusing on NSX Ansible Modules, you must build and configure the base lab. This course does not provide detailed setup instructions for the base lab components, but will provide sufficient configuration details to enable you to replicate the lab environment.

__Before proceeding with this section, you should complete lab 1a and have your virtual-physical or physical hosts connected and powered on__

## Base IP Addressing

## Base Windows Server Configuration
Windows Server is installed with basic Active Directory and DNS services which are configured for the `corp.local` Users are free to use any domain they prefer but will need to adjust any lab exercises by replacing references to `corp.local` with whatever domain name is used in their lab.

1.  Install Windows Server 2012 R2
2.  Configure server to be primary domain controller for the `corp.local` domain.
3.  Configure DNS servers, ensure to include forwarders as needed to enable DNS resolution for both local and public internet name resolutions.
4. Create A Records for each of the virtual-physical layer hosts

## Base ESXi Host Configuration
For this lab, ESXi was installed with only basic configuration sufficient to connect to vCenter. You will need to complete these instructions for each ESXi host used in your lab environment.

1.  Install ESXi
  - You will be prompted for a password. The reference lab uses "VMware1!" for all passwords.
2. Connect to the ESXi terminal and configure the hostname and IP Address for the management network
3. Additional ESXi Configuration will be done after vCenter setup. Additional setup tasks will be provided below in the Base vCenter Server Configuration section.

## Base vCenter Server Configuration

1.  Begin the vcsa setup installer.
  -  The following installation variables were used in the reference lab - for any variable not listed here, fill in the appropriate setting for your environment:
    -  FQDN: `vcsa-01a.corp.local`
    -  Username: `Administrator@vsphere.local`
    -  Password: `VMware1!`
    -  Appliance Name: `vcsa-01a`
    -  OS user name: `root`
    -  OS Password: `VMware1!`
    -  Install vCenter Server with an Embedded Platform Services Controller
    -  Create a new SSO Domain
    -  vCenter SSO username: `administrator`
    -  vCenter SSO Password `VMware1!`
    -  SSO Domain Name: `vsphere.local`
    -  SSO Site Name: `Lab`
    -  Select Appliance Size: `Tiny`
    -  Use an embedded database
    -  Network Address: `192.168.110.22`
    -  Subnet Mask: `255.255.255.0`
    -  Default Gateway: `192.168.110.1`
    -  NTP - it is recommended you use NTP to synchronize time between all elements used in the lab. Installation and configuration of NTP are outside the scope of this course.
    -  Enable SSH: `True`
2. Configure VCSA for Active Directory Integrated Windows Authentication for the corp.local domain and ensure the Administrator@corp.local is added to the Administrators group in vCenter
3. Shared Datastore Requirements:
  -  1x 60 GB NFS
  - Mount point: 10.10.20.60:/mnt/NFSA, fdba:dd06:f00d:aa20::60:/mnt/NFSA
4. The following screen shots provide additional details needed for the base vCenter configuration. Step-by-step instructions are not provided, to replicate the base vcenter configuration ensure that you vCenter configuration settings match those shown in the screen shots:

![Hosts and Clusters](Images/BaseVcenterHostsClusters.PNG)


## Base Ansible Control Server Configuration





## [Click here to proceed to the next lab]()
