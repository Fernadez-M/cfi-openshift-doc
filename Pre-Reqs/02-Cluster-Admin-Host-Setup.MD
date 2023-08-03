![header](../img/convergenewlogo.png)

# Setting Up Cluster Administration Host

- [Setting Up Cluster Administration Host](#setting-up-the-cluster-administration-host)
  - [1. Cluster Admin Host Specs](#1-cluster-admin-host-specs)
  - [2. Generate SSH Key](#2-generate-ssh-key)
  - [3. Download OCP 4.12.x Installation Sources](#3-download-ocp-412x-installation-sources)
    - [3.1 CLI tools download and install](#31-cli-tools-download-and-install)
    - [3.2 Enabling tab completion for Bash](#32-enabling-tab-completion-for-bash)
  - [4. Setup HTTP Server](#4-setup-http-server)
    - [4.1 Setup HTTP Server](#41-setup-http-server)
  - [5. Download and setup additional tools](#5-download-and-setup-additional-tools)
    - [5.1 govc download and install](#51-govc-download-and-install)

<br/>

The Cluster Administration Host acts as a jumpbox when ssh access is required directly to the OCP nodes. This helps in reducing attack surface by limiting the location of the OCP ssh keys. It also hosts a few key services required during the install process such as an HTTP server and DHCP server. For redundancy, 2 cluster admin hosts can be set up with both having the same contents.

## 1. Cluster Admin Host Specs

- **OS:** RHEL 8.x
- **CPU:** 2 vCPU
- **RAM:** 8 GB
- **Storage:** 80 GB disk space
- **Additional Yum repos:** `rhel-8-extras` for RHEL

## 2. Generate SSH Key

On the cluster admin host, generate the ssh-key that the `core` user can use to connect to the OCP cluster nodes. This key will be automatically added to the `~/.ssh/authorized_keys` list on the OCP nodes.

***Command***
``` bash
$ ssh-keygen -t rsa -b 4096 -f <path>/<file_name>
```

## 3. Download OCP 4.12.x Installation Sources

For this setup, we will download `OpenShift installer program` and `oc` command line interface.

### 3.1 CLI tools download and install

- The cli files for various platforms are hosted 
here 
https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/
- Download the `openshift-client-linux.tar.gz` and `openshift-install-linux.tar.gz` on the Cluster Admin host:
  ``` bash
  $ wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
  $ wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz
  ```
- Uncompress the archives and put the executables **`oc`**,**`kubectl`** and **`openshift-install`** in the Cluster Admin host's **`PATH`** such as **`/usr/local/bin`**

### 3.2 Enabling tab completion for Bash

After you install the OpenShift CLI (oc), you can enable tab completion to automatically complete oc commands or suggest options when you press Tab. The following procedure enables tab completion for the Bash shell.

-  Install bash completion package
    
    `yum -y install bash-completion`

- Save the Bash completion code to a file
    
    `oc completion bash > oc_bash_completion`

- Copy the file to /etc/bash_completion.d/

    `sudo cp oc_bash_completion /etc/bash_completion.d/` 

## 4. Setup HTTP Server

### 4.1 Setup HTTP Server

- Install httpd with yum
  ``` bash
  $ yum install -y httpd
  $ systemctl enable httpd && systemctl start httpd
  ```

## 5. Download and setup additional tools

| Tool | Version | Description | Company |
|--|--|--|--|
|**govc**| 0.24 | VMWare/Vsphere CLI | VMWare |

### 5.1 govc download and install

**govc** source is https://github.com/vmware/govmomi/tree/master/govc
- Download and setup **govc**:
  ```bash
  $ export GOVC_URL=https://github.com/vmware/govmomi/releases/download/v0.30.4/govc_Linux_x86_64.tar.gz
  $ curl -L ${GOVC_URL} | gunzip
  $ chmod +x govc
  $ sudo cp govc /usr/bin/govc
  ```
- Verification
  
  ``` bash
  $ govc version
  govc 0.30.4
  ```

- Create an env variables file as below

  > Note: This file should only be readable by an admin level linux account on the cluster Admin Host

  ```bash
  vim govc-creds.sh
  export GOVC_URL='vcenter.example.com'
  export GOVC_USERNAME='openshift_user@vsphere.local'
  export GOVC_PASSWORD='userpassword'
  export GOVC_INSECURE=1
  export GOVC_DATASTORE='Datastore'
  ```

- Verify that the login works
 
  ``` bash
  $ source govc-creds.sh
  $ govc about
  Name: VMware vCenter Server
  Vendor: VMware, Inc.
  Version: 7.x.x
  ...
