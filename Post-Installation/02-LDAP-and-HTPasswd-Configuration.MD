![header](../img/convergenewlogo.png)

# Configuring Identity Providers

- [Configuring Identity Providers](#configuring-identity-providers)
  - [Identity providers in OpenShift Container Platform](#identity-providers-in-openshift-container-platform)
  - [Configuring ad htpasswd identify provider](#configuring-an-htpasswd-identity-provider)
    - [Creating htpasswd file](#creating-htpasswd-file)
    - [Creating htpasswd secret](#creating-htpasswd-secret)
    - [Adding an htpasswd provider to Cluster using CR](#adding-an-htpasswd-identity-provider-to-cluster-using-cr)
  - [Configuring an LDAP identiy provider](#configuring-an-ldap-identity-provider)
    - [Creating the LDAP secret](#creating-the-ldap-secret)
    - [Creating a config map](#creating-a-config-map)
    - [Adding an LDAP identity provide to Cluster using CR](#adding-an-ldap-identify-provider-to-cluster-using-cr)
  
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