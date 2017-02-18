# NSX Ansible Control & Data Plane Setup
In the previous lab, you installed NSX Manager and registered it with vCenter. The next step in the setup of NSX is to install and configure the management and data plane components that will be required to enable NSX Services. 

This Lab includes the following sections:

- [Creating NSX Controllers]()
- [Prepare ESXi hosts - Install NSX VIBs]()
- [VTEP Configuration]()
- [Creating Transport Zones]()


## Creating NSX Controllers
VMware NSX uses controllers to manage and synchronize data plane information for logical networks. Creating controllers requires several different steps. 

The NSX Ansible modules work by interacting with the NSX REST API, and to make an API call to create controllers,  the Management Object ID (MOID) of the cluster, datastore and portgroup that you would like to deploy the controllers on needs to be specified. To gather the MOIDs, one of the NSX Ansible Modules, the`vcenter_gather_moids` module, can be used to simplify this process. 

Before controllers can be created, you must first define an IP Address Pool that NSX Manager will use to assign IP Addresses to the controllers. 

Accordingly, the playbook we use in this section will include 5 tasks, the first 3 tasks gather the needed MOIDs, the fourth task creates the controller IP Pool, and then a fifth task creates the controllers.

- Prepare variables needed for required playbook tasks
  - The following variables, organized by task, will be required to prepare the playbook
    - Variables common to multiple tasks:
      - vCenter hostname (FQDN)
      - vCenter username (Account must have permission to create controllers)
      - vCenter password
      - Name of target datacenter in vCenter
      - nsxmanager_spec (already defined in answerfile.yml created in previous lab)
    - Task 1: Gather vCenter Management Cluster MOID
      - Target Cluster Name
    - Task 2 Gather vCenter Management Datastore MOID
      - Target Datastore Name
    - Task 3 Gather vCenter Management Portgroup MOID
      - Target PortGroup Name
    - Task 4: Create Controller IP Pools
      - IP Pool Name
      - Pool Starting IP Address
      - Pool Ending IP Address
      - Subnet Mask Prefix Length
      - Gateway
      - DNS Server(s)
        - These values will be assigned to the controller by NSX Manager. With Ansible, you will specify these values as variables in your playbook. 
    - Task 5: Create NSX Controller cluster
      - nsxmanager_spec (already defined in answerfile.yml created in previous exercise)
      - deployment type (options related to quantity of controllers desired, for full details please see [official documentation](https://github.com/vmware/nsxansible#module-nsx_controllers))
      - IP Pool ID
        - __Note:__ The IP Pool ID is gathered transparently into a temporary, environmental variable by the playbook created in the previous exercise [Creating Controller IP Pools](). This variable is not persistant across logins or reboots, so it is important to either run this playbook in the same session that you ran the playbook to create the ip pool, or manually specify the controller IP Pool ID as a variable. The instructions in this section assume you are running this playbook in the same bash session as the previous play where you created the controller ip pool, if you have not, you will need to gather the IP Pool ID manually.
      - Management Cluster MOID
      - Datastore MOID
      - Network MOID
      - The controller CLI/SSH password of the 'admin' user
- Update answerfile.yml with needed variables
  - Note: For demonstration/educational purposes we will put the required variables in the external answer file, they could instead be specified directly in the playbook if desired.  
  -Return to you terminal session to the Ansible server 
    - `cd ~/nsxansible`
    - `vi answerfile.yml` 
    - Edit the file to look like the following:
```
---
nsxmanager_spec:
  raml_file: '/home/vmware/nsxraml/nsxvapi.raml'
  host: 'nsxmanager-01a.corp.local'
  user: 'admin'
  password: 'VMware1!'

vcHostname: 'vcsa-01a.corp.local'
vcUser: 'administrator@vsphere.local'
vcPassword: 'VMware1!'
targetDatacenterName: 'RegionA01'
targetVdsName: 'RegionA01-vDS-COMP'
clusterName: 'RegionA01-COMP01'
nsxControllerDatastore: 'RegionA01-ISCI01-COMP01'
nsxControllerPortGroup: 'ESXi-RegionA01-vDS-COMP'
controllerDeployType: 'full'
controllerPassword: 'VMware1!'

targetClusters:
  cluster1: 
    clusterName: 'RegionA01-vDS-COMP'
  cluster2:
    #clusterName: 'In the reference lab, there is only one cluster, this example shows how to use nested notation to create an object schema for variable values'

nsxIpPools:
  controller:
    name: 'ansible_controller_ip_pool'
    start_ip: '192.168.110.31'
    end_ip: '192.168.110.33'
    prefix_length: '24'
    gateway: '192.168.110.1'
    dns_server_1: '192.168.110.10'

```

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
      hostname: "{{ vcHostname }}"
      username: "{{ vcUser }}"
      password: "{{ vcPassword }}"
      datacenter_name: "{{ targetDatacenterName }}"
      cluster_name: "{{ targetClusters.cluster1.clusterName }}"
      validate_certs: False
      register: vcenter_mgmt_cluster_moid

  - name: Gather vCenter management datastore moid
      vcenter_gather_moids:
      hostname: "{{ vcHostname }}"
      username: "{{ vcUser }}"
      password: "{{ vcPassword }}"
      datacenter_name: "{{ targetDatacenterName }}"
      datastore_name: "{{ nsxControllerDatastore }}"
      validate_certs: False
      register: vcenter_mgmt_datastore_moid

  - name: Gather vCenter management portgroup moid
      vcenter_gather_moids:
      hostname: "{{ vcHostname }}"
      username: "{{ vcUser }}"
      password: "{{ vcPassword }}"
      datacenter_name: "{{ targetDatacenterName }}"
      portgroup_name: "{{ nsxControllerPortGroup }}"
      validate_certs: False
      register: vcenter_mgmt_portgroup_moid

  - name: Create IP Controller IP Pools
      nsx_ippool:
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        state: present
        name: "{{ nsxIppools.controller.name }}"
        start_ip: "{{ nsxIppools.controller.start_ip }}"
        end_ip: "{{ nsxIppools.controller.end_ip }}"
        prefix_length: "{{ nsxIppools.controller.prefix_length }}"
        gateway: "{{ nsxIppools.controller.gateway }}"
        dns_server_1: "{{ nsxIppools.controller.dns_server_1 }}"
      register: controller_ip_pool
      tags: controller_nsx_ippools

  - name: Create NSX Controller cluster
      nsx_controllers:
          nsxmanager_spec: "{{ nsxmanager_spec }}"
          state: present
          deploytype: "{{ controllerDeployType }}"
          syslog_server: "{{ controllerSyslogServer }}"
          ippool_id: "{{ controller_ip_pool.ippool_id }}"
          resourcepool_moid: "{{ vcenter_mgmt_cluster_moid.object_id }}"
          datastore_moid: "{{ vcenter_mgmt_datastore_moid.object_id }}"
          network_moid: "{{ vcenter_mgmt_portgroup_moid.object_id }}"
          password: "{{ controllerPassword }}"
      tags: nsx_controllers
  
  ```
- Run  the play and review results
  - If the play completes successfully, you should see output similar to the following:
```

```

## Preparing ESXi Hosts with VIB installation

NSX distributed networking features are provided to each VM by services running in the hyperisor of each ESXi host. To enable these services, you must prepare ESXi hosts in the target environment by installing a vSphere Installation Bundle (VIB).   

__Note:__ In the following playbook example, observe that the install VIBs task requires that you specify the MOID of the target cluster. Accordingly, before running the Install VIBs task, it first runs a task to gather the MOID. Please observe the required variables in each task, and you can see the variables are dependent on the answerfile.yml file you created in the previous exercise. 

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
      hostname: "{{ vcHostname }}"
      username: "{{ vcUser }}"
      password: "{{ vcPassword }}"
      datacenter_name: "{{ targetDatacenterName }}"
      cluster_name: "{{ targetClusters.cluster1.clusterName }}"
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
 - If the play completes successfully, you should see output similar to the following:
```

```

## Configuring VTEPs
NSX provides overlay networking tunnels to enable virtualized network services. To deliver these services, each participating ESXi host must be configured with a vxlan tunnel endpoint (VTEP) interface.  The previous step of installing NSX VIB files to prepare all ESXi hosts in the target environment MUST be done before attempting to configure VTEPs. 

VTEP Configuration is a multi-step process:
- The example playbook will execute 4 tasks:
  - __Gather vCenter Managment Cluster MOID__
    - Prepare Task Variable Data
      - vCenter hostname (FQDN)
      - vCenter username (Account must have permission to create controllers)
      - vCenter password
      - Name of target datacenter in vCenter
      - nsxmanager_spec (already defined in answerfile.yml created in previous lab)
      - Target Cluster Name 
  - __Gather Target vDS MOID__
    - Description: 
      - This task identifies the specific vDS the VTEP interface will be deployed onto. 
    - Prepare Task Variable Data:
      - vCenter Hostname/FQDN
      - vCenter Username
      - vCenter Password
      - Target vCenter Datacenter Name
      - Target vSphere Distributed Switch Name
  - __Create VTEP IP Pool__
    - Description: 
      - IP Pool(s) with network addressing information for VTEP interfaces
    - Prepare Task Variable Data::
      - IP Pool Name
      - Pool Starting IP Address
      - Pool Ending IP Address
      - Subnet Mask Prefix Length
      - Gateway
      - DNS Server(s)
  - __VXLAN Prep on Target Cluster__
    - Description:
      - This tasks creates and configures the VTEP Interface on the target ESXi Host

Update answerfile.yml with the variables needed for this playbook

- Return to your terminal session with the Ansible server
  - `cd ~/nsxansible`
  - `vi answerfile.yml`
  - Edit the file to look like the following:
```
---
nsxmanager_spec:
  raml_file: '/home/vmware/nsxraml/nsxvapi.raml'
  host: 'nsxmanager-01a.corp.local'
  user: 'admin'
  password: 'VMware1!'

vcHostname: 'vcsa-01a.corp.local'
vcUser: 'administrator@vsphere.local'
vcPassword: 'VMware1!'
targetDatacenterName: 'RegionA01'
targetVdsName: 'RegionA01-vDS-COMP'
clusterName: 'RegionA01-COMP01'
nsxControllerDatastore: 'RegionA01-ISCI01-COMP01'
nsxControllerPortGroup: 'ESXi-RegionA01-vDS-COMP'
controllerDeployType: 'full'
controllerPassword: 'VMware1!'

targetClusters:
  cluster1: 
    clusterName: 'RegionA01-vDS-COMP'
  cluster2:
    #clusterName: 'In the reference lab, there is only one cluster, this line is here as an example of one way to organize variables for multi-cluster environments'

nsxIpPools:
  controller:
    name: 'ansible_controller_ip_pool'
    start_ip: '192.168.110.31'
    end_ip: '192.168.110.33'
    prefix_length: '24'
    gateway: '192.168.110.1'
    dns_server_1: '192.168.110.10'
  vteps:
    name: 'ansible_vtep_ip_pool'
    start_ip: '192.168.130.51'
    end_ip: '192.168.130.56'
    prefix_length: '24'
    gateway: '192.168.130.1'
    dns_server_1: '192.168.110.10'

```

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
      hostname: "{{ vcHostname }}"
      username: "{{ vcUser }}"
      password: "{{ vcPassword }}"
      datacenter_name: "{{ targetDatacenterName }}"
      cluster_name: "{{ targetClusters.cluster1.clusterName }}"
      validate_certs: False
    register: vcenter_mgmt_cluster_moid

  - name: Gather vCenter Transport DVS moid
      vcenter_gather_moids:
        hostname: "{{ vcHostname }}"
        username: "{{ vcUser }}"
        password: "{{ vcPassword }}"
        datacenter_name: "{{ targetDatacenterName }}"
        dvs_name: "{{ targetVdsName }}"
        validate_certs: False
      register: vcenter_dvs_moid
      tags: vsphere_facts
      
  - name: Create VTEP IP Pools
      nsx_ippool:
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        state: present
        name: "{{ nsxIppools.vteps.name }}"
        start_ip: "{{ nsxIppools.vteps.start_ip }}"
        end_ip: "{{ nsxIppools.vteps.end_ip }}"
        prefix_length: "{{ nsxIppools.vteps.prefix_length }}"
        gateway: "{{ nsxIppools.vteps.gateway }}"
        dns_server_1: "{{ nsxIppools.vteps.dns_server_1 }}"
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


- If the play completes successfully, you should see output similar to the following:
```
```



## Creating Transport Zones
A transport zone controls which hosts a logical switch can reach and can span one or more vSphere clusters. 

Transport Zone Configuration is a multi-step process:
- The example playbook will execute 3 tasks:
  - __Gather vCenter Managment Cluster MOID__
    - Prepare Task Variable Data
      - vCenter hostname (FQDN)
      - vCenter username (Account must have permission to create controllers)
      - vCenter password
      - Name of target datacenter in vCenter
      - nsxmanager_spec (already defined in answerfile.yml created in previous lab)
      - Target Cluster Name 
  - __Create Segment ID Pool__
    - Description: 
      - Segment IDs are similar to VLAN IDs for VXLAN networks, and each Transport Zone is associated with a range of valid Segment IDs that can be forwarded within the Transport Zone. 
    - Prepare Task Variable Data::
      - ID Pool Starting Number
      - ID Pool Ending Number
  - __Add a Transport Zone with target cluster as member__
    - Description:
      - This task creates and configures the VTEP Interface on the target ESXi Host
    - Prepare Task Variable Data:
      - Control Plane Mode
        - The value can be 'UNICAST_MODE', 'HYBRID_MODE' or 'MULTICAST_MODE'. Default is 'UNICAST_MODE'.
      - Transport Zone Description

Update answerfile.yml with the variables needed for this playbook

- Return to your terminal session with the Ansible server
  - `cd ~/nsxansible`
  - `vi answerfile.yml`
  - Edit the file to look like the following:
```
---
nsxmanager_spec:
  raml_file: '/home/vmware/nsxraml/nsxvapi.raml'
  host: 'nsxmanager-01a.corp.local'
  user: 'admin'
  password: 'VMware1!'

vcHostname: 'vcsa-01a.corp.local'
vcUser: 'administrator@vsphere.local'
vcPassword: 'VMware1!'
targetDatacenterName: 'RegionA01'
targetVdsName: 'RegionA01-vDS-COMP'
clusterName: 'RegionA01-COMP01'
nsxControllerDatastore: 'RegionA01-ISCI01-COMP01'
nsxControllerPortGroup: 'ESXi-RegionA01-vDS-COMP'
controllerDeployType: 'full'
controllerPassword: 'VMware1!'

targetClusters:
  cluster1: 
    clusterName: 'RegionA01-vDS-COMP'
  cluster2:
    #clusterName: 'In the reference lab, there is only one cluster, this line is here as an example of one way to organize variables for multi-cluster environments'

nsxIpPools:
  controller:
    name: 'ansible_controller_ip_pool'
    start_ip: '192.168.110.31'
    end_ip: '192.168.110.33'
    prefix_length: '24'
    gateway: '192.168.110.1'
    dns_server_1: '192.168.110.10'
  vteps:
    name: 'ansible_vtep_ip_pool'
    start_ip: '192.168.130.51'
    end_ip: '192.168.130.56'
    prefix_length: '24'
    gateway: '192.168.130.1'
    dns_server_1: '192.168.110.10'

segmentIdPoolStart: '5000'
segmentIdPoolEnd: '5999'
transportZoneName: 'TZ1'
transportZoneDescription: 'Default TZ'
defaultControlPlaneMode: 'Hybrid_Mode'
```

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
        hostname: "{{ vcHostname }}"
        username: "{{ vcUser }}"
        password: "{{ vcPassword }}"
        datacenter_name: "{{ targetDatacenterName }}"
        cluster_name: "{{ targetClusters.cluster1.clusterName }}"
        validate_certs: False
      register: vcenter_mgmt_cluster_moid
    
    - name: Create Segment Id Pool
      nsx_segment_id_pool:
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        state: present
        idpoolstart: "{{ segmentIdPoolStart }}"
        idpoolend: "{{ segmentIdPoolEnd }}"
        mcast_enabled: "{{ mcastEnabled }}"
        mcastpoolstart: "{{ mcastAddrStart }}"
        mcastpoolend: "{{ mcastAddrEnd }}"
      register: segment_pool
      tags: nsx_segment_pool

    - name: Add a Transport Zone with the Cluters as members
      nsx_transportzone:
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        state: 'present'
        name: "{{ transportZoneName }}"
        controlplanemode: "{{ defaultControllPlaneMode }}"
        description: "{{ transportZoneDescription }}"
        cluster_moid_list:
          - "{{ vcenter_mgmt_cluster_moid.object_id }}"
      register: transport_zone
      tags: nsx_transport_zone
```

- If the play completes successfully, you should see output similar to the following:
```
```







### Congratulations, you have completed Lab 3!
## [Click Here To Proceed To Lab-4](../../Lab4-EnvSetup/)


