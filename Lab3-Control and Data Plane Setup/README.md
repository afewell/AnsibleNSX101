# NSX Ansible Control & Data Plane Setup
In the previous lab, you installed NSX Manager and registered it with vCenter. The next step in the setup of NSX is to install and configure the management and data plane components that will be required to enable NSX Services. 

This Lab includes the following sections:

- [Creating Controller IP Pools]()
- [Create NSX Controller Clusters]()
- [Prepare ESXi hosts - Install NSX VIBs]()
- [Create VTEP IP Pools]()
- [VTEP Configuration]()
- [Create Segment ID Pool]()
- [Add Transport Zone]()

Please note the sections in this lab are done in an intentional order that should be followed for new NSX Installations. The Controller IP pool must be created before the Controllers are deployed, and so on. 

## Creating Controller IP Pools

VMware NSX uses controllers to manage and synchronize data plane information for logical networks. Before controllers can be created, you must first define an IP Address Pool that NSX Manager will use to assign IP Addresses to the controllers. 

1. Prepare an Ansible Playbook to create the controller IP pool
  - When you create an IP Pool, you need to specify the following values:
    - IP Pool Name
    - Pool Starting IP Address
    - Pool Ending IP Address
    - Subnet Mask Prefix Length
    - Gateway
    - DNS Server(s)
      - These values will be assigned to the controller by NSX Manager. With Ansible, you will specify these values as variables in your playbook. 
  - Return to your terminal session with the Ansible server
    - `cd ~/nsxansible`
    - `vi answerfile.yml`
    - Create a createIpPools.yml file and edit it to look like the following:
```
---
- hosts: localhost
    connection: local
    gather_facts: False
    vars_files:
        - answerfile.yml
    tasks:
    - name: Controller IP Pool creation
    nsx_ippool:
        nsxmanager_spec: "{{ nsxmanager_spec }}"
        state: present
        name: 'ansible_controller_ip_pool'
        start_ip: '192.168.110.31'
        end_ip: '192.168.110.33'
        prefix_length: '24'
        gateway: '192.168.110.1'
        dns_server_1: '192.168.110.10'
    register: create_ip_pool

    - debug: var=create_ip_pool
```

2. Run  the play and review results
  - If the play completes successfully, you should see output similar to the following:
```

```

## Creating NSX Controller Clusters
After creating an IP pool for the clusters, you can create a playbook to deploy the NSX Controllers. The NSX Ansible modules work by interacting with the NSX REST API, and to make an API call to create controllers,  the Management Object ID (MOID) of the cluster, datastore and portgroup that you would like to deploy the controllers on needs to be specified. To gather the MOIDs, one of the NSX Ansible Modules, the`vcenter_gather_moids` module, can be used to simplify this process. Accordingly, the playbook we use in this section will include four tasks, the first 3 tasks gather the needed MOIDs, then a fourth task to create the controllers.

1. Prepare variables needed for required playbook tasks
  - The following variables, organized by task, will be required to prepare the playbook
    - Variables common to multiple tasks:
      - vCenter hostname (FQDN)
      - vCenter username (Account must have permission to create controllers)
      - vCenter password
      - Name of target datacenter in vCenter
    - Task 1: Gather vCenter Management Cluster MOID
      - Target Cluster Name
    - Task 2: Gather vCenter Management Datastore MOID
      - Target Datastore Name
    - Task 3: Gather vCenter Management Portgroup MOID
      - Target PortGroup Name
    - Task 4: Create NSX Controller cluster
      - nsxmanager_spec (already defined in answerfile.yml created in previous exercise)
      - deployment type (options related to quantity of controllers desired, for full details please see [official documentation](https://github.com/vmware/nsxansible#module-nsx_controllers))
      - IP Pool ID
        - __Note:__ The IP Pool ID is gathered transparently into a temporary, environmental variable by the playbook created in the previous exercise [Creating Controller IP Pools](). This variable is not persistant across logins or reboots, so it is important to either run this playbook in the same session that you ran the playbook to create the ip pool, or manually specify the controller IP Pool ID as a variable. The instructions in this section assume you are running this playbook in the same bash session as the previous play where you created the controller ip pool, if you have not, you will need to gather the IP Pool ID manually.
      - Management Cluster MOID
      - Datastore MOID
      - Network MOID
      - The controller CLI/SSH password of the 'admin' user
2. Update answerfile.yml with common variables
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

```

2. Run  the play and review results
  - If the play completes successfully, you should see output similar to the following:
```

```


The NSX Ansible Modules works by interacting with the NSX REST API. When you make a call to the API, you are essentially asking it to take some action on some object, and you need to identify which objects you want to modify. For example, typically when you deploy a virtual machine, you need to deploy it into an existing cluster, onto an existing datastore, and connected to an existing portgroup. When using the GUI, you typically click on these values or use their common name, however when making calls to the API, you need to use special values so the API host knows exactly what you want to do. 

This section walks through the `vcenter_gather_moids` module, which is used to gather the Management IDs (MOIDs) of the cluster, datastore and portgroups used by later modules in this course. 

This section will introduce the use of multiple tasks in a single playbook. As you will see in the exercise, each task uses several of the same variables, so rather than including your variables directly within each task, it makes sense to use an external answer file to hold a common set of variables to be used in the play. 

1. Add variables needed to answerfile.yml
    + `cd ~/nsxansible`
    + f
    + `vi answerfile.yml`
    + Edit answerfile.yml to look like the following - be sure to change any variables as needed for your lab environment:
    ```
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
    targetClusters.cluster1.clusterName: 'RegionA01-COMP01'
    nsxControllerDatastore: 'RegionA01-ISCI01-COMP01'
    nsxControllerPortGroup: 'ESXi-RegionA01-vDS-COMP'
    ```
- Prepare an Ansible playbook to gather MOIDs
  - Use a text editor to create and open a "gathermoids.yml" file
    - `cd ~/nsxansible`
    - `vi gathermoids.yml`
  - Edit the gathermoids.yml file to look like the following:
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
      tags: vsphere_facts

    - name: Gather vCenter management datastore moid
      vcenter_gather_moids:
        hostname: "{{ vcHostname }}"
        username: "{{ vcUser }}"
        password: "{{ vcPassword }}"
        datacenter_name: "{{ targetDatacenterName }}"
        datastore_name: "{{ nsxControllerDatastore }}"
        validate_certs: False
      register: vcenter_mgmt_datastore_moid
      tags: vsphere_facts

    - name: Gather vCenter management portgroup moid
      vcenter_gather_moids:
        hostname: "{{ vcHostname }}"
        username: "{{ vcUser }}"
        password: "{{ vcPassword }}"
        datacenter_name: "{{ targetDatacenterName }}"
        portgroup_name: "{{ nsxControllerPortGroup }}"
        validate_certs: False
      register: vcenter_mgmt_portgroup_moid
      tags: vsphere_facts

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
    
    ```
  - If the play completes successfully, you should see output similar to the following:
```

```

### Congratulations, you have completed Lab 3!
## [Click Here To Proceed To Lab-4](../../Lab4-EnvSetup/)


