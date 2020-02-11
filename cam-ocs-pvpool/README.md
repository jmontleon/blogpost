Using an OCS NooBaa PV Pool with CAM 1.1.0
==========================================

Introduction
------------
It may be desirable for many reasons to keep your data out of Amazon S3. OCS MCG provides the means to set up a NooBaa PV Pool in order to host your data locally on your cluster. In this example I will use the default gp2 storageclass but if Ceph or another storageclass is desired it could be used just as easily.

These instructions will focus on setting up the CAM Operator, Controller, and UI on Openshift 4.x target cluster and configuring it to use the OCS MCG PV Pool. After this is completed configuration of the source cluster can be completed normally.

Create the openshift-migration namespace
----------------------------------------
Use oc to create the namespace:
```oc create namespace openshift-migration```

Install OCS
-----------
Navigate to OperatorHub in the Openshift Console
![Operator Hub](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OHub.png)

Search for OCS
![Search for OCS](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OCSSearch.png)

Install OCS
![Install OCS](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/OCSInstall.png)

Create the Noobaa System
------------------------
```

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

Install CAM
![Install CAM](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CAMInstall.png)

Click `View 12 more...`
![View 12](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/View12.png)

Select MigrationController
![Select MC](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/SelectMC.png)

Click Create
![Create MC](https://github.com/jmontleon/blogpost/blob/master/cam-ocs-pvpool/CreateMC.png)

Create the Replication Repository
---------------------------------

