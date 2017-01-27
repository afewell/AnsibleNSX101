## Deploying NSX Manager with Ansible
This section will provide instructions on using Ansible to deploy NSX Manager and register it with SSO and vCenter. To complete this section, you will need to provide your own licensed access to NSX Manager software. The reference lab and OneCloud vApps use VMware-NSX-Manager-6.2.5-4818372, however these instructions should also work with other recent/current versions of NSX Manager.

Please check the [NSX Ansible page on Github](https://github.com/vmware/nsxansible) for any updates to the NSX Ansible Modules - I do try to keep this course updated, but future updates to this course will likely come well after any updates to the modules are made.

1. Prerequisites
  - Complete Lab 1
    - Or for OneCloud Users
        - If you load the vApp "AnsibleNSX_Prepped", this is the correct point to start with the lab exercises.
        - If you load the vApp "AnsibleNSX", you need to complete Lab-1d prior to starting this section
  - Please review the documentation for the modules used in this section:
    - [Module nsx_deploy_ova](https://github.com/vmware/nsxansible#module-nsx_deploy_ova)
    - [Module nsx_vc_registration](https://github.com/vmware/nsxansible#module-nsx_vc_registration)
    - [Module nsx_sso_registration](https://github.com/vmware/nsxansible#module-nsx_sso_registration)
  - Make sure there is an A record in the DNS server for the FQDN you would like to use for the NSX Manager server.
    - In the reference lab,
  - Save NSX Manager OVA on the AnsibleCS Server
    - To complete this lab, the NSX Manager OVA file needs be saved locally on the AnsibleCS server.
    - In the reference lab, IIS is used on Windows Server to transfer the file to AnsibleCS. This course does not include instruction on downloading the NSX Manager OVA File or on the setup of IIS.
    - To complete this section, it doesnt matter how you get the NSX Manager OVA file to the AnibleCS server as long as you get it there.
    - __OneCloud Users:__
      - The NSX Manager OVA File is already loaded onto both of the vApps prepared for this course, and IIS is already pre-configured.
      - You will need to download the NSX Manager OVA file to the AnsibleCS Server per the following instructions:
          - `cd ~`
          - `wget 192.168.110.10/nsxmanager.ova`
- Prepare the NSX Ansible environment
  - Please refer to the official documentation for [How to use these modules](https://github.com/vmware/nsxansible#how-to-use-these-modules) for additional details.
  - The NSX Ansible Modules work by communicating directly from the AnsibleCS server to the vCenter and NSX Manager REST API's.
    - There is no need to install Ansible on any target hosts or perform key exchanges for the playbooks used in this course.
    - The only host needed in the Ansible inventory is localhost. The exercises in this course specify localhost directly within each playbook, but if you prefer you can create an entry for localhost in your Ansible Inventory file.
  - Create a directory you will use to hold the playbook and variable files you will create in this course:
    - `cd ~`
    - `mkdir nsxansible`
    - `cd nsxansible`
  - Create a subdirectory to hold the example playbook files and copy the files
    - `mkdir examples`
    - `cp /usr/share/my_modules/nsxansible/*.yml ./examples/`
  - Copy the example host file to your home nsxansible directory
    - `cp /usr/share/my_modules/nsxansible/hosts .`
  - Verify the hosts file
    - `cat hosts`
    - The File should look like this:


  ```
  [localhost]
  localhost       ansible_connection=local

  [jumphost]
  10.114.209.37   ansible_ssh_user=localadmin ansible_ssh_private_key_file=~/.ssh/ansible_key
  ```

  - Note:
    - The exercises in this course only use the localhost entry, but the sample host file also includes an example of how to specify a jumphost. It is common in production Ansible deployments to use jumphosts to execute plays.
  * dd