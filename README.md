# Watsonx Orchestrate 5.1 - On-Premises Deployment Guide

## Overview
Watsonx Orchestrate is an AI-powered assistant that streamlines repetitive tasks and enhances productivity through generative AI and automation. This guide provides step-by-step instructions to deploy Watsonx Orchestrate 5.1 on an on-premises data platform.

## Deployment Options
Watsonx Orchestrate can be deployed using:
- **Managed SaaS**: IBM Cloud or AWS
- **On-Premises**: IBM Cloud Pak for Data & IBM Software Hub
## Deployment Architecture
![image](https://github.com/user-attachments/assets/485ff9e5-24e4-4163-aa46-aeffa27382bd)

## Installation Steps:
![image](https://github.com/user-attachments/assets/51225ef2-dd56-4f91-9f1d-1b43b6abc681)


Before starting the deployment, ensure the following:
Reserve OpenShift on TechZone for installation training
- **Infrastructure**: OpenShift 4.16 Cluster with 5 worker nodes, Worker node ‘flavor’ (vCPUs x 
Memory): 32x128.300gb
- **Storage**: ODF (local disks) - 2TB
- **Access to IBM TechZone**: [TechZone Reservation](https://techzone.ibm.com/my/reservations/create)

## Prepare bastion workstation
- **use the information in techzone provisioned instance to ssh to bastion workstation**
- after successful login, switch user to become a root.  run: sudo su
- ```mkdir cpd```
- ```cd cpd```

![image](https://github.com/user-attachments/assets/b0a154ea-06d0-4298-8cea-37bf57e2f30c)

## Install Podman
Using the bastion server SSH session as root, install the Podman container runtime engine using the 
Yum package manager: \
```yum install -y podman```

![image](https://github.com/user-attachments/assets/9bda9203-26b0-49cb-984e-2b6a276a0e0b)

## Install Software Hub ( CPD CLI)
- as root, run the following command to get the cpd-cli package
```wget https://github.com/IBM/cpd-cli/releases/download/v14.1.0/cpd-cli-linux-EE-14.1.0.tgz```
- untar the file using the following command
- tar -xvf `<file name>`
- cd cpd-cli-linux-EE-14.1.0-1189
- type `pwd` to get the path
- ```/home/itzuser/cpd/cpd-cli-linux-EE-14.1.0-1189```
## Modify the root account's .bashrc file to include the CPD CLI in the path
Using the same bastion server SSH session as root:

1. Change directory into the cpd-cli directory starting by typing cd cpd-cli<tab> and tab complete for the full directory name.
2. Show the path to the current directory by typing: pwd
3. Edit the .bashrc file: vi ~/.bashrc
4. Go to the bottom of the file, and add a few empty lines at the end.
5. Prepend the cli folder to the path: ```export PATH=/home/itzuser/cpd/cpd-cli-linux-EE-14.1.0-1189:$PATH```
6. Exit the vi editor by typing `:wq` to write the changes and quit
7. Back out of the current folder by typing cd ..
8. Load the path change for this current bash session by sourcing the .bashrc file: source ~/.bashrc
9. Run ```Run cpd-cli manage restart-container```
![image](https://github.com/user-attachments/assets/2fd865ce-1122-47e6-9ed3-83f297d9fe99)
10. Verify that the CPD CLI can be found by typing `cpd-cli version`

## Obtain your IBM Entitlement Key
1. Log in to [Container software library](https://myibm.ibm.com/products-services/containerlibrary) on My IBM with your IBMid and password.
2. Go to the Entitlement keys tab.
3. If you already have a key, click Copy to copy the entitlement key to the clipboard.
4. If you do not have an Entitlement key, Click Add new key to add a key.
5. Copy the entitlement Key
![image](https://github.com/user-attachments/assets/ee132898-7a72-4121-b3cc-b2f53e648339)

## Install OpenShift certification manager operator
Note: IBM cert manager is not needed for WxO 5.1 / CPD5.1 install.
1. Login to openShift Console using ```kubeadmin /password```
2. Click on OperatorHub
3. Search for RedHat cert manager.
4. Choose version 1.13.0 pr 1.14.1
5. Choose the option to Install in a specific namespace on cluster.
6. Click Install
   ![image](https://github.com/user-attachments/assets/6e549343-80a2-4df7-8eeb-a465e5ea6a76)


### Validate deployment 
1. Using ssh terminal run `oc project cert-manager-operator` to connect to the project/namespace.
2. Then run `oc get subscription -n cert-manager-operator`
   ![image](https://github.com/user-attachments/assets/a393632b-6d98-442d-9823-71c2a6a1ad85)

3. Then run `oc get csv -n cert-manager-operator`
   ![image](https://github.com/user-attachments/assets/7c4c12b0-cdc1-47fa-a887-61ff3cd13745)
4. Then run `oc get pods -n cert-manager`

![image](https://github.com/user-attachments/assets/9aa51d97-f955-4fc5-9690-b5bf77d4dd48)


 ## Setting up installation Environment variables cpd_vars.sh
 The commands for installing various components on the OCP cluster from the Baston use variables with
the format ${VARIABLE_NAME}.
While still logged in to bastion workstation as root and in cpd folder
1. run `wget https://raw.githubusercontent.com/CloudPak-Outcomes/OutcomesProjects/refs/heads/main/L4assets/Installation/cpd_vars.sh`.
2. Use vi to edit the cpd_vars.sh file.
3. Scroll down to the "# Cluster" section
   Change the OCP_URL value to the "API URL" value from the Techzone Reservation Details 
   * Un-comment the OCP_USERNAME and set it's value to kubeadmin 
   * Un-comment the OCP_PASSWORD and set its value to the kubeadmin's password from the Techzone Reservation Details 
   * Un-comment the first LOGIN_ARGUMENTS line, it will be the one with the --username and --password arguments 
4. Scroll down to the "# Projects" section 
   * Change the PROJECT_CERT_MANAGER value to ibm-cert-manager ( This is not used since we installed OpenShift Cert Manager)
   * Change the PROJECT_LICENSE_SERVICE value to ibm-licensing 
   * Change the PROJECT_SCHEDULING_SERVICE value to ibm-cpd-scheduler 
   * Change the PROJECT_CPD_INST_OPERATORS value to cpd-operators 
   * Change the PROJECT_CPD_INST_OPERANDS value to cpd
5. Scroll down to the "# IBM Entitled Registry" section
   *Change the IBM_ENTITLEMENT_KEY value to your entitlement key
   
6. Scroll down to the "# IBM Software Hub" section
   * Un-comment and set export VERSION=5.1.0
7. Scroll down to the "# watsonx orchestrate" section
    * Un-comment and set export PROJECT_IBM_APP_CONNECT=app-connect
    * Un-comment and set export AC_CASE_VERSION=12.5.0
    * Un-comment and set export AC_CHANNEL_VERSION=v12.5
8. Save the file using `esc`, then `:wq`.
9. Run the command `bash ./cpd_vars.sh` in the terminal to check for syntax errors
10. Modify the file permissions for the file using the command `chmod 700 cpd_vars.sh`
11. source the file using `source ./cpd_vars.sh`.
12. Test by running ${OC_LOGIN} and ${CPDM_OC_LOGIN}

![image](https://github.com/user-attachments/assets/095e4032-42d9-42aa-8f36-a6d2e279ae4d)

## Updating the global image pull secret for IBM Software Hub
We will be using the IBM Entitled Registry. Run the command below to modify the image pull secret inside of OpenShift 
so that it is set to our entitlement key. This command will reboot nodes in the cluster.
1. ```cpd-cli manage add-icr-cred-to-global-pull-secret \ --entitled_registry_key=${IBM_ENTITLEMENT_KEY}```
2. ![image](https://github.com/user-attachments/assets/c4d1592f-c704-4f8e-940f-d0d6f3f97e4b)
3. Watch to make sure the nodes are updated by running `watch oc get mcp` and looking at Master and Workers.
4. Wait until UPDATING shows FALSE and the UPDATED shows True. Once they do, use ctrl-c in the terminal to stop watching
5. ![image](https://github.com/user-attachments/assets/6c0a9a8a-c2a4-431c-8a00-10836e5f3b26)
5.![image](https://github.com/user-attachments/assets/07d574ae-e7ef-45df-b1f3-9bbcf52d0764)
6. Run this command `cpd-cli manage oc get nodes`
![image](https://github.com/user-attachments/assets/f8ec7ea9-1df8-4990-b762-7ce044f7bb3a)

## Manually creating projects (namespaces) for the shared cluster components
We will create the namespaces for the IBM licensing service, and the IBM Scheduler service. In the bastion terminal window, run the following commands:

* ``oc new-project ${PROJECT_LICENSE_SERVICE}``
* ``oc new-project ${PROJECT_SCHEDULING_SERVICE}``
* ``oc new-project ${PROJECT_CPD_INST_OPERATORS}``
* ``oc new-project ${PROJECT_CPD_INST_OPERANDS}``

## Install shared cluster components

1. Run the command `cpd-cli manage apply-cluster-components \ --release=${VERSION} \ --license_acceptance=true \ --licensing_ns=${PROJECT_LICENSE_SERVICE}`
   ![image](https://github.com/user-attachments/assets/b90d1116-ea92-4cfa-8a52-2991d0f0bef0)


3. Then run `cpd-cli manage apply-scheduler \ --release=${VERSION} \ --license_acceptance=true \ --scheduler_ns=${PROJECT_SCHEDULING_SERVICE}`
   ![image](https://github.com/user-attachments/assets/43a2a0a7-a79f-43df-af8c-c61141ca91fd)


## Confirm persistent storage
Run the following
* `oc get sc`
  
* ![image](https://github.com/user-attachments/assets/2af959f6-5899-4b4c-8258-96364578ef46)

* `oc get pvc -A`

![image](https://github.com/user-attachments/assets/2ac6a34e-cb14-4201-92ac-31a3b69e0509)

## Change required node settings
1. Confirm there is no kubeletconfig on the cluster. Run:
* `oc get kubeletconfig`
2. Since there is no record you can run this command
```sh
oc apply -f - << EOF
apiVersion: machineconfiguration.openshift.io/v1
kind: KubeletConfig
metadata:
  name: cpd-kubeletconfig
spec:
  kubeletConfig:
    podPidsLimit: 16384
  machineConfigPoolSelector:
    matchExpressions:
    - key: pools.operator.machineconfiguration.openshift.io/worker
      operator: Exists
EOF
```
![image](https://github.com/user-attachments/assets/a6dc8d33-149f-4c2c-99fc-325871e6fcf4)

## Install prerequisites requirements
### Install the Node Feature Discovery Operator from the OpenShift Console
1. Go to the Techzone Environment and click on the "Desktop URL" link to open the Openshift Console in a web browser, and log into the cluster using the kubeadmin credentials.
2. Click on "Operators" in the left side menu bar.
3. Click on "OperatorHub", and in the search field, type "node feature discovery"
4. From the results, Click on the "Node Feature Discovery", choosing Red Hat one, and then click "Install", and then "Install" again a the bottom of the page.
Wait until the installation completes.
### Install the Nvidia GPU Operator from the OpenShift Console
1. Click on "Operators" in the left side menu bar.
2. Click on "OperatorHub", and in the search field, type "NVIDIA GPU Operator"
3. From the results, Click on the "NVIDIA GPU Operator", then click "Install", and then "Install" again a the bottom of the page.
Wait until the installation completes.

### Install Redhat Openshift AI
1. From the bastion ssh session, create the namespace for the operator `oc new-project redhat-ods-operator`.
   ![image](https://github.com/user-attachments/assets/9b8d7188-75d8-4c3b-9539-87b2601dca57)

2. Create the operator group
``` sh
cat <<EOF |oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
name: rhods-operator
namespace: redhat-ods-operator
EOF
```
![image](https://github.com/user-attachments/assets/55bf9c58-4293-459b-93c7-1ca9608eab37)

3. Create the subscription
``` sh
cat <<EOF |oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
name: rhods-operator
namespace: redhat-ods-operator
spec:
name: rhods-operator
channel: stable-2.13
source: redhat-operators
sourceNamespace: openshift-marketplace
config:
    env:
        - name: "DISABLE_DSC_CONFIG"
EOF
```
![image](https://github.com/user-attachments/assets/524e16aa-2e8d-43a5-a4ca-3d52e72f512a)

4.You will now need to create a Red Hat OpenShift Dev Space CLI (DSC) object called default-dsci in
the redhat-ods-monitoring project. This is a feature of Red Hat OCP that allows users to work with
certificates.
``` sh
cat <<EOF |oc apply -f -
apiVersion: dscinitialization.opendatahub.io/v1
kind: DSCInitialization
metadata:
  name: default-dsci
spec:
  applicationsNamespace: redhat-ods-applications
  monitoring:
    managementState: Managed
    namespace: redhat-ods-monitoring
  serviceMesh:
    managementState: Removed
  trustedCABundle:
    managementState: Managed
    customCABundle: ""
EOF


```

![image](https://github.com/user-attachments/assets/15f2c5de-796a-4913-810d-30d01532d1d4)

Confirm the operator is running
5. run ` oc get pods -n redhat-ods-operator`
![image](https://github.com/user-attachments/assets/edcb03ac-3b52-49f2-9958-58cb4b86173f)
6. run `oc get dscinitialization`
![image](https://github.com/user-attachments/assets/cecd245c-08af-40a0-92dc-9d1fd0c1eaf3)

7. Create a data science cluster

``` sh
cat <<EOF |oc apply -f -
apiVersion: datasciencecluster.opendatahub.io/v1
kind: DataScienceCluster
metadata:
  name: default-dsc
spec:
  components:
    codeflare:
      managementState: Removed
    dashboard:
      managementState: Removed
    datasciencepipelines:
      managementState: Removed
    kserve:
      managementState: Managed
      defaultDeploymentMode: RawDeployment
      serving:
        managementState: Removed
        name: knative-serving
    kueue:
      managementState: Removed
    modelmeshserving:
      managementState: Removed
    ray:
      managementState: Removed
    trainingoperator:
      managementState: Managed
    trustyai:
      managementState: Removed
    workbenches:
      managementState: Removed
EOF

```
![image](https://github.com/user-attachments/assets/91573e60-7405-4bbe-b655-201c3d3db721)
8. wait for the data science cluster to be ready
9. run to check its status `oc get datasciencecluster default-dsc -o jsonpath='"{.status.phase}" {"\n"}'`

![image](https://github.com/user-attachments/assets/9bc9f678-12d9-4b6c-bb0a-1dadedb23452)
10. Run  `oc get pods -n redhat-ods-applications`
![image](https://github.com/user-attachments/assets/36a5e58b-91ee-41d2-83a8-eaa7d4695edf)

11.The last setup step for setting up Red Hat OpenShift AI involves editing the
inferenceservice-config file in the redhat-ods-applications project. You need to do this on the
OpenShift Console
*Login to openshift console. 
*Click to expand workload>Configmaps
*Select Redhat-ods-applications project
*Select inferenceservice-config
*Click on YAML tab.

![image](https://github.com/user-attachments/assets/87a35275-8995-4986-8a9e-dcbdc1079dfe)

![image](https://github.com/user-attachments/assets/276e144e-eb90-4ad4-bc23-209e8a8d12aa)

![image](https://github.com/user-attachments/assets/e20d98dc-91a3-45c6-84e9-b866a34a6912)

![image](https://github.com/user-attachments/assets/238c54b6-0c16-4eaf-8932-5d91c82de35f)

## Installing Red Hat OpenShift Serverless Knative Eventing
This is a key components and the following steps should be carefully applied. There is an issue with knative eventing installation in 5.1. latest version of redhat's openshift serverless operator changed the kafka-broker-dispatcher from a deployment object to a statefulset object.  The installation is waiting for a deployment to come up and bring up the pods, but the pods are now brought up by a statefulset , so this line will never complete.

## To Fix this
1. Run `podman ps`.
2. ![image](https://github.com/user-attachments/assets/8672088c-9064-4cb1-b3c5-5632ee605f04)
3. Record the COntainer ID.
4. Next run the following command replacing olm-utils container with podman container ID.
5. Example : `podman exec -it (olm-utils container) /bin/bash` - remove the parentheses.
6. ![image](https://github.com/user-attachments/assets/d130aa64-6e66-4df7-b974-d02211e44c99)

7. cd to bin
8. locate the deploy-knative-eventing, use vi to edit the file.
9. Delete this line `oc wait deployment -n knative-eventing kafka-broker-dispatcher --for condition=Available=True --timeout=60s`
10. save the file `:wq`
11. Now you can run `cpd-cli manage deploy-knative-eventing \ --release=${VERSION} \ --block_storage_class=${STG_CLASS_BLOCK}`

![image](https://github.com/user-attachments/assets/deb37d75-d115-4a22-88b8-0c260da7235b)


## Install App Connect 
1. Now we need to install app connect which is required by watsonx orchestrate
2. Run the following command while logged in to bastion workstation using ssh. You should be in cpd folder as root to get the app connect package. ( if you were logged off, re-run the source ./cpd_vars.sh, ${OC_LOGIN} and ${CPDM_OC_LOGIN}
3. Run `curl -sSLO https://github.com/IBM/cloud-pak/raw/master/repo/case/ibm-appconnect/${AC_CASE_VERSION}/ibm-appconnect-${AC_CASE_VERSION}.tgz`
4. untar the file using this command `tar -xf  ibm-appconnect-12.5.0.tgz`
5. ![image](https://github.com/user-attachments/assets/b1d53165-01f8-4f4b-977d-7ff89b1477df)
6.Create the app-connect project , run `oc new-project ${PROJECT_IBM_APP_CONNECT}`
7.![image](https://github.com/user-attachments/assets/0f7adf6a-8d3d-4dbc-b42c-5396fdc29856)
8. Create the catalog source for the App Connect operator.
``` sh
      oc patch \
--filename=ibm-appconnect/inventory/ibmAppconnect/files/op-olm/catalog_source.yaml \
--type=merge \
-o yaml \
--patch="{\"metadata\":{\"namespace\":\"${PROJECT_IBM_APP_CONNECT}\"}}" \
--dry-run=client \
| oc apply -n ${PROJECT_IBM_APP_CONNECT} -f -
 ```

*![image](https://github.com/user-attachments/assets/df273210-880d-4c65-90ab-7934e37a4cde)
 * Create the operator group for app connect.
``` sh
cat <<EOF | oc apply -f -
  apiVersion: operators.coreos.com/v1
  kind: OperatorGroup
  metadata:
    name: appconnect-og
    namespace: ${PROJECT_IBM_APP_CONNECT}
  spec:
    targetNamespaces:
    - ${PROJECT_IBM_APP_CONNECT}
    upgradeStrategy: Default
EOF

```

* ![image](https://github.com/user-attachments/assets/7a2025dd-0b59-4814-8072-a70a03ef2887)

*Create the subscription to App Connect Operator.
``` sh
cat <<EOF | oc apply -f -
  apiVersion: operators.coreos.com/v1alpha1
  kind: Subscription
  metadata:
    name: ibm-appconnect-operator
    namespace: ${PROJECT_IBM_APP_CONNECT}
  spec:
    channel: ${AC_CHANNEL_VERSION}
    config:
      env:
      - name: ACECC_ENABLE_PUBLIC_API
        value: "true"
    installPlanApproval: Automatic
    name: ibm-appconnect
    source: appconnect-operator-catalogsource
    sourceNamespace: ${PROJECT_IBM_APP_CONNECT}
EOF

```
* ![image](https://github.com/user-attachments/assets/a9eb820b-fe26-45f0-af69-a66e5dd5a9b8)

* You should Wait for the App Connect operator to be ready. Run the following command to checl that conditioned is met.
* To check this, run

``` sh
oc wait csv \
--namespace=${PROJECT_IBM_APP_CONNECT} \
--selector=operators.coreos.com/ibm-appconnect.${PROJECT_IBM_APP_CONNECT}='' \
--for='jsonpath={.status.phase}'=Succeeded
```

* ![image](https://github.com/user-attachments/assets/fc0af87e-216b-4c46-882e-d40e5baeca6f)
## Creating secrets for services that use Multicloud Object Gateway
Multicloud Object Gateway service ( noobaa service) is installed by default when ODF services is deployed.
To confirm the services installed run ` oc get secrets --namespace=openshift-storage | grep noobaa`.
![image](https://github.com/user-attachments/assets/51d71bf1-ebdd-477f-823b-029228494b76)
Ensure noobaa-admin and noobaa-s3-serving-cer exist.
* ensure you login `${CPDM_OC_LOGIN}`
* Run `export NOOBAA_ACCOUNT_CREDENTIALS_SECRET=noobaa-admin`
* Run `export NOOBAA_ACCOUNT_CERTIFICATE_SECRET=noobaa-s3-serving-cert`
![image](https://github.com/user-attachments/assets/8dfae9a6-b493-48a4-8186-6229fcb97ddb)
* ## For watsonx_Orchestrate
* First run
``` sh
cpd-cli manage setup-mcg \
--components=watson_assistant \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--noobaa_account_secret=${NOOBAA_ACCOUNT_CREDENTIALS_SECRET} \
--noobaa_cert_secret=${NOOBAA_ACCOUNT_CERTIFICATE_SECRET}
```


![image](https://github.com/user-attachments/assets/696d3a84-d9a9-4353-a652-0faf6b5a1393)
* Confirm the secrets have been created successfully by running
``` sh
oc get secrets --namespace=${PROJECT_CPD_INST_OPERANDS} \
noobaa-account-watson-assistant \
noobaa-cert-watson-assistant \
noobaa-uri-watson-assistant
```

![image](https://github.com/user-attachments/assets/45707e86-4d33-444d-a56d-09729b95177b)


* Now run this
``` sh
cpd-cli manage setup-mcg \
--components=watsonx_orchestrate \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--noobaa_account_secret=${NOOBAA_ACCOUNT_CREDENTIALS_SECRET} \
--noobaa_cert_secret=${NOOBAA_ACCOUNT_CERTIFICATE_SECRET}
```


![image](https://github.com/user-attachments/assets/d7c731d5-5569-4a7e-b2de-04b1cad72547)
* Confirm that the secrets are created by running
``` sh
oc get secrets --namespace=${PROJECT_CPD_INST_OPERANDS} \
noobaa-account-watsonx-orchestrate \
noobaa-cert-watsonx-orchestrate \
noobaa-uri-watsonx-orchestrate
```
![image](https://github.com/user-attachments/assets/e21547bb-dd64-42d7-a836-1d71bd75614e)
## Apply the required permissions
1. Run `${CPDM_OC_LOGIN}`
2. We used instances without tethered projects

``` sh
cpd-cli manage authorize-instance-topology \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```
![image](https://github.com/user-attachments/assets/3be69fd4-e252-4cb8-bb2b-349b569ac436)


## Install IBM Software Hub
1. run `${CPDM_OC_LOGIN}`
2. For Red Hat OpenShift Data Foundation Storage run ( Note: set the storage test to false since this is non-prod)

``` sh
cpd-cli manage setup-instance \
--release=${VERSION} \
--license_acceptance=true \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--run_storage_tests=false

```

3. The above command takes around 45 min and completes around try 63
4.![image](https://github.com/user-attachments/assets/f2592bcd-e603-4ba8-b1c6-7e74565204ef)
5.![image](https://github.com/user-attachments/assets/3cbf09cc-0b64-4c37-ba06-0117cae4bf8c)
6. Let's check the install is successful.
7. Run `cpd-cli manage get-cr-status \ --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}`
![image](https://github.com/user-attachments/assets/1b945b60-99eb-41a1-a78d-5ba41d3ac1f8)

8. Now you get the CPD credential to login to CPD console.
9. Run `cpd-cli manage get-cpd-instance-details \ --cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \ --get_admin_initial_credentials=true`

![image](https://github.com/user-attachments/assets/be081637-ac5c-48cc-9ca9-2b92e2420085)

Use the Credential details of your own setup to login
![image](https://github.com/user-attachments/assets/848a4cdc-c8dd-429a-839c-cba82b649b90)
![image](https://github.com/user-attachments/assets/49078b3b-609d-4ee6-993f-14f3086698d3)


## Apply Entitlements
Even though you are working in a non-production environment, you must provide information about the
licenses that you are using. If you skip this step, you will be unable to install any solutions or services.
We are installing CP4D Enterprise and watsonx orchestrate non production.
1. Run this for CP4D
``` sh
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=cpd-enterprise \
--production=false
```
2. Run this for watsonx orchestrate
```sh
cpd-cli manage apply-entitlement \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--entitlement=watsonx-orchestrate \
--production=false

```


## Install cpfs and cpd_platform
Now we are ready to install the platform and services. You will run a batch installation to install
multiple (though not all) components simultaneously. Batch installations support parallel installation of
some components

![image](https://github.com/user-attachments/assets/1c7d1ba6-208d-4613-8a56-dd6cf784b226)

1. Run `${CPDM_OC_LOGIN}`
2. Then run
``` sh
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=${COMPONENTS}
```

![image](https://github.com/user-attachments/assets/d2688fce-fe0a-42fc-85c5-650c5adc0fec)

3. Now run this command
``` sh
cpd-cli manage apply-cr \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=${COMPONENTS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--license_acceptance=true \

```
![image](https://github.com/user-attachments/assets/975f131f-9c3d-494b-b92e-9df566a08702)
![image](https://github.com/user-attachments/assets/5c79683e-7aa4-4853-8ff1-9d14d6ef59e4)


4. Confirm the status , run

``` sh
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS}
```
![image](https://github.com/user-attachments/assets/9c537e56-f484-46dc-b4da-952e78e5eadc)


5. Run this to check the list of subscriptions
``` sh
oc get subscriptions.operators.coreos.com --all-namespaces | grep cpd-operators

```
![image](https://github.com/user-attachments/assets/38c3317b-1acc-4251-9f59-0da1cf2a38f9)

## Install watsonx orchestrate
### install wxo pre-requisites
1. run `${CPDM_OC_LOGIN}`
2. Create an instance of app connect

``` sh
cpd-cli manage setup-appconnect \
--appconnect_ns=${PROJECT_IBM_APP_CONNECT} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--release=${VERSION} \
--components=watsonx_orchestrate \
--file_storage_class=${STG_CLASS_FILE}
```

![image](https://github.com/user-attachments/assets/b03f04f6-c239-4eed-8a6f-c9921a8848d0)

3. create the required OLM objects for watsonx Orchestrate in the operators project for the instance
``` sh
cpd-cli manage apply-olm \
--release=${VERSION} \
--cpd_operator_ns=${PROJECT_CPD_INST_OPERATORS} \
--components=watsonx_orchestrate
```
![image](https://github.com/user-attachments/assets/c268c65a-65c0-449a-8892-8a46e6de748c)
4. Create the custom resource for watsonx Orchestrate ( using OpenShift Data Foundation option)
``` sh
cpd-cli manage apply-cr \
--components=watsonx_orchestrate \
--release=${VERSION} \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--block_storage_class=${STG_CLASS_BLOCK} \
--file_storage_class=${STG_CLASS_FILE} \
--license_acceptance=true
```

![image](https://github.com/user-attachments/assets/7d0c9452-e4fa-49ed-9a80-42f40214f7c6)

![image](https://github.com/user-attachments/assets/03f9cab4-886c-4088-9a79-e2444d98eb8c)

5. Confirm the status of the custom resource by running
``` sh
cpd-cli manage get-cr-status \
--cpd_instance_ns=${PROJECT_CPD_INST_OPERANDS} \
--components=watsonx_orchestrate
```
![image](https://github.com/user-attachments/assets/c98cdcd0-b11f-4bc9-b57c-601f8750e0dc)

6. Login to CPD console using the credentials obtained in the previous steps.
   ![image](https://github.com/user-attachments/assets/8633ac95-eff3-4333-9748-b023758625d6)
You should see the services in green
7. Run `oc get wo`
![image](https://github.com/user-attachments/assets/381fe32a-04f9-4007-9373-bc3036cc4a1c)
There should be 57 components deployed in total ( for WxO 5.1)

## Creating a service instance for watsonx Orchestrate
There two main methods to create the Wxo Instance:

1. Creating a service instance for watsonx Orchestrate from the web client. In 5.1 release, there is an issue when creating the instance using the Web Client. 

2. Creating a service instance for watsonx Orchestrate with the cpd-cli service-instance create command. I used this method 

### Creating a service instance for watsonx Orchestrate with the cpd-cli
1. Generate the API key that you need for user authentication by going to your Profile and settings page in the web client and clicking Generate API key. Copy the API Key
2. Run these commands in the terminal 
* `export API_KEY=<api-key>`
* `export CPD_USERNAME=<user-name>`  use cpadmin for the user name.
* `export LOCAL_USER=<local-user>`   use any local name. I used wxolocal
* `export CPD_PROFILE_NAME=<cpd-profile-name>`  any arbitrary name I used wxcpro
* `oc get route cpd \ --namespace=${PROJECT_CPD_INST_OPERANDS}` 
* `export CPD_PROFILE_URL=<cpd-url>` ensure to add `https://` to the URL

3. Run
```sh
cpd-cli config users set ${LOCAL_USER} \ --username ${CPD_USERNAME} \ --apikey ${API_KEY}
```
4. Run
``` sh
cpd-cli config profiles set ${CPD_PROFILE_NAME} \ --user ${LOCAL_USER} \ --url ${CPD_PROFILE_URL}
```
### Creating WxO service instance
1. run `export INSTANCE_NAME=<display-name>`     I used wxozak
2. run `export INSTANCE_DESCRIPTION="<description>“` add a description in""
3. Create the orchestrate-instance.json file using this command
``` sh
cat << EOF > ./orchestrate-instance.json { "addon_type": "orchestrate", "display_name": "${INSTANCE_NAME}", "namespace": "${PROJECT_CPD_INST_OPERANDS}", "addon_version": "${VERSION}", "create_arguments": { "description": "${INSTANCE_DESCRIPTION}" } 
} EOF
```
4. Set the PAYLOAD_FILE environment variable to the fully qualified name of the JSON payload file on your workstation
``` sh
export PAYLOAD_FILE=<fully-qualified-JSON-file-name>
```
I used  /home/itzuser/cpd/orchestrate-instance.json
5. Create the instance by running 
``` sh
cpd-cli service-instance create \ --profile=${CPD_PROFILE_NAME} \ --from-source=${PAYLOAD_FILE}

```
6. Login to CPD console , click on the dots in the right up corner
7. Select IBM Cloud Pak for Data
8. On the left pane select instances. You should watsonx orchestrate instance you just created
   ![image](https://github.com/user-attachments/assets/5a47e9c3-bdea-4cf5-8b3a-3943bb13976d)


9. Congratulations

    ![image](https://github.com/user-attachments/assets/a02d9373-4d83-4e6a-8155-53194e4cde1b)





