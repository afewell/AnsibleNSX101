# Lab Toplogy Review

## Software Requirements

The following software is required to complete the lab excercises in this course. This course provides no access to any software, all users of this course are expected to provide their own licensed access to software and are responsible for ensuring their own compliance with licensing rules for the software.

Please Note that while the VMware Ansible Playbooks can be configured to work across a variety of different software versions, the specific versions of software listed here were used during the development of this course and are recommended to ensure the smoothest possible experience following the lab excercises.

### Mandatory Software:
-  VMware vCenter Server 6.0 & Platform Services Controller
  - This lab was developed using VMware vCenter Server Appliance 6.0.0.20000 with an embedded Platform Services Controller.
  - Other vCenter versions and form factors may also work, but the above version is the only one that has been tested for the lab excercises
- VMware ESXi 6.0.0
  - The specific release used in developing this course was VMKernel Release Build 3620759
- VMWare NSX Manager 6.2.5
  - Earlier versions of NSX should also work, but were not tested during the development of this course.
  - Ansible 2.0.0.2 (and required dependencies)
  - Ansible NSX Module (and required dependencies)

### Optional Software:
The following software provides services that are required for the lab excercises, but can be replaced by other software that provides equivalent capabilities if desired. This lab only provides instruction on the software listed below, if users choose to use alternate software, no instructions are provided on setup or configuration.
-  Ubuntu 16_04 Server
  - Used as the Ansible Control Server, Should be fine to use other Linux Distributions
- FreeNAS 9.3
  - Used to provide iSCSI services to provide storage back-end for vCenter Shared Storage. The lab excercises only interact with the vCenter shared storage front end, and do not directly interact with any back-end storage services. While the excercises do require shared storage, any supported mechanism to provide shared storage to vCenter environments should work fine.
-  Windows Server 2012
  -  In the lab excercises, Windows Server is used for several services:
    -  Active Directory & DNS Services
    -  IIS Services are used only to transfer the NSX Manager OVA file to the Ansible server. Any method of your choice can be used to transfer the file, none of the excercises depend on the method used so long as the file is present on the Ansible Control Server.
    -  Jump Host: I used a remotely accessed lab to develop the excercises, so I use a Windows Server in the lab as a jump host for accessing and testing connectivity to privately-addressed elements within the lab environment. Depending on how you setup your lab environment, you may not need a jump host. You will at minimum need IP connectivity to vCenter to access the vcenter web client, and need SSH access into the Ansible Server from the terminal you do the excercises on.

## Hardware (or virtual hardware) Requirements
This lab was built using nested virtualization, meaning the underlying physical server uses the ESXi Hypervisor, and on top of that hypervisor, virtual machines are installed with ESXi Hypervisor, which enables an additional layer of virtual machines to be installed on the virtual ESXi hosts.

While this lab was built with nested virtualization, it is possible to build functionally equivalent lab with physical servers or with standard virtualization. Keep in mind these alternatives may require the user to make adjustments to the lab excercises as this course only provides detailed instruction for the same environment in which it was developed.

A key difference with nested virtualization is that the virtual ESXi hosts become a functional equivalents of physical servers, yet retain the ease of management of VM's along with the ability to use virtualization to significantly minimize hardware requirements. For example, this lab uses the functional equivalent of 8 physical servers, yet because they are virtual machines, they are all running on a single physical server.

#### Nested Lab Considerations
If you are building your lab using nested virtualization, you will need a single physical server with enough hardware capacity to provision virtual machines that match the virtual ESXi host hardware configurations provided below. While specific instructions to install/configure nested virtualization are outside of the scope of this course, there are many blogs and articles on the internet the explain how to set it up.

#### Standard Virtual Lab Considerations
If you are building your lab with standard virtualization, keep in mind you will need an isolated vcenter environment as you should not attempt to do these labs on a production vCenter environment. You can however use ESXi to virtualize a physical server and then install vCenter as a virtual machine and set it up to manage the same ESXi host it is running on. The instructions in this lab will need be adjusted to support a Standard Virtual Lab environment, but it should be pretty straightforward to adapt.

#### Physical Lab Considerations
If you are building a lab using physical servers, provision a physical server to act as a funcitonal equivalent of each of the virtual esxi hosts listed below, with equal or greater hardware provisioned.

#### Minimizing Lab Hardware/Virtual Hardware requirements
Whether you are building a physical or a nested virtualization lab, it should be possible to reduce the amount of servers used in the lab if you need to minimize hardware requirements.

For example, the base lab topology uses 3 ESXi servers, but you could complete all the current lab excercises with a single ESXi physical or virtual host in the lab environment, although you would need to make any needed adjustments yourself.

Similarly you could examine the lab software requirements and attempt to reduce the virtual hardware resources. This lab environment is small and only uses few virtual machines, so software like NSX and vCenter should be very minimally taxed and could likely have hardware resources reduced and still function adequately for the excercises.


## Topology

## Base Lab Setup Information

Before you can proceed with the lab excercises in this course, you must first setup and configure base software elements beyond the hardware & topology setup. The base topology starts with vCenter, ESXi, Windows Server, FreeNAS 9.3 for iSCSI based shared storage (Optional), all pre-installed and setup with a basic configuration prior to the start of the lab excercises in this course. This section will provide information sufficient for users to setup an equivalent configuration, but it does not provide instructions on how to do the installation or configuration of the base lab elements.

-  Please Note: If any needed information is missing from this section, please submit an issue!

### Accounts & Passwords

### vDS configuration

### Shared Storage Information

### vCenter Configuration Details

### ESXi Configuration Details

### Windows Server Configuration Details
