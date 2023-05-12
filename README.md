#   Backup and restore of container application using OADP/Velero
## Overview

### What is OADP/Velero?

OADP (OpenShift APIs for Data Protection) is an operator that Red Hat has created to create backup and restore APIs in the OpenShift cluster.
OADP provides the following APIs:

-   Backup
-   Restore
-   Schedule
-   BackupStorageLocation
-   VolumeSnapshotLocation

The OADP operator will install Velero, and OpenShift plugins for Velero to use, for backup and restore operations.

OADP uses Velero backup controller for Kubernetes resources discovery & backup using the Kubernetes API and Velero CSI Plug-In the Persistent Volume backup.

The backup and restore configuration using OADP is quite complex compared to IBM Spectrum Fusion as it requires OpenShift development skills.

### Pre-Requisites

-   Red Hat OpenShift Cluster (Single Node cluster or Multi Node Cluster) – 8 logical CPU, 32GB, Minimum 2 storage disks (OS and Data)
-   You must be logged in as a user with cluster-admin Privilege
-   ODF-LVM Operator to be installed for Single Node OpenShift Cluster OR OpenShift Data Foundation (ODF) operator be installed for Multi Node OpenShift Cluster
-   IBM Cloud Object Storge  - [Create Instance](https://cloud.ibm.com/login?redirect=%2Fobjectstorage)
-   [Install OpenShift CLI](https://docs.openshift.com/container-platform/4.10/cli_reference/openshift_cli/getting-started-cli.html)

### High-Level Architecture Diagram

![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure1.png)

### Installation Steps

Assume that you have already installed the Single Node OpenShift cluster

1.	Install ODF-LVM Operator on Single Node Red Hat OpenShift Cluster 

    ODF-LVM Operator is recommend for only Single Node OpenShift cluster

    Navigate to **OperatorHub**, search for and install **ODF-LVM Operator**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure2.png)

    **Create LVM Cluster**

    Navigate to **LVMCluster** Tab once installation is completed > Click **Create LVM Cluster**> Create

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure3.png)

    **Verify odf-lvm-vg1  storge class is  created**

    Navigate to Storage > StorageClasses

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure4.png)

    **Set odf-lvm-vg1 storage Class to default**

    ODF-LVM StorageClass is required to set default for deploying the container application with dynamic persistent volume on ODF-LVM storage:

    In order to make ODF-LVM StorageClass to default , Navigate to **OpenShift Console** > Go to **Storage** >  **StorageClasses** > Click on **StorageClass “odf-lvm-vg1”** > Click **Annotation** > Add Key **“storageclass.kubernetes.io/is-default-class” and Value ‘true’ > Save**

2. 	Install OpenShift Data Foundation on Multi Node Red Hat Openshift Cluster

    Assume that you have already installed the Multi Node OpenShift cluster

    Navigate to **OperatorHub**, search for and install **OpenShift Data Foundation**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure5.png)

    **CREATING STORAGESYSTEM**

    Click **Create StorageSystem** button after the install is completed

    Go to Product Documentation for Red Hat OpenShift Data Foundation 4.11

    https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.11

    1.  Filter Category by Deploying
    2.  Open deployment documentation your cloud provider.
    3.  Follow Creating an OpenShift Data Foundation cluster instructions.

    Verify OpenShift Data Foundation has installed successfully by login into **OpenShift Console** > Navigate to **Storage** > **Data Foundation** should visible underneath now & its status should be Green

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure6.png)

    **Verify OCS storge class are created**

    1.	ocs-storagecluster-cephfs
    2.	ocs-storagecluster-ceph-rbd
    3.	ocs-storagecluster-ceph-rgw
    4.	openshift-storage.noobaa.io

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure7.png)

    **Set ODF Storage Class to default**

    ODF StorageClass is required to set default for deploying the container application with dynamic persistent volume on ODF storage:

    In order to make ODF StorageClass to default, Navigate to **OpenShift Console** > Go to **StorageClasses** > Click on **ODF StorageClass “ocs-storagecluster-ceph-rbd”** from the list 

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure8.png)

    Click + Add More > Enter the key name **“storageclass.kubernetes.io/is-default-class”** and value “true” > Save

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure9.png)

