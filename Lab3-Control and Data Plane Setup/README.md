## NSX Ansible Control & Data Plane Setup
In the previous lab, you installed NSX Manager and registered it with vCenter. The next step in the setup of NSX is to install and configure the Management and Data Plane components that will be required to enable NSX Services. 

This Lab will include the following sections:

- Creating Controller IP Pools
- Create NSX Controller Clusters
- Prepare ESXi hosts - Install NSX VIBs
- Create VTEP IP Pools
- VTEP Configuration
- Create Segment ID Pool
- Add Transport Zone

Please note the sections in this lab are done in an intentional order that should be followed for new NSX Installations. The Controller IP pool must be created before the Controllers are deployed, and so on. 

### Creating Controller IP Pools

VMware NSX uses Controllers to manage and synchronize Data Plane information for logical networks. Before Controllers can be created, you must first define an IP Address Pool that NSX Manager will use to assign IP Addresses to Controllers. 

1. Update answerfile.yml with variables to create IP Pool
2.  fff
3. f

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


