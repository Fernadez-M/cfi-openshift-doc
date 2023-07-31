![header](../img/convergenewlogo.png)

# Securing API and Apps Ingress Router with TLS/SSL Certificates

- [Securing API and Apps Ingress Router with TLS/SSL Certificates](#securing-api-and-apps-ingress-router-with-tlsssl-certificates)
  - [Notes for Generating CSRs](#notes-for-generating-csrs)
  - [Replacing Ingress Certificate](#replacing-ingress-certificate)
  - [Adding API Server Certificate](#adding-api-server-certificate)

<div style="page-break-after: always;"></div>

## Notes for Generating CSRS

- Generate separate CSR request for `api` and `*.apps` endpoints. 
- CSRs will need to be created with the SAN (**`subjectAltName`**) extension.
- The `api` certificate is required to have api **`subjectAltName`** extension of **`*.apps.<clustername>.<domain>`**
- The `ingress` router certificate is required to have a wildcard **`subjectAltName`** extension of **`*.apps.<clustername>.<domain>`**

**CSR request example**
```bash
[ req ]  
default_bits       = 2048  
distinguished_name = req_distinguished_name  
req_extensions     = req_ext  
[ req_distinguished_name ]  
countryName                = US
stateOrProvinceName        = North Carolina
localityName               = Raleigh
organizationName           = CFI
commonName                 = api.ral-oscql1.cfi.local
[ req_ext ]  
subjectAltName = @alt_names  
[alt_names]  
DNS.1   = api.ral-oscq1.cfi.local
DNS.2   = *.apps.ral-oscq1.cfi.local
```

> **DO NOT** replace the certificate for **`api-int.<clustername>.<domain>`**, doing so can lead to cluster stopping to function.


## Replacing Ingress Certificate

### Prerequisites
- You must have a wildcard certificate for the fully qualified .apps subdomain and its corresponding private key. Each should be in a separate PEM format file.
- The private key must be unencrypted. If your key is encrypted, decrypt it before importing it into OpenShift Container Platform.
- The certificate must include the subjectAltName extension showing `*.apps.<clustername>.<domain>.`
- The certificate file can contain one or more certificates in a chain. The wildcard certificate must be the first certificate in the file. It can then be followed with any intermediate certificates, and the file should end with the root CA certificate.
- Copy the root CA certificate into an additional PEM format file.
- Save the signed certificate and key with these names for easier use

  ```bash
  wildcard-fullchain-cert.crt
  wildcard-cert-key.key
  ```

### Steps
1. Create a config map that includes only the root CA certificate used to sign the wildcard certificate:
    ```bash
    oc create configmap custom-ca --from-file=ca-bundle.crt=</path/to/root-ca-file.crt> -n openshift-config
    ```
2. Update the cluster-wide proxy configuration with the newly created config map:
    ```bash
    oc patch proxy/cluster --type=merge --patch='{"spec":{"trustedCA":{"name":"custom-ca"}}}'
    ```
3. Create a secret that contains the wildcard certificate chain and key:
    ```bash
    oc create secret tls <secret> --cert=</path/to/fullchain-cert.crt> --key=</path/to/cert-private.key> -n openshift-ingress
    ```
4. Update the Ingress Controller configuration with the newly created secret:
    ```bash
    oc patch ingresscontroller.operator default --type=merge -p '{"spec":{"defaultCertificate": {"name": "<secret>"}}}' -n openshift-ingress-operator
    ```

## Adding API server certificate 

### Prerequisites
- You must have a wildcard certificate for the fully qualified .apps subdomain and its corresponding private key. Each should be in a separate PEM format file.
- The private key must be unencrypted. If your key is encrypted, decrypt it before importing it into OpenShift Container Platform.
- The certificate must include the subjectAltName extension showing `*.apps.<clustername>.<domain>.`
- The certificate file can contain one or more certificates in a chain. The certificate for the API server FQDN must be the first certificate in the file. It can then be followed with any intermediate certificates, and the file should end with the root CA certificate.
- Save the signed certificate and key with these names for easier use

  ```bash
  api-fullchain-cert.crt
  api-cert-key.key
  ```

### Steps
1. Create a secret that contains the certificate chain and private key in the openshift-config namespace.
    ```bash
    oc create secret tls <secret> --cert=</path/to/api-fullchain-cert.crt> --key=</path/to/cert-private.key> -n openshift-config
    ```
2. Update the API server to reference the created secret.
    ```bash
    oc patch apiserver cluster --type=merge -p '{"spec":{"servingCerts": {"namedCertificates": [{"names": ["<FQDN>"], "servingCertificate": {"name": "<secret>"}}]}}}' 
    ```
    FQDN: obtain using `oc whoami --show-server` command.
3. Examine the apiserver/cluster object and confirm the secret is now referenced.
4. Check the kube-apiserver operator, and verify that a new revision of the Kubernetes API server rolls out.

    `oc get clusteroperators kube-apiserver`
    