3.  Create Backup Storage Location

    To save the backup copy to IBM Cloud Object Storage, Create IBM Cloud Object Storage  Instance in IBM Cloud 

    Assume that you have already created the IBM Cloud Object Storage instance. 

    1.  Create a Bucket in IBM Cloud Object Storage

        Login to **IBM Cloud** > Navigate to Resource list > Find the Cloud Object Storage instance that you have created before under the Storage > Access the IBM Cloud Object Storage Instance > Click Buckets > Create Bucket 

        ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure10.png)

        Click **Customize your bucket** > Enter **unique bucket name** > Select **Resiliency** > Select **Location** > Select **Storage Class** > Click **Create bucket**

        ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure11.png)

    2.  Set Permission on Bucket

        Click **Bucket Name** > **Permissions Tab** > Select **Service ID from Access policies** > Select **Role “Writer” and Click Create access policy**

        ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure12.png)

    3.  Copy Endpoint name

        Bucket End point name is required to connect to Object Storage:

        Click **Configuration Tab** > Click **Endpoints**

        ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure13.png)

        Scroll down and copy the Direct Endpoint name from Endpoints

        ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure14.png)

    4.  Copy Access Key and Secrete Key from Service Credential

        To authenticate with IBM Cloud Object storage, Access Key and Secrete key of Service credential are required:

        Navigate to **IBM Cloud Object Storage** > Click **Service Credentials** > Expand **Service Credential** > **Copy Access key from “access_key_id” and Secret key from “secrete_access_key”**

4.	Installing OpenShift API for Data Protection Operator

    You can install the OADP Operator from the Openshift's OperatorHub. You can search for the operator using keywords such as oadp 

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure15.png)

    Now click on Install

    Finally, click on Install again. This will create Project openshift-adp if it does not exist, and install the OADP operator in it.

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure16.png)

5.	Create Credentials Secret for OADP OPERATOR to use

    Now create secret **cloud-credentials** using values obtained from IBM Cloud Object Storage bucket in Project **openshift-adp**.

    From OpenShift Web Console side bar navigate to **Workloads > Secrets** and click **Create > Key/value secret**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure17.png)

    Fill out the following fields:

    -   Secret name: cloud-credentials
    -   Key: cloud
    -   Value:
        -   Replace the values with your own values from earlier steps and enter it in the value field.
        -   [default]
        -   aws_access_key_id=<INSERT_VALUE>
        -   aws_secret_access_key=<INSERT_VALUE>
  
    Note: Do not use quotes while putting values in place of INSERT_VALUE Placeholders

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure18.png)

6.	CREATE THE DATAPROTECTIONAPPLICATION CUSTOM RESOURCE

    From side bars navigate to **Operators > Installed Operators**

    Create an instance of the **DataProtectionApplication** (DPA) CR by clicking on Create Instance as highlighted below:

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure19.png)

    Select Configure via: YAML view

    Finally, copy the values provided below and update fields with comments with information obtained earlier.
    -   update .spec.backupLocations[0].objectStorage.bucket with bucket name from earlier steps.
    -   update .spec.backupLocations[0].config.s3Url with bucket host from earlier steps.

    ```
    kind: DataProtectionApplication
    apiVersion: oadp.openshift.io/v1alpha1
    metadata:
      name: velero-dpa
      namespace: openshift-adp
    spec:
      backupLocations:
        - velero:
            config:
              profile: default
              region: ams03
              s3Url: https://s3.private.ams03.cloud-object-storage.appdomain.cloud
              insecureSkipTLSVerify: "true"
              s3ForcePathStyle: "true"
            credential:
              key: cloud
              name: cloud-credentials
            default: true
            objectStorage:
              bucket: snocontainer-backup
              prefix: velero
            provider: aws
      configuration:
        restic:
          enable: false
        velero:
          defaultPlugins:
            - openshift
            - csi
       - aws
          featureFlags:
          - EnableCSI
    ```

    The object storage we are using is an s3 compatible storage provided by IBM Cloud Object Storage instance. We are using custom s3Url capability of the aws velero plugin to access IBM Cloud Object Storage endpoint in velero.

    Click **Create**
    Verify the status of Data Protection application is showing as **“Condition: Reconciled” and the status of BackupStorage Location is showing as  “Phase: Available”**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure20.png)

