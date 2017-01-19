# Preparing your lab environment
This section will provide detailed information on the hardware and software, topology and configuration of the lab environment used in this course. To follow along with the step-by-step instructions provided, it is recommended that users seek to replicate as close to the exact environment as possible to replicate the lab excercises.

This guide will not provide detailed step by step instructions on the setup of the base lab environment, but will provide enough details for you to replicate the environment in your own lab.

The lab was built using Nested Virtualization, but you can build a fully equivalent lab environment with physical servers, or you can use nested virtualization to simplify the hardware requirements.

The lab used to develop this course was built using VMware vCloud Director, however this is not needed to setup a fully equivalent lab. Lab-1a includes full software requirements. The only commercial software requirements are VMware vSphere and NSX. Windows Server 2012 is also used to provide Active Directory and DNS Services, however open source software could be used to replicate this functionality. This course only provides partial instructions on the setup of Authentication and DNS services for Windows Server 2012.

Lab-1b provides detailed setup instructions to build the lab environment with vCloud Director. This course does not provide detailed setup instructions for building a lab outside of vCloud Director, however Lab-1a provides all of the requirements and information needed for users to setup their own lab.

## Sub-Sections:

- Lab-1a Topology details, Hardware/software requirements
- Lab-1b Building the base lab topology in vCloud Director
- Lab-1c Ansible and NSX Ansible Module Installation
