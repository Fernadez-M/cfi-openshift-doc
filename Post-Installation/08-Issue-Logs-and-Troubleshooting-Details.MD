![header](../img/convergenewlogo.png)

# Issue Logs and Troubleshooting Details

- [Issue Logs and Troubleshooting Details](#issue-logs-and-troubleshooting-details)
    - [Issue 1 - vSphere Connection Configuration from OpenShift](#issue-1---vsphere-connection-configuration-from-openshift)
    - [Issue 2 - LDAP Configuration on OpenShift](#issue-2---ldap-configuration-on-openshift)
    - [Issue 3 - LDAP Group Sync on OpenShift](#issue-3---ldap-group-sync-on-openshift)

<div style="page-break-after: always;"></div>

During CFI OpenShift UAT Cluster Installation and Post Configuration, below discovered issues were worked upon. This document provides details about a particular issue and troublehshooting steps taken for additional insights. 

### Issue 1 - vSphere Connection Configuration from OpenShift 

**Error**
- vSphere connection configuration: Invalid credentials
- VSphereCSIDriverOperatorCRProgressing: Waiting for VSphere operator to report status
    ```bash
    storage                4.12.15   True        True          False      15m     VSphereCSIDriverOperatorCRProgressing: Waiting for VSphere operator to report status
    ```

**Troubleshooting**

```bash
sh-4.4$ curl -kv [https://r0b-vctr03.cfi.local/sdk](https://r0b-vctr03.cfi.local/sdk)
* Uses proxy env variable NO_PROXY == '.cluster.local,.ral-oscq1.cfi.local,.svc,10.128.0.0/14,127.0.0.1,172.17.140.0/24,172.30.0.0/16,api-int.ral-oscq1.cfi.local,cfi.local,cfi.org,localhost,ncmentor.local'
*   Trying 172.17.53.180...
* TCP_NODELAY set
* connect to 172.17.53.180 port 443 failed: Connection timed out
* Failed to connect to r0b-vctr03.cfi.local port 443: Connection timed out
* Closing connection 0
curl: (7) Failed to connect to r0b-vctr03.cfi.local port 443: Connection timed out
```

- Updated OpenShift cluster-wide proxy `noProxy` field for additional values
- Discovered Asymmetrical routing on network switches, Network team added additional route to resolve this 

> Note: Above troubleshooting steps were performed on Control Plane nodes

### Issue 2 - LDAP Configuration on OpenShift 

**Error**
- OpenShift authentication failing with LDAP server

**Troubleshooting**

```bash
sh-4.4# curl -v  'ldaps://r0b-ctrl10.cfi.local/OU="ITOps Administrator Accounts",OU="CFI Special Accounts",DC=cfi,DC=local?sAMAccountName' -u sAMAccountName=_ldaposcp,OU="Service Accounts",OU="CFI Special Accounts",DC=cfi,DC=local:`cat /var/config/user/idp/0/secret/v4-0-config-user-idp-0-bind-password/bindPassword` --cacert /var/config/user/idp/0/configMap/v4-0-config-user-idp-0-ca/ca.crt
* Uses proxy env variable NO_PROXY == '.cluster.local,.ral-oscq1.cfi.local,.svc,10.128.0.0/14,127.0.0.1,172.17.140.0/24,172.30.0.0/16,api-int.ral-oscq1.cfi.local,cfi.local,cfi.org,localhost,ncmentor.local'
*   Trying 172.17.2.157...
* TCP_NODELAY set
* Connected to r0b-ctrl10.cfi.local (172.17.2.157) port 636 (#0)
* successfully set certificate verify locations:
*   CAfile: /var/config/user/idp/0/configMap/v4-0-config-user-idp-0-ca/ca.crt
  CApath: none
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to r0b-ctrl10.cfi.local:636
```
- Updated OpenShift cluster-wide proxy `noProxy` field for additional values
- Discovered Asymmetrical routing on network switches, Network team added additional route to resolve this 

> Note: Above troubleshooting steps were performed on Control Plane nodes

### Issue 3 - LDAP Group Sync on OpenShift 

**Error**
- LDAP Group Sync failing with `LDAP Result Code 49 "Invalid Credentials"`

**Troubleshooting**

```bash
# oc adm groups sync --sync-config=ldap-sync.yaml
apiVersion: v1
items: []
kind: List
metadata: {}
could not bind to the LDAP server: LDAP Result Code 49 "Invalid Credentials": 80090308: LdapErr: DSID-0C090439, comment: AcceptSecurityContext error, data 52e, v4563
error: could not bind to the LDAP server: LDAP Result Code 49 "Invalid Credentials": 80090308: LdapErr: DSID-0C090439, comment: AcceptSecurityContext error, data 52e, v4563
```
- Red Hat recommeded to update LDAPSyncConfig with `rfc2307` schema instead of `activeDirectory`
- Updated `certificate` , `bindPassword` and `filter`
- Please refer to [LDAP Group Sync section](/Post-Installation/02-LDAP-and-HTPasswd-Configuration.MD#configuring-an-ldap-group-sync)