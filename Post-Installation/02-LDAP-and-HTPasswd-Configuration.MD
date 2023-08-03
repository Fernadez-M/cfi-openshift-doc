![header](../img/convergenewlogo.png)

# Configuring Identity Providers

- [Configuring Identity Providers](#configuring-identity-providers)
  - [Identity providers in OpenShift Container Platform](#identity-providers-in-openshift-container-platform)
  - [Configuring an htpasswd identify provider](#configuring-an-htpasswd-identity-provider)
    - [Creating htpasswd file](#creating-htpasswd-file)
    - [Creating htpasswd secret](#creating-htpasswd-secret)
    - [Adding an htpasswd provider to Cluster using CR](#adding-an-htpasswd-identity-provider-to-cluster-using-cr)
  - [Configuring an LDAP identiy provider](#configuring-an-ldap-identity-provider)
    - [Creating the LDAP secret](#creating-the-ldap-secret)
    - [Creating a config map](#creating-a-config-map)
    - [Adding an LDAP identity provide to Cluster using CR](#adding-an-ldap-identify-provider-to-cluster-using-cr)
  - [Configuring an LDAP Group Sync](#configuring-an-ldap-group-sync)
  
<div style="page-break-after: always;"></div>
  
## Identity providers in OpenShift Container Platform

By default, only a `kubeadmin` user exists on your cluster. To specify an identity provider, you must create a custom resource (CR) that describes that identity provider and add it to the cluster.

## Configuring an htpasswd identity provider
To define an htpasswd identity provider, perform the following tasks:
- Create an htpasswd file to store the user and password information.
- Create a secret to represent the htpasswd file.
- Define an htpasswd identity provider resource that references the secret.
- Apply the resource to the default OAuth configuration to add the identity provider.

### Creating htpasswd file
1. Create or update your flat file with a user name and hashed password:

    `htpasswd -c -B -b </path/to/users.htpasswd> <username> <password>`

2. Continue to add or update credentials to the file:

    `htpasswd -B -b </path/to/users.htpasswd> <user_name> <password>`

### Creating htpasswd secret
1. Create a `Secret` object that contains the `htpasswd` users file:

    `oc create secret generic htpass-secret --from-file=htpasswd=<path_to_users.htpasswd> -n openshift-config`

### Adding an htpasswd identity provider to cluster using CR

```yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: cfi_htpasswd_provider
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
```
```bash 
oc apply -f <path-to-cr-file>
oc login -u <username>

oc whoami
```

## Configuring an LDAP identity provider

### Creating the LDAP secret
 1. Create a `Secret` object that contains the `bindPassword` field: 

    `oc create secret generic ldap-secret --from-literal=bindPassword=<secret> -n openshift-config`

    The secret key containing the bindPassword for the `--from-literal` argument must be called `bindPassword`.

### Creating a config map
Define an OpenShift Container Platform `ConfigMap` object containing the certificate authority by using the following command. The certificate authority must be stored in the `ca.crt` key of the `ConfigMap` object.

`oc create configmap ca-config-map --from-file=ca.crt=/path/to/ca -n openshift-config`

### Adding an LDAP identify provider to Cluster using CR 

1. Create below custome resource with **CFI LDAP** configuration details 

> Note: Append LDAP configuration details to htpasswd CR from above section


  ```yaml
  apiVersion: config.openshift.io/v1
  kind: OAuth
  metadata:
    name: cluster
  spec:
    identityProviders:
    - name: cfi_htpasswd_provider
      mappingMethod: claim
      type: HTPasswd
      htpasswd:
        fileData:
          name: htpass-secret
    - name: ldapidp
      mappingMethod: claim
      type: LDAP
      ldap:
        attributes:
          id:
          - dn
          email:
          - mail
          name:
          - cn
          preferredUsername:
          - sAMAccountName
        bindDN: "_ldaposcp"
        bindPassword:
          name: ldap-secret
        ca:
          name: ldap-ca-config-map
        insecure: false
        url: "ldaps://r0b-ctrl10.cfi.local/DC=cfi,DC=local?sAMAccountName?sub"
  ```
  ```bash
  oc apply -f </path/to/CR>
  oc login -u <username>

  oc whoami 
  ```

## Configuring an LDAP Group Sync

1. Create `LDAPSyncConfig` file 

  ```yaml
  # cat ldap-sync-bkup.yaml
  kind: LDAPSyncConfig
  apiVersion: v1
  url: ldaps://r0b-ctrl10.cfi.local
  insecure: false
  ca: /root/oscq/certs/cfi_local_root_2030.crt
  bindDN: "CN=_ldaposcp,OU=Service Accounts,OU=CFI Special Accounts,DC=cfi,DC=local"
  bindPassword: "<bind-password>"
  #  file: "/root/oscq/ldap/bindpassword.txt"
  groupUIDNameMapping:
    "CN=OpenShift_Administrator,OU=OpenShift,OU=Application Groups,OU=CFI_Groups,DC=cfi,DC=local": Administrators
    "CN=OpenShift_Developer,OU=OpenShift,OU=Application Groups,OU=CFI_Groups,DC=cfi,DC=local": Developers
  #activeDirectory:
  #  usersQuery:
  #    baseDN: "DC=cfi,DC=local"
  #    scope: sub
  #    derefAliases: never
  #    filter: (objectclass=person)
  #    pageSize: 0
  #  userNameAttributes: [ sAMAccountName ]
  #  groupMembershipAttributes: [ memberOf ]
  #
  rfc2307:
      groupsQuery:
          baseDN: "ou=OpenShift,ou=Application Groups,ou=CFI_Groups,dc=cfi,dc=local"
          #baseDN: "dc=cfi,dc=local"
          scope: sub
          derefAliases: never
          pageSize: 0
          filter: (objectclass=*)
      groupUIDAttribute: dn
      groupNameAttributes: [ cn ]
      groupMembershipAttributes: [ member ]
      usersQuery:
          baseDN: "dc=cfi,dc=local"
          scope: sub
          derefAliases: never
          pageSize: 0
      userUIDAttribute: dn
      userNameAttributes: [ sAMAccountName ]
      tolerateMemberNotFoundErrors: false
      tolerateMemberOutOfScopeErrors: false
  ```
2. Run LDAP Group Sync using below command
  ```bash
  $ oc adm groups sync --sync-config=ldap-sync.yaml --confirm
  ```

3. Verify LDAP Group Sync
  ```bash
  $ oc get groups
  ```