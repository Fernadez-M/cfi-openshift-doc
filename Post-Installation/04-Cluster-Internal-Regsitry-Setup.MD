![header](../img/convergenewlogo.png)

# Cluster Internal Registry Setup

- [Cluster Internal Registry Setup](#cluster-internal-registry-setup)
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
    # oc get config.imageregistry.operator.openshift cluster -o yaml
    apiVersion: imageregistry.operator.openshift.io/v1
    kind: Config
    metadata:
    creationTimestamp: "2023-05-15T17:49:42Z"
    finalizers:
    - imageregistry.operator.openshift.io/finalizer
    generation: 13
    name: cluster
    resourceVersion: "63887172"
    uid: 83029e97-030c-46c2-ad4a-74adc1f1a6cf
    spec:
    httpSecret: 40f4f0ecd45552bf3194bb168657fa08b05a7067c9d90cc8534382bf5dbee44e8772e9041b1bbce03f7aba3dd270c85529b3c76ee6630522e78eff37dc3acaba
    logLevel: Normal
    managementState: Managed
    observedConfig: null
    operatorLogLevel: Normal
    proxy: {}
    replicas: 1
    requests:
        read:
        maxWaitInQueue: 0s
        write:
        maxWaitInQueue: 0s
    rolloutStrategy: Recreate
    storage:
        managementState: Managed
        pvc:
        claim: internal-registry-storage # update with image-registry pvc claim 
    ```
4. Check registry pods are running in `openshift-image-registry` project.

    `oc get pods -n openshift-image-registry`