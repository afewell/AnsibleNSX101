# Ansible Installation and Configuration
This section will provide step-by-step instructions on the installation and configuration of Ansible, the NSX Modules for Ansible, and required Dependencies.

## Before you begin
-  Please Be sure you have completed Labs 1a and 1b
-  You should be able to establish a remote desktop connection to the Main Console/Windows Server.
-  You should be able to establish an SSH session to the AnsibleCS server
-  The Windows and AnsibleCS servers should have internet access
-  You should have IP reachability between all hosts on the labs internal management (192.168.110.0/24) network from the Ansible control server.

### VMware Employees using OneCloud:
  - A vApp named "AnsibleNSX" is available in OneCloud that is preloaded with all configurations provided in labs 1a-1c allowing you to start here with lab-1d which walks you through the installation and setup of the Ansible Control Server. **Please Note:** the AnsibleNSX  vApp is experiencing issues. For now please use the "AnsibleNsx_Prepped" vApp. The site will be updated once the problem is fixed.
  - If you would like to skip Ansible installation and proceed directly to using the NSX Ansible modules, another vApp called "AnsibleNSX_prepped" is available. If you load this vApp, you can [proceed directly to Lab-2](https://github.com/afewell/AnsibleNSX101/tree/master/Lab2-NSXDeploy#prerequisites)
  - Both of these vApp Templates are located in the "US01-5 Sandbox-SDDC Internal" catalog
  -  If you do not have access to the Sandbox-SDDC partition, please open a ticket with the OneCloud support team to have the vApp moved to a partition you can access. 
  - For OneCloud and vCloud Director users, to find the external IP address of the AnsibleCS server, open your vApp in vCloud Director and go to the networking tab, right click on the "vAppNet-single" network and select "Configure Services". Select the NAT tab to find the external IP addres
  - __Note:__  After you power on the vApp, please wait ~10 minutes before starting the lab exercises for everything to boot and connect. This lab does not make use of the lab startup script so the watermark on the desktop will not indicate when the lab is ready. To verify lab readiness, open a web browser connection to the vSphere web client. Once you can log in successfully to the vSphere web client, the lab is ready to proceed.  

## Ansible Installation
- Open an SSH or other terminal session to the AnsibleCS server
  - For OneCloud and vCloud Director users, to find the external IP address of the AnsibleCS server, open your vApp in vCloud Director and go to the networking tab, right click on the "vAppNet-single" network and select "Configure Services". Select the NAT tab to find the external IP address
    - Username: vmware
    - Password: VMware1!
-  Update Ubuntu
  - `sudo apt-get update -y && sudo apt-get upgrade -y`
-  Install required dependencies for Ansible & NSX Ansible Modules
  -  `sudo apt-get install python python-dev python-pip -y`
  -  `sudo apt-get install build-essential libssl-dev libffi-dev libxml2-dev libxslt-dev python-dev zlib1g-dev python-openssl -y`
-  Add Ansible Personal Package Archive and Install Ansible
  -  `sudo apt-get install software-properties-common -y`
  -  `sudo add-apt-repository ppa:ansible/ansible`
  -  `sudo apt-get update -y`
  -  `sudo apt-get install ansible -y`
-  Install [NSX RAML client](http://github.com/vmware/nsxramlclient) (Required for NSX Ansible Module)
  -  `sudo pip install nsxramlclient --upgrade`
-  Install [Python Client for vCenter](http://github.com/vmware/pyvmomi) (Required for NSX Ansible Module)
  -  `sudo pip install pyvmomi --upgrade`
-  Install [VMware OVF Tool](https://www.vmware.com/support/developer/ovf/) (Required for NSX Ansible Module)
  -  Download VMware OVF Tool and install on AnsibleCS Server. The reference lab used the Windows server to download OVF Tool from vmware.com, and used IIS on the windows server to transfer the OVF Tool file to the AnsibleCS server. It does not matter exactly how you get the OVF tool file to the AnsibleCS server as long as you get it there.
  - The following instructions assume the OVF Tool installer file and the MSX Manager virtual appliance are stored locally on the windows server and IIS has already been configured to share those files via HTTP. The following steps show how to transfer OVF tool from the Windows Server to the AnsibleCS server and then install it. Instructions on getting the OVF tool file and configuring IIS are not provided. It does not matter how you get the ovftool and NSX Manager files to the Ansible server so long as you get them there. 
  - OneCloud Users can follow the following instructions exactly, for others you may use this as a reference, be sure to change any variables for your own environment as needed.
  - Download OVF Tool from the Windows (IIS) Server to the home directory:
    -  `cd ~`
    -  `wget 192.168.110.10/ovftool41.bundle`
    -  `chmod +x ovftool41.bundle`
    -  `sudo ./ovftool41.bundle`
    -  Accept license and follow prompts to complete installation
  - Download NSX Manger OVA File from the Windows (IIS) Server to the home directory:
    -  `cd ~`
    -  `wget 192.168.110.10/nsxmanager.ova`
- Clone the NSX Ansible and RAML repositories into your home directory
   -  `cd ~`
   -  `sudo git clone https://github.com/vmware/nsxraml`
   -  `sudo git clone https://github.com/vmware/nsxansible`
-  Assign ownership of the ~/nsxansible directory to your account
   -  `cd ~`
   -  `sudo chown nsxansible/`

### Prepare the NSX Ansible Environment
- Prepare the NSX Ansible environment
  - Please refer to the official documentation for [How to use these modules](https://github.com/vmware/nsxansible#how-to-use-these-modules) for additional details.
- The NSX Ansible Modules work by communicating directly from the AnsibleCS server to the vCenter and NSX Manager REST API's.
- There is no need to install Ansible on any target hosts or perform key exchanges for the playbooks used in this course.
- Prepare the directory you will use to hold the playbook and variable files you will create in this course:
    - In this course the examples use the ~/nsxansible directory to store and execute Ansible plays. When the NSX Ansible modules were downloaded onto the Ansible server from github, it created this folder  and loaded it with a large number of example files. You can see these by executing the `ls` command from the `~/nsxansible/` directory. 
    - To keep the files you build in this course seperate from the examples included in the nsxansible repository, create an examples folder and move these files
    - Open a terminal or ssh connection to the Ansible control server
      - Username: `vmware`
      - Password: `VMware1!`
    - `cd ~/nsxansible`
    - `mkdir ~/nsxansible/examples`
    - `mv *.yml ~/nsxansible/examples/`
    - Now if you execute an `ls` you should see only a few files in the ~/nsxansible directory
    - Verify the hosts file
    - `cat ~/nsxansible/hosts`
    - The output should look like this:
    
  ```
  [localhost]
  localhost       ansible_connection=local

  [jumphost]
  10.114.209.37   ansible_ssh_user=localadmin ansible_ssh_private_key_file=~/.ssh/ansible_key
  ```

  - Note:
    - The exercises in this course only use the localhost entry, but the sample host file also includes an example of how to specify a jumphost. It is common in production Ansible deployments to use jumphosts to execute plays.
- Prepare Answer File for NSX Manager Spec
  - In this exercise you need to use a text editor to create a file. I use vi editor, if you are newer to Linux you may prefer to use the gedit text editor, which you can do by simply replacing any reference in these instructions to `vi` with `gedit`. Or you can use any text editor you prefer. 
  - Create an answerfile.yml file
    - `cd ~/nsxansible/`
    - `vi answerfile.yml`
  - Edit the file to look like this - making sure to change any variables that are different in your environment:
    ```
    nsxmanager_spec:
      raml_file: 'home/vmware/nsxraml/nsxvapi.raml'
      host: 'nsxmanager-01a.corp.local'
      user: 'admin'
      password: 'VMware1!'
    ```

  - __Note:__ At this point in the exercises, you have not yet deployed NSX Manager, so it is important when you deploy it in Lab-2, the host, username and password variable you use here match. 

### Congratulations, you have completed Lab 1!
## [Click Here To Proceed To Lab-2](../../Lab2-NSXDeploy/)
