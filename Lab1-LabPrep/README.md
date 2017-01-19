# Preparing your lab environment
This section will provide detailed information on the hardware and software, topology and configuration of the lab environment used in this course. To follow along with the step-by-step instructions provided, it is recommended that users seek to replicate as close to the exact environment as possible for the smoothest experience with the lab excercises.

This guide will not provide detailed step by step instructions on the setup of the base lab environment, but will provide enough details for you to replicate the base environment in your own lab. Users with more Ansible experience can also adjust the playbooks in the labs to their own environment.

The lab was built using Nested Virtualization, but you can build a fully equivalent lab environment with physical servers, or you can use nested virtualization to simplify the hardware requirements.

The lab used to develop this course was built using VMware vCloud Director, however this is not needed to setup a fully equivalent lab. Lab-1a includes full software requirements. The only commercial software requirements are VMware vSphere and NSX. Windows Server 2012 is also used to provide Authention and DNS Services, however open source software could be used to provide equivalent functionality. This course only provides partial instructions on the setup of DNS services for Windows Server 2012.

Lab-1b provides detailed setup instructions to build the lab environment with vCloud Director. This course does not provide detailed setup instructions for building a lab outside of vCloud Director, however Lab-1a provides all of the requirements and information needed for users to setup their own lab.

## Sub-Sections:

-  [Lab-1a](Lab1a-TopologyReview/) Topology details, Hardware/software requirements
-  [Lab-1b](Lab1b-vCDSetup/) Building the base lab topology in vCloud Director
-  [Lab-1c](Lab1c-AnsibleInstall) Ansible and NSX Ansible Module Installation

## VMware Employees:
OneCloud Templates are available for you to use if you prefer that to building the environment from scratch.
-  The first vApp Template is named "AnsibleNSX" including an Ubuntu server ready to install ansible onto. If you use this template, complete Lab-1c prior to moving on to Lab 2
-  The second vApp Template is named "AnsibleNSX_prepped", and already has Ansible and NSX Modules installed. This is useful if you dont want to install ansible, or if you are returning to the lab to finish later excercices and dont need to repeat the install. If you use this template, you can skip all of Lab-1 and proceed to Lab-2.
-   Both of these vApp Templates are located in the "US01-5 Sandbox-SDDC Internal" catalog. If you do not have access to this catalog, you can open a ticket with the OneCloud Service Desk (Look for OCSN OneCloud Service Now in Workspace ONE to open ticket) to have the template moved to a partition you can access, or if you need further assistance, email me - afewell at vmware.
