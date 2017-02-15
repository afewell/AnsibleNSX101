## NSX Ansible Discovery Modules
The NSX Ansible Modules works by interacting with the NSX REST API. When you make a call to the API, you are essentially asking it to take some action on some object, and you need to identify which objects you want to modify. For example, typically when you deploy a virtual machine, you need to deploy it into an existing cluster, onto an existing datastore, and connected to an existing portgroup. When using the GUI, you typically click on these values or use their common name, however when making calls to the API, you need to use special values so the API host knows exactly what you want to do. 

This section walks through the `vcenter_gather_moids` module, which is used to gather the Management IDs (MOIDs) of the cluster, datastore and portgroups used by later modules in this course. 

This section will introduce the use of multiple tasks in a single playbook. As you will see in the exercise, each task uses several of the same variables, so rather than including your variables directly within each task, it makes sense to use an external answer file to hold a common set of variables to be used in the play. 

1. Add variables needed to gather MOIDs to answerfile.yml
    + `cd ~/nsxansible`
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
targetClusters.cluster1.clusterName: 'RegionA01-COMP01'
nsxControllerDatastore: 'RegionA01-ISCI01-COMP01'
nsxControllerPortGroup: 'ESXi-RegionA01-vDS-COMP'
```

