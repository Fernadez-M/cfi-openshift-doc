![header](../img/convergenewlogo.png)

# Nexus Repo Manager Integration with OpenShift 

- [Nexus Repo Manager Integration with OpenShift](#nexus-repo-manager-integration-with-openshift)
    - [Prerequisites](#prerequisites)
    - [Steps](#steps)

<div style="page-break-after: always;"></div>

Setting up a Sonatype Nexus 3 repository manager in CFI environment to act as an external private image registry for an OpenShift 4.12 cluster. Sonatype Nexus is one of the best repository managers out there. It is a tool that you cannot avoid in your CI/CD pipeline. It effectively manages deployable artifacts.

### Prerequisites

- Nexus Repo Manager Server installation completed and configured
- Docker private registry created on Nexus Repo 
- Registry Username and Password 
- Reverse proxy to handle TLS for Docker private registry 
    - In CFI environment, it's handle via F5 implementation 
    - Alternatively, configure nginx reverse proxy to server TLS certificate 

### Steps
- To use images from Docker private registry, OpenShift cluster requires to authenticate with Docker private registry. This is done by creating docker-registry `secret`.

    ```bash
    oc create secret docker-registry nexus-registry-secret --docker-server=<nexus-docker-registry-url> --docker-username=<username> --docker-password=<password>
    ```
> Note: Create above secret in a namespace where you are deploying your application.

- Create `configmap` on `openshift-config` namespace using TLS certificate data.

    ```bash
    oc create configmap nexus-registry-config -n openshift-config --from-file=<nexus-docker-registry-url>=<tls-certificate-path>
    ```
> Note: use CFI root CA as TLS Certificate

- Deploy `test` appliation from Nexus repo Docker private registry. 

    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
    name: nginx-deployment
    namespace: default
    labels:
        app: nginx
    spec:
    replicas: 3
    selector:
        matchLabels:
        app: nginx
    template:
        metadata:
        labels:
            app: nginx
        spec:
        imagePullSecrets:
        - name: nexus-registry-secret
        containers:
        - name: nginx
            image: <nexus-docker-registry-url>/nginx:latest
            ports:
            - containerPort: 80
    ```