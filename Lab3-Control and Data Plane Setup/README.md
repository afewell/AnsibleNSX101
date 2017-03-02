# NSX Ansible Control & Data Plane Setup
In the previous lab, you installed NSX Manager and registered it with vCenter. The next step in the setup of NSX is to install and configure the management and data plane components that will be required to enable NSX Services. 

This Lab includes the following sections:

- [Creating NSX Controllers](https://github.com/afewell/AnsibleNSX101/tree/master/Lab3-Control%20and%20Data%20Plane%20Setup#creating-nsx-controllers)
- [Prepare ESXi hosts with VIB installation](https://github.com/afewell/AnsibleNSX101/tree/master/Lab3-Control%20and%20Data%20Plane%20Setup#preparing-esxi-hosts-with-vib-installation)
- [VTEP Configuration](https://github.com/afewell/AnsibleNSX101/tree/master/Lab3-Control%20and%20Data%20Plane%20Setup#configuring-vteps)
- [Creating Transport Zones](https://github.com/afewell/AnsibleNSX101/tree/master/Lab3-Control%20and%20Data%20Plane%20Setup#creating-transport-zones)


## Creating NSX Controllers
### About The Playbook
VMware NSX uses controllers to manage and synchronize data plane information for logical networks. Creating controllers requires several different steps. 

The NSX Ansible modules work by interacting with the NSX REST API, and to make an API call to create controllers,  the Management Object ID (MOID) of the cluster, datastore and portgroup that you would like to deploy the controllers on needs to be specified. To gather the MOIDs, one of the NSX Ansible Modules, the`vcenter_gather_moids` module, can be used to simplify this process. 

Before controllers can be created, you must first define an IP Address Pool that NSX Manager will use to assign IP Addresses to the controllers. 

Accordingly, the playbook we use in this section will include 5 tasks, the first 3 tasks gather the needed MOIDs, the fourth task creates the controller IP Pool, and then a fifth task creates the controllers.

- About the playbook
  - Task 1: Gather vCenter Management Cluster MOID
    - Description:
      - This task uses the `vcenter_gather_moids` NSX Ansible module to gather the MOID of the Management Cluster, which is the deployment target for NSX Controllers in this lab. If you look at the playbook below, you will see that the `Create NSX Controller Cluster` task requires the MOID of the managment cluster. 
      - [Documentation for the vcenter_gather_moids module](https://github.com/vmware/nsxansible#module-vcenter_gather_moids)
  - Task 2 Gather vCenter Management Datastore MOID
    - Description:
      - This task uses the `vcenter_gather_moids` NSX Ansible module to gather the MOID of the target datastore. If you look at the playbook below, you will see that the `Create NSX Controller Cluster` task requires the MOID of the target datastore. 
  - Task 3 Gather vCenter Management Portgroup MOID
    - Description:
      - This task uses the `vcenter_gather_moids` NSX Ansible module to gather the MOID of the target portgroup. If you look at the playbook below, you will see that the `Create NSX Controller Cluster` task requires the MOID of the target portgroup. 
  - Task 4: Create Controller IP Pools
    - Description:
      - This task creates the IP pool which specifies the IP Address information which is assigned dynamically to the controllers by NSX manager at creation. 
      - [Documentation for the nsx_ippool module](https://github.com/vmware/nsxansible#module-nsx_ippool)
  - Task 5: Create NSX Controller cluster
    - Description:
      - This task creates the controller(s) using the information from the previous 4 tasks plus the variables specified in the playbook below. 
      - [Documentation for the nsx_controllers module](https://github.com/vmware/nsxansible#module-nsx_controllers)

### Create the playbook

- Create the playbook
  - Return to your terminal session with the Ansible server
  - `cd ~/nsxansible`
  - `vi createControllers.yml`
  - Edit the file to look like the following:
```
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
     - answerfile.yml
  tasks:
  - name: Gather vCenter Mgmt Cluster moid
    vcenter_gather_moids:
      hostname: 'vcsa-01a.corp.local'
      username: 'administrator@corp.local'
      password: 'VMware1!'
      datacenter_name: 'RegionA01'
      cluster_name: 'RegionA01-COMP01'
      validate_certs: False
    register: vcenter_mgmt_cluster_moid

  - name: Gather vCenter management datastore moid
    vcenter_gather_moids:
      hostname: 'vcsa-01a.corp.local'
      username: 'administrator@corp.local'
      password: 'VMware1!'
      datacenter_name: 'RegionA01'
      datastore_name: 'RegionA01-ISCSI01-COMP01'
      validate_certs: False
    register: vcenter_mgmt_datastore_moid

  - name: Gather vCenter management portgroup moid
    vcenter_gather_moids:
      hostname: 'vcsa-01a.corp.local'
      username: 'administrator@corp.local'
      password: 'VMware1!'
      datacenter_name: 'RegionA01'
      portgroup_name: 'ESXi-RegionA01-vDS-COMP'
      validate_certs: False
    register: vcenter_mgmt_portgroup_moid

  - name: Create IP Controller IP Pools
    nsx_ippool:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      name: 'regionA01ControllerPool'
      start_ip: '192.168.110.31'
      end_ip: '192.168.110.33'
      prefix_length: '24'
      gateway: '192.168.110.1'
      dns_server_1: '192.168.110.10'
    register: controller_ip_pool
    tags: controller_nsx_ippools

  - name: Create NSX Controller cluster
    nsx_controllers:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      deploytype: 'full'
      ippool_id: "{{ controller_ip_pool.ippool_id }}"
      resourcepool_moid: "{{ vcenter_mgmt_cluster_moid.object_id }}"
      datastore_moid: "{{ vcenter_mgmt_datastore_moid.object_id }}"
      network_moid: "{{ vcenter_mgmt_portgroup_moid.object_id }}"
      password: 'VMware1!VMware1!'
    tags: nsx_controllers
  
  ```
### Run the playbook
- Run  the play and review results
  - Return to your terminal session with the Ansible server
    - `cd ~/nsxansible`
    - `ansible-playbook -i hosts createControllers.yml`
  - Note: Once you run the play, Ansible will execute the tasks in order and display the status of each tasks. After the first 4 tasks complete successfully, the `Create NSX Controllers` task will take a long time (15 minutes or more) to run, during which time no output will be displayed. During this time you simply must wait, if the task fails, you will recieve an error message. 
  - If the play completes successfully, you should see output similar to the following:
    - Note: Your output may not look exactly like the output below, the only thing that really matters is that there are no errors when running the play. To verify this, in the `PLAY RECAP` section, make sure all tasks executed as `ok` or `changed`. The other values should be `unreachable=0` and `failed=0`, if any tasks are unreachable or failed, you need to troubleshoot until you can run the play with no errors before proceeding. 
```
vmware@vmware:~/nsxansible$ ansible-playbook -i hosts createControllers.yml

PLAY [localhost] ***************************************************************

TASK [Gather vCenter Mgmt Cluster moid] ****************************************
ok: [localhost]

TASK [Gather vCenter management datastore moid] ********************************
ok: [localhost]

TASK [Gather vCenter management portgroup moid] ********************************
ok: [localhost]

TASK [Create IP Controller IP Pools] *******************************************
changed: [localhost]

TASK [Create NSX Controller cluster] *******************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=5    changed=2    unreachable=0    failed=0

```
### Verify Results

- Open a browser connection to the vCenter web client home page
  - Verify IP Pool Creation
    - Click on the `Networking & Security` tab
    - Click on the link to `NSX Managers` 
    - Click on the IP Address of your NSX Manager
    - Click on the Manage Tab
    - Click on the "Grouping Objects" tab
    - Click on "IP Pools"
    - You should see the "regionA01ControllerPool" that was created by this play
![controllerPoolVerify](images/controllerPoolVerify.PNG)
  - Verify Controller Deployment
    - From the vSphere web client home page, navigate to the "Networking & Security" Section
    - Click the "Installation" tab on the left Navigator bar
    - In the "NSX Controller Nodes" Section, you should see 3 controllers listed
![controllerVerify](images/controllerVerify.PNG)

## Preparing ESXi Hosts with VIB installation
### About the playbook
NSX distributed networking features are provided to each VM by services running in the hyperisor of each ESXi host. To enable these services, you must prepare ESXi hosts in the target environment by installing a vSphere Installation Bundle (VIB). 
- Task 1: Gather vCenter Management Cluster MOID
  - Description:
    - This task uses the `vcenter_gather_moids` NSX Ansible module to gather the MOID of the Management Cluster, which is the deployment target for NSX Controllers in this lab. 
    - This task will gather the MOID of the specified cluster, in this case although the tasked is named management cluster, there is only a single cluster in the reference lab. If you have more than one cluster in your environment, you will need to execute the VIB installation on each cluster providing NSX services.
    - [Documentation for the vcenter_gather_moids module](https://github.com/vmware/nsxansible#module-vcenter_gather_moids)
- Task 2: Install VIBs (prepare) the target cluster
    - Description:
      - This task executes the installation of the NSX VIB files on the target cluster. 
      - [Documentation for the nsx_cluster_prep module](https://github.com/vmware/nsxansible#module-nsx_cluster_prep)

### Create the playbook

- Create a playbook to install NSX VIBs
  - Return to your terminal session with the Ansible server
    - `cd ~/nsxansible/`
    - `vi installNsxVibs.yml`
    - Edit the playbook to look like the following:
```
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
     - answerfile.yml
  tasks:
  - name: Gather vCenter Mgmt Cluster moid
    vcenter_gather_moids:
      hostname: 'vcsa-01a.corp.local'
      username: 'administrator@corp.local'
      password: 'VMware1!'
      datacenter_name: 'RegionA01'
      cluster_name: 'RegionA01-COMP01'
      validate_certs: False
    register: vcenter_mgmt_cluster_moid

  - name: Install VIBs (prepare) the target cluster
    nsx_cluster_prep:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      cluster_moid: "{{ vcenter_mgmt_cluster_moid.object_id }}"
    register: cluster_prep_mgmt
    tags: nsx_cluster_prep
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
vmware@vmware:~/nsxansible$ ansible-playbook -i hosts installNsxVibs.yml

PLAY [localhost] ***************************************************************

TASK [Gather vCenter Mgmt Cluster moid] ****************************************
ok: [localhost]

TASK [Install VIBs (prepare) the target cluster] *******************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0
```

### Verify results

- From the vSphere web client home page, navigate to the "Networking & Security" Section
  - Click the "Installation" tab on the left Navigator bar
  - Click on the "Host Preparation" tab
  - RegionA01-COMP01 should display the installed version, the firewall should be enabled, and VXLAN should not yet be configured, as in the image below
![vibVerify](images/vibVerify.PNG)

## Configuring VTEPs
### About the playbook
NSX provides overlay networking tunnels to enable virtualized network services. To deliver these services, each participating ESXi host must be configured with a vxlan tunnel endpoint (VTEP) interface.  The previous step of installing NSX VIB files to prepare all ESXi hosts in the target environment MUST be done before attempting to configure VTEPs. 

VTEP Configuration is a multi-step process:
- The example playbook will execute 4 tasks:
  - Task 1: Gather vCenter Managment Cluster MOID
    - Description:
      - This task will gather the MOID of the specified cluster, in this case although the tasked is named management cluster, there is only a single cluster in the reference lab. If you have more than one cluster in your environment, you will need to execute the VTEP configuration on each cluster providing NSX services. 
      - [Documentation for the vcenter_gather_moids module](https://github.com/vmware/nsxansible#module-vcenter_gather_moids)
  - Task 2: Gather Target vDS MOID
    - Description: 
      - This task identifies the MOID of the vDS the VTEP interface will be deployed onto. 
  - Task 3: Create VTEP IP Pool
    - Description: 
      - IP Pool(s) with network addressing information for VTEP interfaces
    - [Documentation for the nsx_ippool module](https://github.com/vmware/nsxansible#module-nsx_ippool)
  - Task 4: VXLAN Prep on Target Cluster
    - Description:
      - This task creates and configures the VTEP Interfaces on ESXi hosts in the target cluster
    - [Documentation for the nsx_vxlan_prep module](https://github.com/vmware/nsxansible#module-nsx_vxlan_prep)

### Create the playbook

- Create the playbook
  - Return to your terminal session with the Ansible server
  - `cd ~/nsxansible`
  - `vi setupVtep.yml`
  - Edit the file to look like the following:
```
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
    - answerfile.yml
  tasks:
  - name: Gather vCenter Mgmt Cluster moid
    vcenter_gather_moids:
      hostname: 'vcsa-01a.corp.local'
      username: 'administrator@corp.local'
      password: 'VMware1!'
      datacenter_name: 'RegionA01'
      cluster_name: 'RegionA01-COMP01'
      validate_certs: False
    register: vcenter_mgmt_cluster_moid

  - name: Gather vCenter Transport DVS moid
    vcenter_gather_moids:
      hostname: 'vcsa-01a.corp.local'
      username: 'administrator@corp.local'
      password: 'VMware1!'
      datacenter_name: 'RegionA01'
      dvs_name: 'RegionA01-vDS-COMP'
      validate_certs: False
    register: vcenter_dvs_moid
      
  - name: Create VTEP IP Pools
    nsx_ippool:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      name: 'vtepPool-130'
      start_ip: '192.168.130.51'
      end_ip: '192.168.130.56'
      prefix_length: '24'
      gateway: '192.168.130.1'
      dns_server_1: '192.168.110.10'
    register: vtep_ip_pool

  - name: VXLAN Prep (configure VTEP) on mgmt cluster
    nsx_vxlan_prep:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      cluster_moid: "{{ vcenter_mgmt_cluster_moid.object_id }}"
      dvs_moid: "{{ vcenter_dvs_moid.object_id }}"
      ippool_id: "{{ vtep_ip_pool.ippool_id }}"
    register: vxlan_prep

```

### Run the playbook
- Run  the play and review results
  - Return to your terminal session with the Ansible server
    - `cd ~/nsxansible`
    - `ansible-playbook -i hosts setupVtep.yml`
  - If the play completes successfully, you should see output similar to the following:
```
vmware@vmware:~/nsxansible$ ansible-playbook -i hosts setupVtep.yml

PLAY [localhost] ***************************************************************

TASK [Gather vCenter Mgmt Cluster moid] ****************************************
ok: [localhost]

TASK [Gather vCenter Transport DVS moid] ***************************************
ok: [localhost]

TASK [Create VTEP IP Pools] ****************************************************
changed: [localhost]

TASK [VXLAN Prep (configure VTEP) on mgmt cluster] *****************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=4    changed=2    unreachable=0    failed=0

vmware@vmware:~/nsxansible$
```
### Verify results

- Open a browser connection to the vCenter web client home page
  - Verify IP Pool Creation
    - Click on the `Networking & Security` tab
    - Click on the link to `NSX Managers` 
    - Click on the IP Address of your NSX Manager
    - Click on the Manage Tab
    - Click on the "Grouping Objects" tab
    - Click on "IP Pools"
    - You should see the "vtepPool-130" that was created by this play
![vtepPoolVerify](images/vtepPoolVerify.PNG)
  - Verify VXLAN Configuration
    - From the vSphere web client home page, navigate to the "Networking & Security" Section
    - Click the "Installation" tab on the left Navigator bar
    - Click on the "Logical Network Preparation" tab
    - Expand "RegionA01-COMP01" to see each ESXi host
    - Verify that the "Configuration Status" for each host is "Ready" and that an IP VMKNic IP Address has been assigned to each host as in the image below
![vtepVerify](images/vtepVerify.PNG)

## Creating Transport Zones
### About the playbook
A transport zone controls which hosts a logical switch can reach and can span one or more vSphere clusters. 

Transport Zone Configuration is a multi-step process:
- The example playbook will execute 3 tasks:
  - __Gather vCenter Managment Cluster MOID__
    - [Documentation for the vcenter_gather_moids module](https://github.com/vmware/nsxansible#module-vcenter_gather_moids)
  - __Create Segment ID Pool__
    - Description: 
      - Segment IDs are similar to VLAN IDs, but for VXLAN networks. Each Transport Zone is associated with a range of valid Segment IDs that can be forwarded within the Transport Zone. 
    - [Documentation for the nsx_segment_id_pool module](https://github.com/vmware/nsxansible#module-nsx_segment_id_pool)
  - __Add a Transport Zone with target cluster as member__
    - Description:
      - This task creates and configures the VTEP Interface on the target ESXi Host(s)
    - [Documentation for the nsx_transportzone module](https://github.com/vmware/nsxansible#module-nsx_transportzone)

### Create the playbook
- Create the playbook
  - Return to your terminal session with the Ansible server
  - `cd ~/nsxansible`
  - `vi setupTz.yml`
  - Edit the file to look like the following:
```
---
- hosts: localhost
  connection: local
  gather_facts: False
  vars_files:
    - answerfile.yml
  tasks:
  - name: Gather vCenter Mgmt Cluster moid
    vcenter_gather_moids:
      hostname: 'vcsa-01a.corp.local'
      username: 'administrator@corp.local'
      password: 'VMware1!'
      datacenter_name: 'RegionA01'
      cluster_name: 'RegionA01-COMP01'
      validate_certs: False
    register: vcenter_mgmt_cluster_moid
    
  - name: Create Segment Id Pool
    nsx_segment_id_pool:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: present
      idpoolstart: '5000'
      idpoolend: '5999'
    register: segment_pool

  - name: Add a Transport Zone with the Cluters as members
    nsx_transportzone:
      nsxmanager_spec: "{{ nsxmanager_spec }}"
      state: 'present'
      name: 'TZ'
      controlplanemode: 'UNICAST_MODE'
      description: 'Default TZ'
      cluster_moid_list:
        - "{{ vcenter_mgmt_cluster_moid.object_id }}"
    register: transport_zone
```

### Run the playbook
- Run  the play and review results
  - Return to your terminal session with the Ansible server
    - `cd ~/nsxansible`
    - `ansible-playbook -i hosts setupTz.yml`
  - If the play completes successfully, you should see output similar to the following:
```
vmware@vmware:~/nsxansible$ ansible-playbook -i hosts setupTz.yml

PLAY [localhost] ***************************************************************

TASK [Gather vCenter Mgmt Cluster moid] ****************************************
ok: [localhost]

TASK [Create Segment Id Pool] **************************************************
changed: [localhost]

TASK [Add a Transport Zone with the Cluters as members] ************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=3    changed=2    unreachable=0    failed=0

vmware@vmware:~/nsxansible$
```
### Verify results
- Open a browser connection to the vCenter web client home page
  - Verify Segment ID Pool Configuration
    - From the vSphere web client home page, navigate to the "Networking & Security" Section
    - Click the "Installation" tab on the left Navigator bar
    - Click on the "Logical Network Preparation" tab
    - Click on the "Segment ID" tab
    - Verify that the "Segment ID Pool" is set to "5000-5999
![segmentPoolVerify](images/segmentPoolVerify.PNG)
  - Verify Transport Zone Configuration
    - From the vSphere web client home page, navigate to the "Networking & Security" Section
    - Click the "Installation" tab on the left Navigator bar
    - Click on the "Logical Network Preparation" tab
    - Click on the "Transport Zones" tab
    - Verify that the transport zone "TZ" is listed with the "Control Plane Mode" set to "Unicast" as in the image below
![tzVerify](images/tzVerify.PNG)

### Congratulations, you have completed Lab 3!
## [Click Here To Proceed To Lab-4](../../Lab4-EnvSetup/)


