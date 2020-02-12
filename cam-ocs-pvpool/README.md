Using an OpenShift Container Storage NooBaa PV Pool with CAM 1.1
=================================================================

Introduction
------------
Red Hat provides a tool for helping users migrate workloads from OpenShift 3 to OpenShift 4 called [Cluster Application Migration (CAM) Tool](https://www.openshift.com/learn/topics/migration).  One of the requirements to using CAM is to provide Object Storage, there are several types of Object Storage CAM can work with; AWS S3, Azure Blob Storage, and Google Cloud Storage.  In this blog post we will describe how to leverage [Multi-Cloud Object Gateway (MCG)](https://blog.openshift.com/introducing-multi-cloud-object-gateway-for-openshift/) from [OpenShift Container Storage 4](https://www.openshift.com/products/container-storage/) to satisfy CAM's needs for S3 Object Storage, specifically we will use a NooBaa PV Pool Backing Store (tech preview only in OpenShift Container Storage 4.2) to allow a solution of object storage that will work for on-premise and even disconnected customers (coming in OpenShift Container Storage 4.3) to perform migrations.

It may be desirable for many reasons to keep your data out of Amazon S3. OpenShift Container Storage MCG provides the means to set up a NooBaa PV Pool in order to host your data locally on your cluster. In this example I will use my cluster's default storageclass of 'gp2', yet this could just as easily be Ceph, or another storageclass that has been configured on the cluster.

These instructions will focus on setting up the CAM Operator, Controller, and UI on Openshift 4.3 target cluster and configuring it to use the OpenShift Container Storage MCG PV Pool. After this is completed configuration of the source cluster can be completed normally.

Create the requisite namespaces
----------------------------------------
Use oc to create the namespaces:
```
oc create namespace openshift-migration
oc create namespace openshift-storage
```

Install OpenShift Container Storage
-----------------------------------
Navigate to OperatorHub in the Openshift Console
![Operator Hub](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OHub.png)

Search for OCS
![Search for OCS](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OCSSearch.png)

Install OCS in the openshift-storage namespace
![Install OCS](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/InstallOCS.png)

Install CAM
-----------
Navigate to OperatorHub in the Openshift Console
![Operator Hub](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OHub.png)

Search for CAM
![Search for CAM](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CAMSearch.png)

Install CAM in the openshift-migration namespace
![Install CAM](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CAMInstall.png)

Click `View 12 more...`
![View](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/View.png)

Select MigrationController
![Select MC](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/SelectMC.png)

Click Create
![Create MC](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CreateMC.png)

Create the NooBaa System
------------------------
Add the contents below to `noobaa.yml` If you're testing in a small cluster it may be necessary to reduce the CPU requests to `0.1`. Then run `oc create -f noobaa.yml`.

```
apiVersion: noobaa.io/v1alpha1
kind: NooBaa
metadata:
  name: noobaa
  namespace: openshift-storage
spec:
 dbResources:
   requests:
     cpu: 0.5
     memory: 1Gi
 coreResources:
   requests:
     cpu: 0.5
     memory: 500Mi
```

PV Pool and Bucket
------------------
Create `bs.yml` with the following contents. You can change the number of volumes, size of the volumes, and the storageClass used for PV's here. In this example our data will be spread across 3 volumes with 50Gi of space each. Run `oc create -f bs.yml` after making any desired edits.

```
apiVersion: noobaa.io/v1alpha1
kind: BackingStore
metadata:
  finalizers:
  - noobaa.io/finalizer
  labels:
    app: noobaa
  name: mcg-pv-pool-bs
  namespace: openshift-storage
spec:
  pvPool:
    numVolumes: 3
    resources:
      requests:
        storage: 50Gi
    storageClass: gp2
  type: pv-pool
```

Create `bc.yml` with the following contents and then run `oc create -f bc.yml`. Alternative placements are possible, but not covered here.
```
apiVersion: noobaa.io/v1alpha1
kind: BucketClass
metadata:
  labels:
    app: noobaa
  name: mcg-pv-pool-bc
  namespace: openshift-storage
spec:
  placementPolicy:
    tiers:
    - backingStores:
      - mcg-pv-pool-bs
      placement: Spread
```

Create `obc.yml` with the following contents. In this example we will create a bucket called `migstorage`. Run `oc create -f obc.yml` after saving the file.
```
apiVersion: objectbucket.io/v1alpha1
kind: ObjectBucketClaim
metadata:
  name: migstorage
  namespace: openshift-storage
spec:
  bucketName: migstorage
  storageClassName: openshift-storage.noobaa.io
  additionalConfig:
    bucketclass: mcg-pv-pool-bc
```

After this resource is created you will want to watch the status and wait for the objectbucketclaim status to change to `Bound`. This may take 5-10 minutes and can be watched with the command `watch -n 30 'oc get -n openshift-storage objectbucketclaim migstorage -o yaml'`. The noobaa-operator logs can also be monitored for progress.

Create the Replication Repository
---------------------------------
Use `oc get route -n openshift-migration migration` to find the hostname for the CAM UI and navigate to it.

Click Add next to Replication Repositories once logged in.
![Add](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/Add.png)

Select AWS from the drop down.
![AWS](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/Aws.png)

1. Give the Replication Repository a name.
1. The bucket name should be set to match the name chosen in the `ObjectBucketClaim` resource. We used `migstorage` in our example.
1. Leave the Region blank.
1. Run `oc get route -n openshift-storage s3` to find the route for NooBaa S3. Prefix the value with https:// when entering it into the UI.
1. Run `oc get secret -n openshift-storage migstorage -o go-template='{{ .data.AWS_ACCESS_KEY_ID }}' | base64 -d` to get the Access Key ID value.
1. Run `oc get secret -n openshift-storage migstorage -o go-template='{{ .data.AWS_SECRET_ACCESS_KEY }}' | base64 -d` to get the Secret Access Key.
1. Click `Add Repository`.

At this point you should get confirmation that the connection was successful.
![Update](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/Update.png)

Preparing for Migrations
------------------------
From this point set up your source cluster normally and [begin your migrations as you normally would](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/migration/migrating-openshift-container-platform-3-to-4#migrating-applications-with-cam_targetcluster-3-4). We are looking into automating these steps in a future release of our Operator to make it even easier to get started with OpenShift Container Storage and CAM.

Links
-----
[mig-operator PV Pool feature branch](https://github.com/fusor/mig-operator/tree/feature-pv-pool)
