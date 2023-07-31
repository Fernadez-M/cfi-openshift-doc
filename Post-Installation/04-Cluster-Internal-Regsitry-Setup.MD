![header](../img/convergenewlogo.png)

# Image Registry Setup on VMWare

- [Image Registry Setup on VMWare](#image-registry-setup-on-vmware)
    - [1. Image registry removed during installation](#image-registry-removed-during-installation)
    - [2. Changing the image registry's mamangement state](#2-changing-the-image-registrys-mamangement-state)
    - [3. Configurating registry storage for VMware vSphere](#3-configurating-registry-storage-for-vmware-vsphere)

<div style="page-break-after: always;"></div>

**Note**: This is for the Cluster's internal image registry, NOT Quay or Nexus.

### 1. Image registry removed during installation

On platforms that do not provide shareable object storage or storage with access mode `ReadWriteMany`, the OpenShift Image Registry Operator bootstraps itself as `Removed`. This allows openshift-installer to complete installations on these platform types.

After installation, you must edit the Image Registry Operator configuration to switch the `ManagementState` from `Removed` to `Managed`.

### 2. Changing the image registry's mamangement state 

To start the image registry, you must change the Image Registry Operator configuration’s `managementState` from `Removed` to `Managed`.

```bash
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed"}}'
```

### 3. Configurating registry storage for VMware vSphere
1. Create PVC claim of atleaset "**100GB**" in size.
    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
    name: internal-registry-storage
    namespace: openshift-image-registry 
    spec:
    accessModes:
    - ReadWriteOnce 
    resources:
        requests:
        storage: 100Gi 
    ```
2. Create the `PersistentVolumeClaim` in `openshift-image-registry` project.
    
    `oc create -f pvc.yaml -n openshift-image-registry`

    ```bash
    oc get pvc -A
    NAMESPACE                  NAME                        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    openshift-image-registry   internal-registry-storage   Bound    pvc-d74a57aa-2e43-4292-b3f8-8c2576c22024   100Gi      RWO            thin           55d
    ```

3. Edit registry configuration so that it references the correct PVC.

    `oc edit config.imageregistry.operator.openshift.io -o yaml`

    ```yaml
    capture additional info 
    ```
4. Check registry pods are running in `openshift-image-registry` project.

    `oc get pods -n openshift-image-registry`