7.	Modifying Volumesnapshotclass

    Setting a **DeletionPolicy** of **Retain** on the **VolumeSnapshotClass** will preserve the volume snapshot in the storage system for the lifetime of the Velero backup and will prevent the deletion of the volume snapshot, in the storage system, in the event of a disaster where the namesace with the VolumeSnapshot object may be lost
    
    The Velero CSI plugin, to backup CSI backed PVCs, will choose the VolumeSnapshotClass in the cluster that has the same driver name and also has the **velero.io/csi-volumesnapshot-class: "true"** label set on it.
    
    **Using OpenShift Web Console**, Navigate to **Storage > VolumeSnapshotClasses** and click **odf-lvm-vg1 storgeclass**

    Click **YAML view** to modify values **deletionPolicy and labels** as shown below:

    ```
    apiVersion: snapshot.storage.k8s.io/v1
    deletionPolicy: Retain
    driver: topolvm.cybozu.com
    kind: VolumeSnapshotClass
    metadata:
        name: odf-lvm-vg1
        labels:
            velero.io/csi-volumesnapshot-class: "true"
    ```

8.	Container Application Backup Steps

    Deploy Sample application in OpenShift and verify it creates the persistent volume in ODF LVM storage

    Navigate to OpenShift Console > In **Administrator view > Projects > Create Project**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure21.png)

    Enter **Name** “demoapps” and click **Create**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure22.png)

    Switch to **Developer view** from OpenShift console > change the project to “**demoapps**” > Click **+Add**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure23.png)

    Click **All services** from Developer Catalog > Search for catalog “**Node.js + PostgreSQL**” templates

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure24.png)

    Click catalog **“Node.js + PostgreSQL” templates > Click Instantiate Template > Verify namespace is “demoapps” > Click Create**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure25.png)

    Wait until pods are showing in running state

    Switch back to **Administrator view** from Console > Verify **nodejs-postgresql-persistent-XXX & postgesqlXX** pods are in running state

    Pods are in running state means it has created with PerstistentVolume successfully using odf-lvm-vg1 default StorageClass. 

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure26.png)

    In order to verify persistent volume created, Navigate to **Storage** > **PersistentVolumeClaims** > Verify **“postgresql”** is in **bound** state and its StorgeClass is “**odf-lvm-vg1**" which is a defaut StorageClass that you setup in previous steps.

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure27.png)

9.	Backup Application

    From side menu, navigate to **Operators > Installed Operators Under Project openshift-adp,** click on **OADP Operator**. Under Provided APIs > **Backup**, click on **Create instance**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure28.png)

    In name > type **demoapps-backup**
    
    In **IncludedNamespace** > add  **demoapps**

    In **snapshotVolumes > Check Mark** on snapshotVolumes

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure29.png)

    Click **Create**.

    The status of backup should eventually show **Phase: Complete**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure30.png)

