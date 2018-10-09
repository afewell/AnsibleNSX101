# Preparing your lab environment
This section will provide detailed information on the hardware and software, topology and configuration of the lab environment used in this course.

The lab was built using Nested Virtualization, but you can build a fully equivalent lab environment with physical servers, or use nested virtualization to simplify the hardware requirements.

The lab used to develop this course was built using VMware vCloud Director, however this is not needed to setup a fully equivalent lab. Lab-1a includes full software requirements needed to complete the lab exercises.

This course does not provide detailed setup instructions for building a lab outside of vCloud Director, however Lab-1a provides all of the requirements and information needed for users to setup their own lab.

## Sub-Sections:

-  [Lab-1a](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep/Lab1a-TopologyReview) Topology details, Hardware/software requirements
-  [Lab-1b](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep/Lab1b-BaseSoftwareConfig) Base lab software configuration
-  [Lab-1c](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep/Lab1c-vCDSetup) Base lab - additional information for vCloud Director environments
-  [Lab-1d](https://github.com/afewell/AnsibleNSX101/tree/master/Lab1-LabPrep/Lab1c-vCDSetup) Ansible and NSX Ansible Module Installation

## VMware Employees:
OneCloud Templates are available for you to use if you prefer that to building the environment from scratch.
-  The first vApp Template is named "AnsibleNSX" including an Ubuntu server ready to install ansible onto. If you use this template, complete Lab-1d prior to moving on to Lab 2
-  The second vApp Template is named "AnsibleNSX_prepped", and already has Ansible and NSX Modules installed. This is useful if you dont want to install ansible, or if you are returning to the lab to finish later excercices and dont need to repeat the install. If you use this template, you can skip all of Lab-1 and proceed to Lab-2.
-   Both of these vApp Templates are located in the "US01-5 Sandbox-SDDC Internal" catalog. If you do not have access to this catalog, you can open a ticket with the OneCloud Service Desk (Look for OCSN OneCloud Service Now in Workspace ONE to open ticket) to have the template moved to a partition you can access, or if you need further assistance, email me - afewell at vmware.
