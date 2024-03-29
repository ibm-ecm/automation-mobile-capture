# AMC Installation and Deployment Guide

## Index
* [Overview](#1-overview)
* [Prerequisites](#2-prerequisites)
    * [Versions](#21-versions)
    * [Sizing](#22-sizing)
* [Configuration](#3-configuration)
    * [Standard Kubernetes or OpenShift](#31-standard-kubernetes-or-openshift)
    * [OpenShift on IBM Cloud](#32-openshift-on-ibm-cloud)
    * [Azure AKS](#33-azure-aks)
    * [Amazon EKS](#34-amazon-eks)
* [Deploying](#4-deploying)
    * [Uploading your images](#41-uploading-your-images)
    * [New Deployment](#42-new-deployment)
    * [Upgrading an existing deployment](#43-upgrading-an-existing-deployment)
* [Possible Errors](#5-possible-errors)
    * [Upgrading an existing deployment](#51-upgrading-an-existing-deployment)

## 1. Overview
This guide provides the information on how to deploy IBM Automation Mobile Capture (AMC) on a Kubernetes certified cluster.

It includes the information specific to common Kubernetes providers beyond generic instructions appliable to a standard cluster: Azure AKS, Amazon AWS EKS and OpenShift.

## 2. Prerequisites

### 2.1. Versions
* Kubernetes >= v1.15 or OpenShift >= v3.11 as a deployment target
* Helm v3.x.x installed on the machine you deploy from
* PostgreSQL Server >= v10 reachable from the deployment cluster

### 2.2. Sizing

#### Processing
The following table expresses the CPU requirement in terms of vCPU units. This means that in order to run AMC you should have 4 cores assigned in total, where the following distribution in the table:
|                    | vCPU |
|--------------------|------|
| Main Application   | 1    |
| Application Worker | 1    |
| HTTP Static Server | 1    |
| Attachments Worker | 1    |

The underlying hardware should be of similar or better performance of one any of the following:
* Intel® Xeon® 8171M 2.1GHz (Skylake)
* Intel® Xeon® E5-2673 v4 2.3 GHz (Broadwell) 
* Intel® Xeon® E5-2673 v3 2.4 GHz (Haswell) processors

#### Memory
For running on single node, AMC needs at least 12GiB total memory, which is recommend.

If components are running in different nodes, the requirements are defined as follows:
|                    | Memory (GiB) |
|--------------------|--------------|
| Main Application   | 2            |
| Application Worker | 4            |
| HTTP Static Server | 1            |
| Attachments Worker | 4            |

#### Typical Machines Sizes
|                    | Size            |
|--------------------|-----------------|
| Azure AKS          | Standard_D4s_v3 |
| AWS EKS            | m4.xlarge       |

## 3. Configuration

### 3.1. Standard Kubernetes or OpenShift

#### kubectl
To connect with your Kubernetes cluster, you need to have _kubectl CLI_ installed on the machine you are using to deploy. If you are already managing a Kubernetes cluster, this should already be installed on your machine.

If it is not the case, you can follow the instructions provided [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

### Helm
To deploy AMC, you need to have Helm installed on the machine you are using to deploy.

Follow the instructions for your platform provided [here](https://helm.sh/docs/intro/install/).

---
**ℹ️ Quick macOS install with `brew`**
```bash
brew install helm
```
---

#### Accessing your cluster
Before you proceed with deploying the *Helm chart* you need to have access to your Kubernetes cluster.

Contact your cluster administrator to provide you with the required access instructions.

#### Ingress Controller
It is required to have an _Ingress Controller_ available on your cluster. AMC uses `Ingress` resources to configure access to the product.
You should contact your cluster administrator to get information about this.

If you do not have a provisioned _Ingress Controller_ you can follow the instructions for [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/).
Refer to the section for easy deployment using Helm.

#### _PostgreSQL_ Server
You need to provide a _PostgreSQL_ Server with version >= 10.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to access it.

If you do not have one, and your deployment is on a standard certified Kubernetes cluster, there is an optional _Helm Chart_ dependency provided that deploys a third-party _PostgreSQL_ deployment alongside with AMC.

To enable this, before installing the _Helm Chart_, edit `values.yaml` and set `postgresqlInternalServer.enabled` to `true`.

If you are on OpenShift, the platform provides an inbuilt Operator that is capable of deploying a fully functional PostgreSQL Server. You may need to configure cluster networking configurations to allow Pod domain resolution and network reachability.

#### Persistent Volume provisioner
For this deployment to function, the cluster must be able to provision a persistent volume that supports `ReadWriteMany` mode (`RWX`).
To support dynamic provisioning, a StorageClass that provisions volumes in `RWX` mode must be made available.

Contact your cluster administrator to provide you with the required instructions.

#### Domain and TLS Certificate
Due to security concerns, it is required that the server is reachable via HTTPS. 
You will need the provide the appropriate Secret with the TLS certificate for a valid domain.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to configure it.

If you already have a correctly configured public DNS zone that points to the cluster, you can install _cert-manager_, to issue and manage certificates automatically, provided by `Let's Encrypt`, following the instructions [here](https://cert-manager.readthedocs.io/en/latest/tutorials/acme/quick-start/index.html).

#### SMTP Server
This product requires an SMTP server that can send invitation emails, as well as password recovery emails. It is not possible to add users to the platform without a correctly configured SMTP Server.

#### Docker images
The docker images (`ibm/mobile-capture-api` and `ibm/mobile-capture-static-file-server`) need to be present on a docker image registry accessible by the cluster.
If the registry is private, you can configure the pull secrets required to pull the image from the registry.
Contact your cluster administration for this information.

#### Secrets
Before the initial deployment, you can use the following operations:

Create image pull secret for the registry where the docker images are located at. This is dependent on where the images are stored, your cluster administrator should be able to assist you on this. Keep a note of the image pull secret name.

##### Create secret key base secret
```bash
kubectl create secret generic mobilecapture --from-literal=secret-key-base=$(openssl rand -hex 64)
```
##### Create SMTP password secret
```bash
$ kubectl create secret generic mobilecapture-smtp --from-literal=smtp-password=<smtp server password>
```
##### Create seed admin user password
```bash
$ kubectl create secret generic mobilecapture-admin-seed --from-literal=password=<admin seed password>
```
##### Create PostgreSQL database secret
```bash
$ kubectl create secret generic mobilecapture-postgresql --from-literal=postgresql-password="$(openssl rand -hex 32)”
```
Replace `"$(openssl rand -base64 32)”` with your own password if you are connecting to an existing PostgreSQL server.

#### `values.yaml` Configuration
In the directory of the AMC _Helm Chart_ you can find a file named `values.yaml`.
Use your editor of preference to edit and update the file so that its contents remain according to the following template:
```yaml
postgresql:
  existingSecret: mobilecapture-postgresql
images:
  pullSecrets:
  - <image pull secret name>
  rails:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-api
  react:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-static-file-server
existingSecret: mobilecapture
authentication: internal
seed:
  adminUser:
    username: 'admin@ibm.com'
    passwordSecretName: mobilecapture-admin-seed
hostname: <hostname for the deployed server>
smtpConfiguration:
  senderName: IBM Mobile Capture
  senderEmail: donotreply@<hostname for the deployed server>
  domain: <smtp email domain>
  address: <smtp address>
  port: <smtp port>
  username: <smtp username>
  passwordSecret: mobilecapture-smtp
tls:
  host: <hostname for the deployed server>
  secret: <tls secret name>
```

---
**NOTE**

Indentation on YAML files is important, so make sure you use the correct indentation spacing.

---

---
**NOTE**

If you decided to deploy an internal PostgreSQL server automatically, set on `values.yaml`:
```yaml
postgresqlInternalServer:
  enabled: true
```

---


#### LDAP Configuration
Under `ldap` set the values for:
- `host`: LDAP server address.
- `port`: LDAP server port.
- `treebase`: The treebase where the user search can be done.
- `emailAttribute`: The LDAP attribute where the user's email is stored.
- `adminsGroup`: The LDAP group users, who must be members having access to workspace admin role on Mobile Capture.
- `usersGroup`: The LDAP group users, who must be members having access to Mobile Capture.
- `enableLDAPS`: Enable LDAP over SSL
- `ldapsCertificateLocation`: LDAPS certificate file location, in PEM format, relative to the Chart's root. If LDAPS is active, this field should contain the LDAP over SSL server’s public certificate in PEM format (.pem), the path is relative to the helm chart’s root directory. It validates the server's authenticity against this certificate. It can be a CA root certificate, or a self-signed certificate. If you save the public certificate named as `ldaps_cert.pem` and copy it to the same directory as the helm chart, the content of `ldapsCertificateLocation`  should be `ldaps_cert.pem`.

```yaml
authentication: internal
ldap:
  host: <ldap host>
  port: <ldap port>
  treebse: <treebase>
  emailAttribute: <email attribute>
  adminGroup: <admin group>
  usersGroup: <users group>
  enableLDAPS: <true/false>
  ldapsCertificateLocation: 'certificate.pem'
```

#### External OpenID Connect Configuration

Read instructions [here](AMC_OIDC_Configuration_Guide.md) on how to configure an external OIDC Provider as an authentication provider.

#### Virtual Root Configuration
If traffic into the cluster is rooted by an external proxy employing a URL rewrite strategy, effectively creating a virtual root, you need to define the virtual root's path on the configuration.
This option does not serve Automation Mobile Capture under a virtual root, but is needed in order to generate links correctly for inviting new users, reset password, upload endpoint URLs, etc.
This value is used in conjunction with the value of hostname.
The value must begin with a forward slash, and terminate without a trailing slash.
For example:
    - You employ a URL rewrite strategy to serve mobile capture at https://www.example.com/mobilecapture
    - Set the value of `hostname` to `www.example.com`
    - Set the value of `virtualRoot` to `/mobilecapture`

```yaml
hostaname: www.example.com
virtualRoot: /mobilecapture
```


### 3.2. OpenShift on IBM Cloud

#### OpenShift Container Platform command-line interface
To connect with your OpenShift cluster, you need to have _OpenShift CLI_ installed on the machine you are using to deploy. If you are already managing an OpenShift cluster, this should already be installed on your machine.

If it is not the case, you can follow the instructions provided [here](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html#cli-installing-cli_cli-developer-commands)

---
**ℹ️ Quick macOS install with `brew`**
```bash
brew install openshift-cli
```
---

#### kubectl
To connect with your Kubernetes cluster, you need to have _kubectl CLI_ installed on the machine you are using to deploy. If you are already managing a Kubernetes cluster, this should already be installed on your machine.

If it is not the case, you can follow the instructions provided [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

---
**ℹ️ Quick macOS install with `brew`**
```bash
brew install kubernetes-cli
```
---

#### Helm
To deploy AMC you need to have Helm installed on the machine you are using to deploy.

Follow the instructions for your platform provided [here](https://helm.sh/docs/intro/install/).

---
**ℹ️ Quick macOS install with `brew`**
```bash
brew install helm
```
---


#### Accessing your cluster
Before you proceed with deploying the *Helm chart* you need to have access to your OpenShift cluster.

1. Login into **[IBM Cloud](https://cloud.ibm.com)**.
1. Open the **Openshift dashboard**
1. Click on the target cluster to view details
1. On this page, click the **OpenShift** web console* button
1. Create new OpenShift project:
    1. Click the project selector and click **New project**
    1. Set project name and click **Create**
1. Login to the OpenShift cluster on the terminal:
    1. On the cluster details page, click your username on the top right corner
    1. Click **Copy login command**
    1. On the new page, click **Display Token**
    1. Copy the content of `Log in with this token` field
    1. Paste the command on your terminal and execute
    1. Change the OpenShift CLI current project to the one where you will install AMC
        1. `oc project <project name>`


#### Ingress Controller
It is required to have an _Ingress Controller_ available on your cluster. AMC uses `Ingress` resources to configure access to the product.

OpenShift on IBM Cloud already provides an Ingress controller.


#### _PostgreSQL_ Server
You need to provide a _PostgreSQL_ Server with version >= 10.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to access it.

If you do not have one, you can use the inbuilt OpenShift operator:

On the **OpenShift web console** page:
  1. Click **+Add**
  1. Click **From catalog**
  1. Click **PostgreSQL *Provided by Red Hat, Inc.***
  1. Click **Instanciate template**
  1. Configure accordingly
  1. Take note of:
      1. `Database Service Name` (this is the hostname to be used when configuring the Helm Chart's values)
      1. `PostgreSQL Connection Username` (if you used generated values, you can find them on the `Secret` generated by the template)
      1. `PostgreSQL Connection Password` (if you used generated values, you can find them on the `Secret` generated by the template)
      1. `PostgreSQL Database Name`

#### Persistent Volume provisioner
For the deployment to function, the cluster must be able to provision a persistent volume that supports `ReadWriteMany` mode (`RWX`).
To support dynamic provisioning, a StorageClass that provisions volumes in `RWX` mode must be made available.

OpenShift on IBM Cloud already provide storage classes that support this.
Any of the `ibmc-file-*` provided storage classes are compatible with AMC.
Please choose a storage class according to your needs.

#### Domain and TLS Certificate
Due to security concerns, it is required that the server is reachable via HTTPS. 
You need to provide the appropriate Secret with the TLS certificate for a valid domain.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to configure it.

If you already have a correctly configured public DNS zone that points to the cluster, you can install _cert-manager_, to issue and manage certificates automatically, provided by `Let's Encrypt`, following the instructions [here](https://cert-manager.io/docs/installation/openshift/).

#### SMTP Server
This product requires an SMTP server that can send invitation emails, as well as password recovery emails. It is not possible to add users to the platform without a correctly configured SMTP server.

#### Docker images
The docker images (`ibm/mobile-capture-api` and `ibm/mobile-capture-static-file-server`) need to be present on a docker image registry accessible by the cluster.
If the registry is private, you can configure the pull secrets required to pull the image from the registry.
Contact your cluster administration for this information.

#### Secrets
Before the initial deployment, you can use the following operations:
Create image pull secret for the registry where the docker images are located at. This is dependent on where the images are stored, your cluster administrator should be able to assist you on this. Keep a note of the image pull secret name.

##### Create secret key base secret
```bash
kubectl create secret generic mobilecapture --from-literal=secret-key-base=$(openssl rand -hex 64)
```
##### Create SMTP password secret
```bash
$ kubectl create secret generic mobilecapture-smtp --from-literal=smtp-password=<smtp server password>
```
##### Create seed admin user password
```bash
$ kubectl create secret generic mobilecapture-admin-seed --from-literal=password=<admin seed password>
```
##### Create PostgreSQL database secret
```bash
$ kubectl create secret generic mobilecapture-postgresql --from-literal=postgresql-password="<PostgreSQL Connection Password>”
```
Replace `"<PostgreSQL Connection Password>”` with the password you configured when deploying the PostgreSQL server operator.

#### `values.yaml` Configuration
In the directory of the AMC _Helm Chart_ you can find a file named `values.yaml`.
Use your editor of preference to edit and update the file so that its contents remain according to the following template:
```yaml
postgresClient:
  host: <Database Service Name>
  port: 5432
  username: <PostgreSQL Connection Username>
  database: <PostgreSQL Database Name>
  secret: mobilecapture-postgresql
postgresqlInternalServer:
  enabled: false
images:
  pullSecrets:
  - <image pull secret name>
  rails:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-api
  react:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-static-file-server
existingSecret: mobilecapture
authentication: internal
seed:
  adminUser:
    username: 'admin@ibm.com'
    passwordSecretName: mobilecapture-admin-seed
hostname: <hostname for the deployed server>
smtpConfiguration:
  senderName: IBM Mobile Capture
  senderEmail: donotreply@<hostname for the deployed server>
  domain: <smtp email domain>
  address: <smtp address>
  port: <smtp port>
  username: <smtp username>
  passwordSecret: mobilecapture-smtp
tls:
  host: <hostname for the deployed server>
  secret: <tls secret name>
```

The values of `<Database Service Name>`, `<PostgreSQL Connection Username>`, and `<PostgreSQL Database Name>` are the ones provided when deploying the OpenShift inbuilt PostgreSQL operator.

To use the IBM provided storage class:
```yaml
uploadPersistence:
  enabled: true
  storageClass: ibmc-file-*
  size: 16Gi
  subPath: ""
```
Replace `ibmc-file-*` with the storage class you selected.

---
**NOTE**

Indentation on YAML files is important, make sure you use the correct indentation spacing.

---


#### LDAP Configuration
Under `ldap` set the values for:
- `host`: LDAP server addresss.
- `port`: LDAP server port.
- `treebase`: The treebase where the user search can be done.
- `emailAttribute`: The LDAP attribute where the user's email is stored.
- `adminsGroup`: The LDAP group users, who must be members having access to workspace admin role on Mobile Capture.
- `usersGroup`: The LDAP group users, who must be members having access to Mobile Capture.
- `enableLDAPS`: Enable LDAP over SSL
- `ldapsCertificateLocation`: 
If LDAPS is active, this field should contain the LDAP over SSL server’s public certificate in PEM format (.pem), the path is relative to the helm chart’s root directory. It validates the server's authenticity against this certificate. It can be a CA root certificate, or a self-signed certificate. If you save the public certificate named as `ldaps_cert.pem` and copy it to the same directory as the helm chart, the content of `ldapsCertificateLocation`  should be `ldaps_cert.pem`.


```yaml
authentication: internal
ldap:
  host: <ldap host>
  port: <ldap port>
  treebse: <treebase>
  emailAttribute: <email attribute>
  adminGroup: <admin group>
  usersGroup: <users group>
  enableLDAPS: <true/false>
  ldapsCertificateLocation: 'certificate.pem'
```

#### External OpenID Connect Configuration

Read instructions [here](AMC_OIDC_Configuration_Guide.md) on how to configure an external OIDC Provider as an authentication provider.

#### Virtual Root Configuration
If traffic into the cluster is rooted by an external proxy employing a URL rewrite strategy, effectively creating a virtual root, you need to define the virtual root's path on the configuration.
This option does not serve mobile capture under a virtual root, but it is needed in order to generate links correctly for inviting new users, reset password, upload endpoint URLs, etc.
This value is used in conjunction with the value of hostname.
The value must begin with a forward slash, and terminate without a trailing slash.
For example:
    - You employ a URL rewrite strategy to serve mobile capture at https://www.example.com/mobilecapture
    - Set the value of `hostname` to `www.example.com`
    - Set the value of `virtualRoot` to `/mobilecapture`

```yaml
hostaname: www.example.com
virtualRoot: /mobilecapture
```

### 3.3. Azure AKS

#### kubectl
To connect with your Kubernetes cluster, you need to have _kubectl CLI_ installed on the machine you are using to deploy. If you are already managing a Kubernetes cluster, this should already be installed on your machine.

If it is not the case, you can follow the instructions provided [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

---
**ℹ️Tip**

If you already have _Azure CLI_ installed you can execute:
```bash
az aks install-cli
```
---

#### Helm
To deploy AMC you need to have Helm installed on the machine you are using to deploy.

Follow the instructions for your platform provided [here](https://helm.sh/docs/intro/install/).

---
**ℹ️ Quick macOS install with `brew`**
```bash
brew install helm
```
---



#### Accessing your cluster
Before you proceed with deploying the *Helm chart* you need to have access to your Kubernetes cluster.

If you do not have _Azure CLI_ installed yet, follow the instructions [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest).
```bash
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```
Where `myResourceGroup` is the resource group where you have the cluster deployed, and `myAKSCluster` is the name of the cluster.

#### Ingress Controller
It is required to have an _Ingress Controller_ available on your cluster. AMC uses `Ingress` resources to configure access to the product.
You should contact your cluster administrator in order to get information about this.

If you do not have a provisioned _Ingress Controller_ you may can the instructions for [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/).
Refer to the section for easy deployment using Helm.


#### _PostgreSQL_ Server
You need to provide a _PostgreSQL_ Server with version >= 10.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to access it.

If you do not have one, and your deployment is on a standard certified Kubernetes cluster, there is an optional _Helm Chart_ dependency provided that deploys a third-party _PostgreSQL_ deployment alongside with AMC.

To enable this, before installing the _Helm Chart_, edit `values.yaml` and set `postgresqlInternalServer.enabled` to `true`.

#### Persistent Volume provisioner
For the deployment to function, the cluster must be able to provision a persistent volume that supports `ReadWriteMany` mode (`RWX`).
To support dynamic provisioning, a StorageClass that provisions volumes in `RWX` mode must be made available.

The easiest way to satisfy this on Azure AKS is to create an _Azure Files_ dynamic provisioner.
Follow the instructions [here](https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv).

---
**NOTE**

You only need to follow the instructions up to, and including, [Create a persistent volume claim](https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv). You do not need to follow the instructions on _"Use the persistent volume"_.

---

#### Domain and TLS Certificate
Due to security concerns, it is required that the server is reachable via HTTPS. 
You need to provide the appropriate Secret with the TLS certificate for a valid domain.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to configure it.

If you already have a correctly configured public DNS zone that points to the cluster, you can install _cert-manager_, to issue and manage certificates automatically, provided by `Let's Encrypt`, following the instructions [here](https://cert-manager.readthedocs.io/en/latest/tutorials/acme/quick-start/index.html).

#### SMTP Server
This product requires an SMTP server that can send invitation emails, as well as password recovery emails. It is not possible to add users to the platform without a correctly configured SMTP server.

#### Docker images
The docker images (`ibm/mobile-capture-api` and `ibm/mobile-capture-static-file-server`) need to be present on a docker image registry accessible by the cluster.
If the registry is private, you can configure the pull secrets required to pull the image from the registry.
Contact your cluster administration for this information.

#### Secrets
Before the initial deployment, you can use the following operations:
Create image pull secret for the registry where the docker images are located at. This is dependent on where the images are stored, your cluster administrator should be able to assist you on this. Keep a note of the image pull secret name.

##### Create secret key base secret
```bash
kubectl create secret generic mobilecapture --from-literal=secret-key-base=$(openssl rand -hex 64)
```
##### Create SMTP password secret
```bash
$ kubectl create secret generic mobilecapture-smtp --from-literal=smtp-password=<smtp server password>
```
##### Create seed admin user password
```bash
$ kubectl create secret generic mobilecapture-admin-seed --from-literal=password=<admin seed password>
```
##### Create PostgreSQL database secret
```bash
$ kubectl create secret generic mobilecapture-postgresql --from-literal=postgresql-password="$(openssl rand -hex 32)”
```
Replace `"$(openssl rand -base64 32)”` with your own password if you are connecting to an existing PostgreSQL server.

#### `values.yaml` Configuration
In the directory of the AMC _Helm Chart_ you can find a file named `values.yaml`.
Use your editor of preference to edit and update the file so that its contents remain according to the following template:
```yaml
postgresql:
  existingSecret: mobilecapture-postgresql
images:
  pullSecrets:
  - <image pull secret name>
  rails:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-api
  react:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-static-file-server
existingSecret: mobilecapture
authentication: internal
seed:
  adminUser:
    username: 'admin@ibm.com'
    passwordSecretName: mobilecapture-admin-seed
hostname: <hostname for the deployed server>
smtpConfiguration:
  senderName: IBM Mobile Capture
  senderEmail: donotreply@<hostname for the deployed server>
  domain: <smtp email domain>
  address: <smtp address>
  port: <smtp port>
  username: <smtp username>
  passwordSecret: mobilecapture-smtp
tls:
  host: <hostname for the deployed server>
  secret: <tls secret name>
```

If you setup _Azure Files_ dynamic provisioner set:
```yaml
uploadPersistence:
  storageClass: azurefile
```

---
**NOTE**

Indentation on YAML files is important, make sure you use the correct indentation spacing.

---

---
**NOTE**

If you decided to deploy an internal PostgreSQL server automatically, set on `values.yaml`:
```yaml
postgresqlInternalServer:
  enabled: true
```

---

#### LDAP Configuration
Under `ldap` set the values for:
- `host`: LDAP server addresss.
- `port`: LDAP server port.
- `treebase`: The treebase where the user search can be done.
- `emailAttribute`: The LDAP attribute where the user's email is stored.
- `adminsGroup`: The LDAP group users, who must be members having access to workspace admin role on Mobile Capture.
- `usersGroup`: The LDAP group users, who must be members having access to Mobile Capture.
- `enableLDAPS`: Enable LDAP over SSL
- `ldapsCertificateLocation`: 
If LDAPS is active, this field should contain the LDAP over SSL server’s public certificate in PEM format (.pem), the path is relative to the helm chart’s root directory. It validates the server's authenticity against this certificate. It can be a CA root certificate, or a self-signed certificate. If you save the public certificate named as `ldaps_cert.pem` and copy it to the same directory as the helm chart, the content of `ldapsCertificateLocation`  should be `ldaps_cert.pem`.


```yaml
authentication: internal
ldap:
  host: <ldap host>
  port: <ldap port>
  treebse: <treebase>
  emailAttribute: <email attribute>
  adminGroup: <admin group>
  usersGroup: <users group>
  enableLDAPS: <true/false>
  ldapsCertificateLocation: 'certificate.pem'
```

#### External OpenID Connect Configuration

Read instructions [here](AMC_OIDC_Configuration_Guide.md) on how to configure an external OIDC Provider as an authentication provider.

#### Virtual Root Configuration
If traffic into the cluster is rooted by an external proxy employing a URL rewrite strategy, effectively creating a virtual root, you need to define the virtual root's path on the configuration.
This option does not serve mobile capture under a virtual root, but it is needed in order to generate links correctly for inviting new users, reset password, upload endpoint URLs, etc.
This value is used in conjunction with the value of hostname.
The value must begin with a forward slash, and terminate without a trailing slash.
For example:
    - You employ a URL rewrite strategy to serve mobile capture at https://www.example.com/mobilecapture
    - Set the value of `hostname` to `www.example.com`
    - Set the value of `virtualRoot` to `/mobilecapture`

```yaml
hostaname: www.example.com
virtualRoot: /mobilecapture
```


### 3.4. Amazon EKS

#### kubectl
To connect with your Kubernetes cluster, you need to have _kubectl CLI_ installed on the machine you are using to deploy. If you are already managing a Kubernetes cluster, this should already be installed on your machine.

If it is not the case, you can follow the instructions provided [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

---
**ℹ️ Quick macOS install with `brew`**
```bash
brew install kubernetes-cli
```
---

#### Helm
To deploy AMC you need to have Helm installed on the machine you are using to deploy.

Follow the instructions for your platform provided [here](https://helm.sh/docs/intro/install/).

---
**ℹ️ Quick macOS install with `brew`**
```bash
brew install helm
```
---

#### Accessing your cluster
Before you proceed with deploying the *Helm chart* you need to have access to your Kubernetes cluster.

If you do not have _AWS CLI version 2_ installed yet, follow the instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
```bash
aws eks --region region-code update-kubeconfig --name cluster_name
```
Where `region-code` is the region code where you have the cluster deployed, and `cluster_name` is the name of the cluster.

For more information, refer to Amazon's User Guide [Create a kubeconfig for Amazon EKS](https://docs.aws.amazon.com/eks/latest/userguide/create-kubeconfig.html).

#### Ingress Controller
It is required to have an _Ingress Controller_ available on your cluster. AMC uses `Ingress` resources to configure access to the product.
You should contact your cluster administrator to get information about this.

If you do not have a provisioned _Ingress Controller_ you may follow the instructions for [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/deploy/).
Refer the section for easy deployment using Helm.

#### _PostgreSQL_ Server
You need to provide a _PostgreSQL_ Server with version >= 10.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to access it.

If you do not have one, and your deployment is on a standard certified Kubernetes cluster, there is an optional _Helm Chart_ dependency provided that deploys a third-party _PostgreSQL_ deployment alongside with AMC.

To enable this, before installing the _Helm Chart_, edit `values.yaml` and set `postgresqlInternalServer.enabled` to `true`.

#### Persistent Volume provisioner
For this deployment to function, the cluster must be able to provision a persistent volume that supports `ReadWriteMany` mode (`RWX`).
To support dynamic provisioning, a StorageClass that provisions volumes in `RWX` mode must be made available.

The easiest way to satisfy this is to integrate _Amazon EFS CSI driver_.
Follow the instruction [here](https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html).

After deploying the #AMC _Helm Chart_ you might need to set the correct ownership on the volume provisioned by Amazon EFS to user id `1001` and group id `0` (`1001:0`).

---
**NOTE**

To change ownership using Kubernetes:
1. Install [kubectl exec-as](https://github.com/jordanwilson230/kubectl-plugins/tree/krew#kubectl-exec-as)
1. List running _pods_ using `kubectl get pod`
1. Find pod named `<releasename>-mobile-capture-api-<pod identifier>`
1. `kubectl exec-as exec-as <releasename>-mobile-capture-api-<pod identifier>`
1. `bash-4.4$ chown 1001:0 /opt/app-root/src/public/uploads`
1. `bash-4.4$ exit`

---

#### Domain and TLS Certificate
Due to security concerns, it is required that the server is reachable via HTTPS. 
You need to provide the appropriate Secret with the TLS certificate for a valid domain.

Your cluster administrator should be able to provide you the additional information if this requirement is already satisfied and how to configure it.

If you already have a correctly configured public DNS zone that points to the cluster, you can install _cert-manager_, to issue and manage certificates automatically, provided by `Let's Encrypt`, following the instructions [here](https://cert-manager.readthedocs.io/en/latest/tutorials/acme/quick-start/index.html).

#### SMTP Server
This product requires an SMTP server that can send invitation emails, as well as password recovery emails. It is not possible to add users to the platform without a correctly configured SMTP Server.

#### Docker images
The docker images (`ibm/mobile-capture-api` and `ibm/mobile-capture-static-file-server`) need to be present on a docker image registry accessible by the cluster.
If the registry is private, you can configure the pull secrets required to pull the image from the registry.
Contact your cluster administration for this information.

#### Secrets
These operations only need to be used, before the initial deployment.

Create image pull secret for the registry where the docker images are located. This is dependent on where the images are stored, your cluster administrator should be able to assist you on this. Keep a note of the image pull secret name.

##### Create secret key base secret
```bash
kubectl create secret generic mobilecapture --from-literal=secret-key-base=$(openssl rand -hex 64)
```
##### Create SMTP password secret
```bash
$ kubectl create secret generic mobilecapture-smtp --from-literal=smtp-password=<smtp server password>
```
##### Create seed admin user password
```bash
$ kubectl create secret generic mobilecapture-admin-seed --from-literal=password=<admin seed password>
```
##### Create PostgreSQL database secret
```bash
$ kubectl create secret generic mobilecapture-postgresql --from-literal=postgresql-password="$(openssl rand -hex 32)”
```
Replace `"$(openssl rand -base64 32)”` with your own password if you are connecting to an existing PostgreSQL server.

#### `values.yaml` Configuration
In the directory of the AMC _Helm Chart_ you can find a file named `values.yaml`.
Use your editor of preference to edit the file so that its contents remain according to the following template:
```yaml
postgresql:
  existingSecret: mobilecapture-postgresql
images:
  pullSecrets:
  - <image pull secret name>
  rails:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-api
  react:
    registry: <registry where the docker image is>
    repository: ibm/mobile-capture-static-file-server
existingSecret: mobilecapture
authentication: internal
seed:
  adminUser:
    username: 'admin@ibm.com'
    passwordSecretName: mobilecapture-admin-seed
hostname: <hostname for the deployed server>
smtpConfiguration:
  senderName: IBM Mobile Capture
  senderEmail: donotreply@<hostname for the deployed server>
  domain: <smtp email domain>
  address: <smtp address>
  port: <smtp port>
  username: <smtp username>
  passwordSecret: mobilecapture-smtp
tls:
  host: <hostname for the deployed server>
  secret: <tls secret name>
```

If you setup _Amazon EFS CSI driver_  up:
```yaml
uploadPersistence:
  existingClaim: efs-claim
  storageClass: efs-sc
```

---
**NOTE**

Indentation on YAML files is important, make sure you use the correct indentation spacing.

---

---
**NOTE**

If you decided to  deploy an internal PostgreSQL server automatically, set on `values.yaml`:
```yaml
postgresqlInternalServer:
  enabled: true
```

---

#### LDAP Configuration
Under `ldap` set the values for:
- `host`: LDAP server address.
- `port`: LDAP server port.
- `treebase`: The treebase where the user search can be done.
- `emailAttribute`: The LDAP attribute where the user's email is stored.
- `adminsGroup`: The LDAP group users, who must be members having access to workspace admin role on Mobile Capture.
- `usersGroup`: The LDAP group users, who must be members having access to Mobile Capture.
- `enableLDAPS`: Enable LDAP over SSL
- `ldapsCertificateLocation`: 
If LDAPS is active, this field should contain the LDAP over SSL server’s public certificate in PEM format (.pem), the path is relative to the helm chart’s root directory. It validates the server's authenticity against this certificate. It can be a CA root certificate, or a self-signed certificate. If you save the public certificate named as `ldaps_cert.pem` and copy it to the same directory as the helm chart, the content of `ldapsCertificateLocation`  should be `ldaps_cert.pem`.

```yaml
authentication: internal
ldap:
  host: <ldap host>
  port: <ldap port>
  treebse: <treebase>
  emailAttribute: <email attribute>
  adminGroup: <admin group>
  usersGroup: <users group>
  enableLDAPS: <true/false>
  ldapsCertificateLocation: 'certificate.pem'
```

#### External OpenID Connect Configuration

Read instructions [here](AMC_OIDC_Configuration_Guide.md) on how to configure an external OIDC Provider as an authentication provider.

#### Virtual Root Configuration
If traffic into the cluster is rooted by an external proxy employing a URL rewrite strategy, effectively creating a virtual root, you need to define the virtual root's path on the configuration.
This option cannot serve mobile capture under a virtual root, but is needed to generate links correctly for inviting new users, reset password, upload endpoint URLs, etc.
This value is used in conjunction with the value of hostname.
The value must begin with a forward slash, and terminate without a trailing slash.
For example: 
    - You employ a URL rewrite strategy to serve mobile capture at https://www.example.com/mobilecapture
    - Set the value of `hostname` to `www.example.com`
    - Set the value of `virtualRoot` to `/mobilecapture`

```yaml
hostaname: www.example.com
virtualRoot: /mobilecapture
```

## 4. Deploying

Before deploying, ensure that you have followed all [configuration instructions](#configuration).

### 4.1 Uploading your images

Your images need to be uploaded to a registry accessible by your cluster.

#### Gather information

To upload to a docker image registry service gather the following information:
- Registry name (example: `quay.io`)
- Repository names:
  -  For `mobile-capture-api` image (example: `company/mobile-capture-api`)
  - For `mobile-capture-static-file-server` image (example: `company/mobile-capture-static-file-server`)

You also need the archive containing the docker images (example: `docker-images-269.tar.gz`).

#### Load images locally

Once you have this information, execute on the terminal:
```bash
docker load -i docker-images-269.tar.gz
```
Replacing `docker-images-269.tar.gz` with the appropriate image archive name.


The output will have the format of 
```bash
Loaded image: ibm/mobile-capture-api:<api-image-tag>
Loaded image: ibm/mobile-capture-static-file-server:<static-file-server-image-tag>
```
and will be similar to:
```bash
Loaded image: ibm/mobile-capture-api:9d9954a1c13445b8177a47db9cb77181c8040f0d
Loaded image: ibm/mobile-capture-static-file-server:b48314f7eeb17b308dbb5e93002cd12799a54208
```

#### Tag images locally

With this information, tag, for each image, with the new registry and repository names as follows:
```bash
docker tag ibm/mobile-capture-api:<api-image-tag> <destination registry name>/<api-image-destination-repository>:<api-image-tag>

docker tag ibm/mobile-capture-static-file-server:<api-image-tag> <destination registry name>/<static-file-server-image-destination-repository>:<static-file-server-image-tag>
```

Example:
```bash
docker tag ibm/mobile-capture-api:9d9954a1c13445b8177a47db9cb77181c8040f0d quay.io/company/mobile-capture-api:9d9954a1c13445b8177a47db9cb77181c8040f0d

docker tag ibm/mobile-capture-static-file-server:b48314f7eeb17b308dbb5e93002cd12799a54208 quay.io/company/mobile-capture-static-file-server:b48314f7eeb17b308dbb5e93002cd12799a54208
```

#### Push images to registry

Make sure you have logged in to your registry with the following command:
```bash
docker login <registry name>
```
Example:
```bash
docker login quay.io
```

Push the images to your registry:
```bash
docker push <destination registry name>/<api-image-destination-repository>:<api-image-tag>
docker push <destination registry name>/<static-file-server-image-destination-repository>:<static-file-server-image-tag>
```
Example:
```bash
docker push quay.io/company/mobile-capture-api:9d9954a1c13445b8177a47db9cb77181c8040f0d
docker push quay.io/company/mobile-capture-static-file-server:b48314f7eeb17b308dbb5e93002cd12799a54208
```


### 4.2. New Deployment
For new deployments execute on the terminal:
```bash
helm install mobilecapture .
```

---
**NOTE**

If you have installed _NGINX_ as your ingress controller you need to patch the ingress resource to support bigger file upload size.
```bash
kubectl patch ingress/mobilecapture-mobile-capture -p '{"metadata":{"annotations":{"nginx.ingress.kubernetes.io/proxy-body-size":"50m"}}}'
```

---

---
**NOTE**

If you have installed `cert-manager` as you also need to patch the ingress resource to have it configure the TLS certificate automatically.
```bash
$ kubectl patch ingress/mobilecapture-mobile-capture -p '{"metadata":{"annotations":{"cert-manager.io/issuer":"letsencrypt-prod"}}}'

```

---

### 4.3. Upgrading an existing deployment

#### Backup your data

##### Database

Mobile Capture stores its configuration and metadata on a PostgreSQL database.

Make sure you have an up-to-date backup of your database.

##### Persistent Data Volumes

Mobile Capture stores files on one Persistent Volume. Depending on how you configured your deployment you need to make sure you have an up-to-date backup of the volume.

On a default deployment, the Persistent Volume Claim's name for the corresponding Persistent Volume is as follows:
- `<helm release name>-mobile-capture-upload`

Where `<helm release name>` is replaced by the name you have given to the helm release when deploying.


#### Tools

##### Helm version

Make sure you have installed on your machine *Helm v3*. 

Previous releases could be installed using *Helm v2*, while the current release **requires** *Helm v3*.

#### Upgrade from version 3.1.1

##### Helm version upgrade preparation

If your last deployment was performed using *Helm v2* and you installed the optional PostgreSQL chart dependency, you need to perform a simple update to your release before proceeding.

To verify this, execute on a terminal:
```bash
kubectl get StatefulSet <helm release name>-postgresql -o jsonpath='{.metadata.labels.app\.kubernetes\.io/managed-by}'
```

Where `<helm release name>` is replaced by the name you have given to the helm release when deploying.

If `Helm` is printed as a result of the execution, then you may skip this step and continue with the upgrade instructions.

If nothing or another values is printed, execute the following commands on a terminal:
```bash
RESOURCE_TYPE=StatefulSet
RELEASE_NAME=<helm release name>
RESOURCE_NAME=$RELEASE_NAME-postgresql
NAMESPACE=default
kubectl label $RESOURCE_TYPE $RESOURCE_NAME app.kubernetes.io/managed-by=Helm
kubectl annotate $RESOURCE_TYPE $RESOURCE_NAME meta.helm.sh/release-name=$RELEASE_NAME
kubectl annotate $RESOURCE_TYPE $RESOURCE_NAME meta.helm.sh/release-namespace=$NAMESPACE
```
Where `<helm release name>` is replaced by the name you have given to the helm release when deploying.

You only need to perform this update once when moving to *Helm v3*.

##### Values.yaml configuration

Use your previously configured `Values.yaml` file, configured for deployment of version 3.1.1, as a reference for editing the `Values.yaml` file contained in the new helm chart.

Make sure you leave the image configuration entries intact on the new version (configurations under the `images` key).

Added configurations:
- _Segment Analytics configuration_

Updated configurations:
- _LDAP configuration_

##### Upgrade Helm Chart

Once you have backed up your data and validated your configurations you can upgrade your Helm chart deployment.

To do this, execute on a terminal:
```bash
helm upgrade <helm release name> .
```

##  5. Possible Errors

### 5.1. Upgrading an existing deployment
If you find an error, when upgrading, as such:

> Error: UPGRADE FAILED: rendered manifests contain a resource that already exists. Unable to continue with update: StatefulSet "mc-ibm-demo-staging-postgresql" in namespace "default" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "mc-ibm-demo-staging"; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "default"

This means you have previously installed the chart using Helm v2, and migrated to Helm v3. You have to fix this only once, and the following upgrades should work as expected, with no errors being presented.

To correct this, proceed with the following on the terminal:
```bash
RESOURCE_TYPE=StatefulSet
RESOURCE_NAME=mc-ibm-demo-staging-postgresql
RELEASE_NAME=mc-ibm-demo-staging
NAMESPACE=default
kubectl label $RESOURCE_TYPE $RESOURCE_NAME app.kubernetes.io/managed-by=Helm
kubectl annotate $RESOURCE_TYPE $RESOURCE_NAME meta.helm.sh/release-name=$RELEASE_NAME
kubectl annotate $RESOURCE_TYPE $RESOURCE_NAME meta.helm.sh/release-namespace=$NAMESPACE
```
Where you set the RESOURCE_TYPE, RESOURCE_NAME, RELEASE_NAME and NAMESPACE variables according to the error presented by Helm.