10.	Application Consistent Backup

    OADP/Veloro backs up application data consistently across all the Persistent Volumes (PVs). It ensures that the recovered data is error-free, reliable, and usable.

    In Application Consistency approach, application I/O must be paused before a snapshot is taken by using hooks. Though it achieves the highest level of consistency, it comes at the cost of application downtime that can be disruptive. The pause can be a long time depending on the application. For instructions about specifying these application hooks, see [Backup Hooks](https://velero.io/docs/v1.6/backup-hooks/).

    1.  Backup Hooks

        When performing a backup, you can specify one or more commands to execute in a container in a pod when that pod is being backed up. The commands can be configured to run before any custom action processing (“pre” hooks), or after all custom actions have been completed and any additional items specified by custom action have been backed up (“post” hooks). Note that hooks are not executed within a shell on the containers.

        The backup Hooks can be specified via annotations on the pod itself
    
    2.  Annotations

        You can use the following annotations on a pod to make Velero execute a hook when backing up the pod:

        Pre hooks:
        -   pre.hook.backup.velero.io/container
            -   The container where the command should be executed. Defaults to the first container in the pod. Optional.
        -   pre.hook.backup.velero.io/command
            -   The command to execute. If you need multiple arguments, specify the command as a JSON array, such as ["/usr/bin/uname", "-a"]
        -   pre.hook.backup.velero.io/on-error
            -   What to do if the command returns a non-zero exit code. Defaults to Fail. Valid values are Fail and Continue. Optional.
        -   pre.hook.backup.velero.io/timeout
            -   How long to wait for the command to execute. The hook is considered in error if the command exceeds the timeout. Defaults to 30s. Optional.
        Post hooks
        -   post.hook.backup.velero.io/container
            -   The container where the command should be executed. Defaults to the first container in the pod. Optional.
        -   post.hook.backup.velero.io/command
            -   The command to execute. If you need multiple arguments, specify the command as a JSON array, such as ["/usr/bin/uname", "-a"]
        -   post.hook.backup.velero.io/on-error
            -   What to do if the command returns a non-zero exit code. Defaults to Fail. Valid values are Fail and Continue. Optional.
        -   post.hook.backup.velero.io/timeout
            -   How long to wait for the command to execute. The hook is considered in error if the command exceeds the timeout. Defaults to 30s. Optional.

        Apply the pre and post backup hooks annotations directly to declarative deployment.   Below is an example of what updating an object in place might look like.

        ```
        oc annotate pod -n demoapps -l name=postgresql \
		pre.hook.backup.velero.io/command='["/bin/bash", "-c", 'psql  -c "\"select pg_start_backup('app_cons');\""']' \ pre.hook.backup.velero.io/container=postgresql \
		post.hook.backup.velero.io/command='["/bin/bash", "-c", 'psql -c "\"select pg_stop_backup( );\"""]'\ 
		post.hook.backup.velero.io/container=postgresql

        ```
11.	Restore application

    Backups that are taken on an **Object Storage** can be restored to same project as well as new or different project 

    From side menu, navigate to **Operators** > **Installed Operators** Under Project **openshift-adp**, click on **OADP Operator**. Under **Provided APIs** > **Restore**, click on **Create instance**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure31.png)

    Under **Backup Name**, type **backup**

    In **Name** > type restore job name “restore-demoapps”

    In **backupName** > type the backup job name that was successful before and  you want to restore from , i.e. “testbackup”

    In **IncludedNamespaces** >  add target namespace  “**demoapps**” >  check **restorePVs**

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure32.png)

    Click **Create**.

    The status of **restore** should eventually show **Phase: Completed**.

    After a few minutes, you should see the chat application up and running. You can check via **Workloads > Pods > Project: demoapps** and see the following 

    ![alt text](https://github.com/mdhamat/oadp-velero/blob/3f86af055fc0f38641783c27d41a7fcb57225117/Images/Figure33.png)

    Try to access the application via URL

## Conclusion

This tutorial walked you through the steps of installation of ODF-LVM Operator on Single Node OpenShift Cluster and Installation of OADP/Velero Operator in OpenShift cluster.  This tutorial also showed you how to backup the container application to IBM Cloud Object Storage and its restore steps using OADP/Velero

 












 






		












