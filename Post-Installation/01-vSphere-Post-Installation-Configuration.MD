![header](../img/convergenewlogo.png)

# vSphere Post Installation Configuration

- [vSphere Post Installation and Configuration](#vsphere-post-installation-configuration)
  - [Prerequisites](#prerequisites)
  - [Steps](#steps)

<div style="page-break-after: always;"></div>

After installing an OpenShift Container Platform cluster using the Assisted Installer on vSphere with the platform integration feature enabled, you must update the following vSphere configuration settings manually:
- vCenter username
- vCenter password
- vCenter address
- vCenter cluster
- datacenter
- datastore
- folder

### Prerequisites
- The Assisted Installer has finished installing the cluster successfully.
- The cluster is connected to `console.redhat.com`.

### Steps

1. Generate a base64-encoded username and password for vCenter:
    ```bash
    echo -n "<vcenter_username>" | base64 -w0
    echo -n "<vcenter_password>" | base64 -w0
    ```
    Replace `vcenter_username` and `vcenter-password` with your vCenter environment values.

2. Backup the vSphere credentials:

    `oc get secret vsphere-creds -o yaml -n kube-system > creds_backup.yaml`

3. Edit the vSphere credentials:
    ```bash
    cp creds_backup.yaml vsphere-creds.yaml
    vi vsphere-creds.yaml
    ```
    ```yaml
    apiVersion: v1
    data:
    r0b-vctr03.cfi.local.password: <value from step 1>
    r0b-vctr03.cfi.local.username: <value from step 1>
    kind: Secret
    metadata:
    annotations:
        cloudcredential.openshift.io/mode: passthrough
    name: vsphere-creds
    namespace: kube-system
    type: Opaque
    ```

4. Replace the vSphere credentials:

    `oc replace -f vsphere-creds.yaml`

5. Redeploy the kube-controller-manager pods:

    `oc patch kubecontrollermanager cluster -p='{"spec": {"forceRedeploymentReason": "recovery-'"$( date --rfc-3339=ns )"'"}}' --type=merge`

6. Backup the vSphere cloud provider configuration:

    `oc get cm cloud-provider-config -o yaml -n openshift-config > cloud-provider-config_backup.yaml`

7. Edit the cloud provider configuration:

    ```yaml
    apiVersion: v1
    data:
    config: |
        [Global]
        secret-name = "vsphere-creds"
        secret-namespace = "kube-system"
        insecure-flag = "1"

        [Workspace]
        server = "r0b-vctr03.cfi.local"
        datacenter = "CFI Raleigh"
        default-datastore = "Pure01"
        folder = "/CFI Raleigh/vm/OpenShiftQA"

        [VirtualCenter "r0b-vctr03.cfi.local"]
        datacenters = "CFI Raleigh"
    kind: ConfigMap
    metadata:
    creationTimestamp: "2023-05-15T17:23:03Z"
    name: cloud-provider-config
    namespace: openshift-config
    resourceVersion: "1917"
    uid: c5cc24d1-09f7-4a12-9644-f1be372e76e0
    ```
8. Apply the cloud provider configuration:

    `oc apply -f cloud-provider-config.yaml`

9. Check vSpehere Operator status:

    `oc get co`






