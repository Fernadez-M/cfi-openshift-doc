![header](../img/convergenewlogo.png)

# Cluster-wide Proxy Configuration

- [Cluster-wide Proxy Configuration](#cluster-wide-proxy-configuration)
    - [Enabling the cluster-wide proxy](#enabling-the-cluster-wide-proxy)

<div style="page-break-after: always;"></div>

Production environments can deny direct access to the internet and instead have an HTTP or HTTPS proxy available. You can configure OpenShift Container Platform to use a proxy by modifying the Proxy object for existing clusters or by configuring the proxy settings in the install-config.yaml file for new clusters.

### Enabling the cluster-wide proxy
The `Proxy` object is used to manage the cluster-wide egress proxy. When a cluster is installed or upgraded without the proxy configured, a Proxy object is still generated but it will have a nil `spec`. 

- Use `oc edit` command to modify the `Proxy` object

    `oc edit proxy/cluster`

    ```yaml
    # oc get proxy cluster -o yaml
    apiVersion: config.openshift.io/v1
    kind: Proxy
    metadata:
    creationTimestamp: "2023-05-15T17:22:15Z"
    generation: 3
    name: cluster
    resourceVersion: "8659895"
    uid: f539fbcc-67dc-4820-ad68-2c04cd1b42f6
    spec:
    httpProxy: http://r0b-prxy06.cfi.local:8080
    httpsProxy: http://r0b-prxy06.cfi.local:8080
    noProxy: .cfi.local,cfi.local,r0b-ctrl10.cfi.local,r0b-ctrl11.cfi.local,ncmentor.local,cfi.org,.ral-oscq1.cfi.local,ral-oscq1.cfi.local,10.128.0.0/14,172.30.0.0/16,172.17.140.0/24
    trustedCA:
        name: custom-ca
    status:
    httpProxy: http://r0b-prxy06.cfi.local:8080 # add CFI proxy details
    httpsProxy: http://r0b-prxy06.cfi.local:8080 # add CFI proxy details 
    noProxy: .cfi.local,.cluster.local,.ral-oscq1.cfi.local,.svc,10.128.0.0/14,127.0.0.1,172.17.140.0/24,172.30.0.0/16,api-int.ral-oscq1.cfi.local,cfi.local,cfi.org,localhost,ncmentor.local,r0b-ctrl10.cfi.local,r0b-ctrl11.cfi.local,ral-oscq1.cfi.local # update required values for noProxy field
    ```


