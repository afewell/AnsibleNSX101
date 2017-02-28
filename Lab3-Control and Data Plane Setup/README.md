# NSX Ansible Control & Data Plane Setup
In the previous lab, you installed NSX Manager and registered it with vCenter. The next step in the setup of NSX is to install and configure the management and data plane components that will be required to enable NSX Services. 

This Lab includes the following sections:

- [Creating NSX Controllers]()
- [Prepare ESXi hosts - Install NSX VIBs]()
- [VTEP Configuration]()
- [Creating Transport Zones]()


## Creating NSX Controllers

### About The Playbook
VMware NSX uses controllers to manage and synchronize data plane information for logical networks. Creating controllers requires several different steps. 

The NSX Ansible modules work by interacting with the NSX REST API, and to make an API call to create controllers,  the Management Object ID (MOID) of the cluster, datastore and portgroup that you would like to deploy the controllers on needs to be specified. To gather the MOIDs, one of the NSX Ansible Modules, the`vcenter_gather_moids` module, can be used to simplify this process. 

Before controllers can be created, you must first define an IP Address Pool that NSX Manager will use to assign IP Addresses to the controllers. 

Accordingly, the playbook we use in this section will include 5 tasks, the first 3 tasks gather the needed MOIDs, the fourth task creates the controller IP Pool, and then a fifth task creates the controllers.

- About the playbook
  - Task 1: Gather vCenter Management Cluster MOID
    - Description:
      - This tasks uses the `vcenter_gather_moids` NSX Ansible module to gather the MOID of the Management Cluster, which is the deployment target for NSX Controllers in this lab. If you look at the playbook below, you will see that the `Create NSX Controller Cluster` task requires the MOID of the managment cluster. 
  - Task 2 Gather vCenter Management Datastore MOID
    - Description:
      - This tasks uses the `vcenter_gather_moids` NSX Ansible module to gather the MOID of the target datastore. If you look at the playbook below, you will see that the `Create NSX Controller Cluster` task requires the MOID of the target datastore. 
  - Task 3 Gather vCenter Management Portgroup MOID
    - Description:
      - This tasks uses the `vcenter_gather_moids` NSX Ansible module to gather the MOID of the target portgroup. If you look at the playbook below, you will see that the `Create NSX Controller Cluster` task requires the MOID of the target portgroup. 
  - Task 4: Create Controller IP Pools
    - Description:
      - This task creates the IP pool which specifies the IP Address information which is assigned dynamically to the controllers by NSX manager at creation. 
  - Task 5: Create NSX Controller cluster
    - Description:
      - This task creates the controller(s) using the information from the previous 4 tasks plus the variables specified in the playbook below. 

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
    - Note: Your output may not look exactly like the output below, for example in developing this lab I sometimes have to run the same play multiple times before I get it working correctly which may create differences between the output below and what you see in your terminal. The only thing that really matters is that there are no errors when running the play. To verify this, in the `PLAY RECAP` section, make sure all tasks executed as `ok` or `changed`. The other values should be `unreachable=0` and `failed=0`, if any tasks are unreachable or failed, you need to troubleshoot until you can run the play with no errors before proceeding. 
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
ok: [localhost]

TASK [Create NSX Controller cluster] *******************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=5    changed=1    unreachable=0    failed=0

```
### Verify Results

- Open a browser connection to the vCenter web client home page
  - Click on the `Networking & Security` tab
  - Click on the link to `NSX Managers`  

## Preparing ESXi Hosts with VIB installation

### About the playbook

NSX distributed networking features are provided to each VM by services running in the hyperisor of each ESXi host. To enable these services, you must prepare ESXi hosts in the target environment by installing a vSphere Installation Bundle (VIB). 

Note: The first task in the plabook below will gather the MOID of the specified cluster, in this case although the tasked is named management cluster, there is only a single cluster in the reference lab. If you have more than one cluster in your environment, you will need to execute the VIB installation on each cluster providing NSX services.   

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

```
### Verify results

- Do something

## Configuring VTEPs

### About the playbook
NSX provides overlay networking tunnels to enable virtualized network services. To deliver these services, each participating ESXi host must be configured with a vxlan tunnel endpoint (VTEP) interface.  The previous step of installing NSX VIB files to prepare all ESXi hosts in the target environment MUST be done before attempting to configure VTEPs. 

VTEP Configuration is a multi-step process:
- The example playbook will execute 4 tasks:
  - __Gather vCenter Managment Cluster MOID__
    - Description:
      - This task will gather the MOID of the specified cluster, in this case although the tasked is named management cluster, there is only a single cluster in the reference lab. If you have more than one cluster in your environment, you will need to execute the VTEP configuration on each cluster providing NSX services. 
  - __Gather Target vDS MOID__
    - Description: 
      - This task identifies the MOID of the vDS the VTEP interface will be deployed onto. 
  - __Create VTEP IP Pool__
    - Description: 
      - IP Pool(s) with network addressing information for VTEP interfaces
  - __VXLAN Prep on Target Cluster__
    - Description:
      - This tasks creates and configures the VTEP Interface on the target ESXi Host

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
    tags: vsphere_facts
      
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
      tags: vtep_nsx_ippools

  - name: VXLAN Prep (configure VTEP) on mgmt cluster
      nsx_vxlan_prep:
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        state: present
        cluster_moid: "{{ vcenter_mgmt_cluster_moid.object_id }}"
        dvs_moid: "{{ vcenter_dvs_moid.object_id }}"
        ippool_id: "{{ vtep_ip_pool.ippool_id }}"
      register: vxlan_prep
      tags: nsx_vxlan_prep  

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
```
### Verify results

- Do something


## Creating Transport Zones

### About the playbook
A transport zone controls which hosts a logical switch can reach and can span one or more vSphere clusters. 

Transport Zone Configuration is a multi-step process:
- The example playbook will execute 3 tasks:
  - __Gather vCenter Managment Cluster MOID__
  - __Create Segment ID Pool__
    - Description: 
      - Segment IDs are similar to VLAN IDs for VXLAN networks, and each Transport Zone is associated with a range of valid Segment IDs that can be forwarded within the Transport Zone. 
  - __Add a Transport Zone with target cluster as member__
    - Description:
      - This task creates and configures the VTEP Interface on the target ESXi Host(s)

### Create the playbook
- Create the playbook
  - Return to your terminal session with the Ansible server
  - `cd ~/nsxansible`
  - `vi setupTZ.yml`
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

- If the play completes successfully, you should see output similar to the following:
```
```
### Verify results

- Do something






### Congratulations, you have completed Lab 3!
## [Click Here To Proceed To Lab-4](../../Lab4-EnvSetup/)


