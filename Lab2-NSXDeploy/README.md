# Deploying NSX Manager with Ansible
Lab 2 provides instructions on using Ansible to deploy NSX Manager and register it with SSO and vCenter. To complete this section, you will need to provide your own licensed access to NSX Manager software. The reference lab and OneCloud vApps for this course use VMware-NSX-Manager-6.2.5-4818372, however these instructions should also work with other recent/current versions of NSX Manager.

This Lab includes the following sections:

- [Prerequisites]()
- [Deploy NSX Manager]()
- [Register NSX Manager with vCenter]()
- [Register NSX Manger with vCenter Single Sign On]()

Please check the [NSX Ansible page on Github](https://github.com/vmware/nsxansible) for any updates to the NSX Ansible Modules. The lab materials in this course were developed and tested using only the versions of the software specified. 
## Prerequisites
- Prerequisites
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

## Deploy NSX Manager
This section provides instructions to create and execute a playbook to deploy the NSX Manager virtual appliance. 
 
### About the playbook
- Prepare a playbook for deploying NSX Manager
  - The [Offical Documentation for NSX Ansible](https://github.com/vmware/nsxansible) includes a sample playbook called deploynsx.yml [Click here to view file](https://github.com/vmware/nsxansible/blob/master/deployNsx.yml) which uses a sample Ansible Role to deploy NSX Manager. While you can use these files for reference, the use of Ansible Roles for NSX will be covered in section 7.
  - NSX Manager can be deployed with a single task, as shown in the playbook below

### Create the playbook
  - Create a playbook to deploy NSX Manager
    - The following commands create a new blank file called 'deploynsx.yml' and opens it in the vi text editor
    - `cd ~/nsxansible/`
    - `vi deploynsx.yml`
  - Edit the file to look like the example below:
    - Be sure to change any variables as needed for your environment
    - __OneCloud Users:__ your file should look exactly like below 
```
---
- hosts: localhost
connection: local
gather_facts: false
vars_files:
    - answerfile.yml
tasks:
- name: deploy nsx-man
    nsx_deploy_ova:
    ovftool_path: '/usr/bin'
    datacenter: 'RegionA01'
    datastore: 'RegionA01-ISCSI01-COMP01'
    portgroup: 'ESXi-RegionA01-vDS-COMP'
    cluster: 'RegionA01-COMP01'
    vmname: 'nsxmanager-01a'
    hostname: 'nsxmanager-01a.corp.local'
    dns_server: '192.168.110.10'
    dns_domain: 'corp.local'
    ntp_server: '192.168.100.1'
    gateway: '192.168.110.1'
    ip_address: '192.168.110.110'
    netmask: '255.255.255.0'
    admin_password: 'VMware1!'
    enable_password: 'VMware1!'
    path_to_ova: '~/'
    ova_file: 'nsxmanager.ova'
    vcenter: 'vcsa-01a.corp.local'
    vcenter_user: 'administrator@corp.local'
    vcenter_passwd: 'VMware1!'
  register: deploy_nsx_man
```
  - __Note:__ Ansible playbooks are saved in the YAML format. YAML format does not understand tabs for indentation, so each level of indentation is seperated by 2 spaces, not tabs. YAML doesnt care if you use exactly 2 spaces or 3 spaces (etc...) to indent, but it does matter that the indentation is consistent and each line is indented just like in the above example. I use 2 spaces for each level of indentation, which is a good general practice.

### Run the playbook

- Return to your terminal session with the Ansible server 
  - `cd ~/nsxansible/`
  - `ansible-playbook -i hosts deploynsx.yml`
  - After you run the above command, it will take a long time to run (~10 minutes or more) as it will transfer the large NSX Manager OVA file from the AnsibleCS server to the vCenter datastore. While the play is running, there will be no output on the screen to indicate if the play is still running, so you just have to wait until you the play completes successfully or fails.  
  - After you run the play, while the play is running you should see output similar to the following:
```
vmware@vmware:~/nsxansible$ ansible-playbook -i hosts deploynsx.yml

PLAY ***************************************************************************

TASK [deploy nsx-man] **********************************************************
```

  - The screen should remain just like above until the play is finished running. 
  - Once the play is finished running, you should see output similar to the following:
```
vmware@vmware:~/nsxansible$ ansible-playbook -i hosts deploynsx.yml

PLAY ***************************************************************************

TASK [deploy nsx-man] **********************************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=1    changed=1    unreachable=0    failed=0

vmware@vmware:~/nsxansible$ 
```
  - Ensure that when the play is completed, you see output similar to above, and the values under the "PLAY RECAP" section state "ok=1" and "changed=1", just like in the output above

### Verify results
- After the play runs successfully, open the vCenter web client and verify that the NSX Manager virtual machine has been deployed and powered on in your vCenter environment:
![NSX Deploy Validation](images/NsxDeployValidation.PNG)


## Register NSX Manager with vCenter
After deploying the NSX Manager virtual appliance, you must register it with vCenter. 

### About the playbook
- Prepare a playbook for deploying NSX Manager
  - The [Offical Documentation for NSX Ansible](https://github.com/vmware/nsxansible) includes a sample playbook called deploynsx.yml [Click here to view file](https://github.com/vmware/nsxansible/blob/master/deployNsx.yml) which uses a sample Ansible Role to deploy NSX Manager. While you can use these files for reference, the use of Ansible Roles for NSX will be covered in section 7.
  - NSX Manager can be deployed with a single task, as shown in the playbook below

## Create the Playbook
- Create a playbook to register NSX Manager with vCenter
  - `cd ~/nsxansible/`
  - `vi registerNsxVcenter.yml`
  - Edit the file to look like the example below:
```
---
- hosts: localhost
    connection: local
    gather_facts: False
    vars_files:
        - answerfile.yml
    tasks:
    - name: NSX Manager VC Registration
    nsx_vc_registration:
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        vcenter: 'vcsa-01a.corp.local'
        vcusername: 'administrator@vsphere.local'
        vcpassword: 'VMware1!'
        accept_all_certs: "True"
    register: register_to_vc

    - debug: var=register_to_vc
```
- In the above example file:
    - look at line for the variable nsxmanager_spec, notice that the value is "{{ nsxmanager_spec }}". 
    - Values like this that have the double curly brackets inside of quotes are how Ansible formats variables that are located in an external file.  The actual variables for nsxmanager_spec are in the 'answerfile.yml' file you created in Lab-1d. Notice that the above example lists the answerfile.yml as a vars_file - Ansible will look through all the files you list as vars_files to find values for any variables that use the double curly bracket format like the nsxmanager_spec in the example above.

### Run the playbook 
- Run  the play and review results
  - Return to your terminal session with the Ansible server
    - `cd ~/nsxansible`
    - `ansible-playbook -i hosts registerNsxVcenter.yml`
  - Note: Once you run the play, Ansible will execute the tasks in order and display the status of each tasks. After the first 4 tasks complete successfully, the `Create NSX Controllers` task will take a long time (15 minutes or more) to run, during which time no output will be displayed. During this time you simply must wait, if the task fails, you will recieve an error message. 
  - If the play completes successfully, you should see output similar to the following:
    - Note: Your output may not look exactly like the output below, for example in developing this lab I sometimes have to run the same play multiple times before I get it working correctly which may create differences between the output below and what you see in your terminal. The only thing that really matters is that there are no errors when running the play. To verify this, in the `PLAY RECAP` section, make sure all tasks executed as `ok` or `changed`. The other values should be `unreachable=0` and `failed=0`, if any tasks are unreachable or failed, you need to troubleshoot until you can run the play with no errors before proceeding. 

### Verify results

- Do something

## Register NSX Manager with the Single Sign-on Service
In addition to registering NSX Manager with vCenter, it must also be registered with the vCenter Single Sign On service. 

### Create the playbook
- Create a playbook to register NSX Manager with SSO
  - `cd ~/nsxansible/`
  - `vi registerNsxSso.yml`
  - Edit the file to look like the example below:
```
---
- hosts: localhost
    connection: local
    gather_facts: False
    vars_files:
        - answerfile.yml
    tasks:
    - name: NSX Manager SSO Registration
    nsx_sso_registration:
        state: present
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        sso_lookupservice_url: 'lookupservice/sdk'
        sso_lookupservice_port: 7444
        sso_lookupservice_server: 'vcsa-01a.corp.local'
        sso_admin_username: 'Administrator@vsphere.local'
        sso_admin_password: 'VMware1!'
        accept_all_certs: true
    register: register_to_sso

    - debug: var=register_to_sso
```
### Run the playbook
- Run  the play and review results
  - Return to your terminal session with the Ansible server
    - `cd ~/nsxansible`
    - `ansible-playbook -i hosts createControllers.yml`
  - Note: Once you run the play, Ansible will execute the tasks in order and display the status of each tasks. After the first 4 tasks complete successfully, the `Create NSX Controllers` task will take a long time (15 minutes or more) to run, during which time no output will be displayed. During this time you simply must wait, if the task fails, you will recieve an error message. 
  - If the play completes successfully, you should see output similar to the following:
    - Note: Your output may not look exactly like the output below, for example in developing this lab I sometimes have to run the same play multiple times before I get it working correctly which may create differences between the output below and what you see in your terminal. The only thing that really matters is that there are no errors when running the play. To verify this, in the `PLAY RECAP` section, make sure all tasks executed as `ok` or `changed`. The other values should be `unreachable=0` and `failed=0`, if any tasks are unreachable or failed, you need to troubleshoot until you can run the play with no errors before proceeding. 
```
vmware@vmware:~/nsxansible$ ansible-playbook -i hosts registersso.yml

PLAY ***************************************************************************

TASK [NSX Manager SSO Registration] ********************************************
changed: [localhost]

TASK [debug] *******************************************************************
ok: [localhost] => {
    "register_to_sso": {
        "argument_spec": {
            "accept_all_certs": true,
            "nsxmanager_spec": {
                "host": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
                "password": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
                "raml_file": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
                "user": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER"
            },
            "sso_admin_password": "VALUE_SPECIFIED_IN_NO_LOG_PARAMETER",
            "sso_admin_username": "Administrator@vsphere.local",
            "sso_certthumbprint": "FA:DA:47:9C:6B:B9:4C:6D:91:1F:84:17:2D:B7:66:1B:4D:BD:2A:49",
            "sso_lookupservice_port": 7444,
            "sso_lookupservice_server": "vcsa-01a.corp.local",
            "sso_lookupservice_url": "lookupservice/sdk",
            "state": "present"
        },
        "changed": true,
        "sso_config_response": {
            "Etag": null,
            "body": {
                "ssoConfigStatus": {
                    "message": "Done",
                    "status": "true"
                }
            },
            "location": null,
            "objectId": null,
            "status": 200
        }
    }
}

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0

vmware@vmware:~/nsxansible$
```
- Note that the output from running this command is a bit long and contains some detailed information about the changes that were made. The reason for the detailed output is because if you look at the ssoregistration.yml playbook, look at the last 2 lines of the play:
    - `register: register_to_sso`
    - This command creates a variable array called "register-to-sso" which keeps a record of the actions done within the task
    - Next the entry `debug: var=register_to_sso` causes ansible to display the values in the "register_to_sso" array.

### Verify results

- Do something

### Congratulations, you have completed Lab 2!
## [Click Here To Proceed To Lab-3](../../Lab3-Discovery/)
