# OpenShift MachineSets
Creating a MachineSet to build and scale nodes on VMware 6.7U3 and OCP 4.6

Ref: \
YouTube Video  \
OCP Documentation - https://docs.openshift.com/container-platform/4.6/machine_management/creating-infrastructure-machinesets.html

I usally recommend backing up a copy of one of your worker mainifest during install time to make this a bit easier, but you can use the templates found in the documentation to create your instance(s), per your environment.

1. Retrieve your unique infrastrutor id. \
`oc get -o jsonpath='{.status.infrastructureName}{"\n"}' infrastructure cluster`

2. Edit (or Create) your MachineSet yaml. \
`vi 99_openshift-cluster-api_worker-machineset-0.yaml`

3. In VMware, pay particutlar attention to the "workspace" section, in my video, I did not have a "Resource Pool", you may otherwise the inframtion will be what you completed as part of your install-config.yaml
    ```
    workspace:
      datacenter: <vcenter_datacenter_name> 
      datastore: <vcenter_datastore_name> 
      folder: <vcenter_vm_folder_path> 
      resourcepool: <vsphere_resource_pool> 
      server: <vcenter_server_ip
    ```

4. You also should include a label for your machine "role", here again I was using "infra" in the video. If you do not include a specific role, it will default to "worker" only. 
    ``` 
    spec:
        metadata:
        labels:
            node-role.kubernetes.io/<role>: ""
    ```

5. Verify your compute values...
    ```
    diskGiB: 120
    memoryMiB: 8192
    numCPUs: 4
    numCoresPerSocket: 1
    network:
    devices:
        - networkName: "<vm_network_name>" 
    ```
    and your template name. 
    ```
    template: <vm_template_name>
    ```

6. Make sure your userDataSecret remains unchanged.
    ```
    userDataSecret:
    name: worker-user-data
    ```

7. Install your MachineSet \
`oc create -f 99_openshift-cluster-api_worker-machineset-0.yaml`

8. Access the UI and increment your # of Machines or set the replica value \
Scale by OC... \
a. View your machinesets \
    `oc get machinesets -n openshift-machine-api` \
b. Scale the machineset \
    `oc scale --replicas=${#} machineset ${machinesetname} -n openshift-machine-api`
