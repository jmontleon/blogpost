Using an OCS NooBaa PV Pool with CAM 1.1.0
==========================================

Introduction
------------
It may be desirable for many reasons to keep your data out of Amazon S3. OCS MCG provides the means to set up a NooBaa PV Pool in order to host your data locally on your cluster. In this example I will use the default gp2 storageclass but if Ceph or another storageclass is desired it could be used just as easily.

These instructions will focus on setting up the CAM Operator, Controller, and UI on Openshift 4.x target cluster and configuring it to use the OCS MCG PV Pool. After this is completed configuration of the source cluster can be completed normally.

Create the requisite namespaces
----------------------------------------
Use oc to create the namespaces:
```
oc create namespace openshift-migration
oc create namespace ocs-storage
```

Install OCS
-----------
Navigate to OperatorHub in the Openshift Console
![Operator Hub](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OHub.png)

Search for OCS
![Search for OCS](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OCSSearch.png)

Install OCS in the openshift-migration namespace
![Install OCS](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OCSInstall.png)

Create the Noobaa System
------------------------
Add the contents below to `noobaa.yml` If you're testing in a small cluster it may be necessary to reduce the CPU requests to `0.1`. Then run `oc create -f noobaa.yml`.

```
apiVersion: noobaa.io/v1alpha1
kind: NooBaa
metadata:
  name: noobaa
  namespace: ocs-storage
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
```

```

```

```

```

```

Install CAM
-----------
Navigate to OperatorHub in the Openshift Console
![Operator Hub](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OHub.png)

Search for CAM
![Search for CAM](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CAMSearch.png)

Install CAM in the openshift-migration namespace
![Install CAM](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CAMInstall.png)

Click `View 12 more...`
![View 12](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/View12.png)

Select MigrationController
![Select MC](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/SelectMC.png)

Click Create
![Create MC](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CreateMC.png)

Create the Replication Repository
---------------------------------

