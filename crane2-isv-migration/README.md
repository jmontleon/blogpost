# Crane 2.0 Migration of the ISV Registry

## Introduction
The Konveyor Crane project  has been leading the way in providing the capability to Migrate applications from OpenShift 3.x to Openshift 4.x for almost two years. With Crane 2.0 we have several goals in mind to improve the process.

- Produce [crane-lib](https://github.com/konveyor/crane-lib), a set of reusable libraries for different aspects of the migration process. 
  - Export
  - Transform
  - State Transfer
  - Imge Transfer
- Enable non-admins to complete migrations.
- Prepare applications for use with GitOps.
- Enable Kubernetes to OpenShift migrations

To prove the reusability of these libraries we will replace code in Crane 1.x with crane-lib as different aspects reach maturity. We intend to use the new state transfer library in crane 1.6.0. We will also be releasing a CLI utlity that aims to allow doing migrations in more granular steps. We also have ideas for other projects we would like to integrate this library with so that we can all benefit by sharing expertise and development.

## First Steps
Our first concern was proving that we could orchestrate a migration from one cluster to another with a CLI utility and reduced privileges so we set off to create a proof of concept. The developers on the team each worked on an aspect and created a utility that, for the purposes of demoing, we would string together using scripts in order to complete a migration.  
  
We already had a utility to migrate images.  Resource discovery, export, and transform was very closely tied together so work and the state transfer was started on a go utility that could complete these tasks. The state transfer component was initially done using Ansible.

In April we were able to demonstrate a non-admin migration of a stateful application from EKS to ROSA.

## ISV migration
An opportunity came up to use Crane 2.0 for the ISV migration. The production migration cluster contains about 6,500 namespaces, and over 100,000 image objects. It is rated as the top cluster by object count on OSD, and 12th globally, so it would be an excellent trial for the Crane 2.0 utility to pass, but we were missing functionality to perform such a large migration.

Fortunately we had a development environment to work in that contained a subset of the date in production. The most obvious problem with the new utility was the inability to process more than one namespace at a time. In order to facilitate batch migration we decided to use Ansible rather than extensively rewriting PoC code that would be thrown away. We also decided to make use of the existing image migration utility, Konveyor ImageStream-migrate, since the Crane CLI does not yet have this functionality. Though imagestream-migrate proved overall to be reliable we discovered that the documentation was repetitive and difficult to follow, so we made a large update to clarify the installation and migration procedures.

As testing in development proceeded the migration steps solidified:
- Perform image migrations, which also recreates the namespaces on the destination cluster.
- Perform k8s resource export and transform
- Perform oc create on the transformed resources

As work progressed additional minor bugs were also uncovered and fixed along the way and performance issues were addressed to the extent we could with this workflow.

The final migration completed on July 15th, after 9 days, most of which was dealing with image migrations. Multiple batches of images were migrated in parallel and we learned that at about 10 parallel runs it would take down the cluster. The feedback we received is that namespace creation is extremely intensive, particularly for etcd. Parallel runs were limited to a max of 7 instances after this painful lesson, with no more than 3 of them creating namespaces simultaneously, which prevented further trouble. The K8S resource migration was completed afterwards and was uneventful.

## Next Steps
The imagestream migration utility proved to be slower than expected on reruns. During the final migration it was observed that many of the images had a large number of tags. A possible cause for this is that we are unable to affect the QPS and Burst setting for API calls when using Ansible. Since Ansible was doing most of the heavy lifting outside of the actual transfers this may have been a cause. We experienced similarly poor results when operating on a large number of resources with the PoC utilities as well.

However, the PoC export and transform utilities were written in go and by exposing the QPS and Burst settings we were able to quickly cut down the time to run to a third of their original. As we move forward bringing more of this functionality into crane-lib and the new crane CLI we aim to be mindful of performance and doing what we can to make migrations as fast as possible.

We are also planning to integrate crane-lib into other projects, starting with the state transfer package being used in Crane 1.6.0 for direct volume migrations. We're also looking to form relationships with other projects so we can benefit from joint expertise and development in solving the similar problems we face.

## Links
[Konveyor Crane Project](https://www.konveyor.io/crane)
[crane-lib](https://github.com/konveyor/crane-lib)
[Crane 2.0 CLI](https://github.com/konveyor/crane)
[Imagestream Migrate](https://github.com/konveyor/imagestream-migrate)
[Crane 2.0 PoC export and transform CLI](https://github.com/water-hole/poc-retablir)
[Crane 2.0 PoC transform library](https://github.com/water-hole/transformations)
[Crane 2.0 State Transfer Utility](https://github.com/jmontleon/rsync-poc)
[Crane 2.0 PoC Batch Migration Playbooks](https://github.com/water-hole/crane-batch-migration)
