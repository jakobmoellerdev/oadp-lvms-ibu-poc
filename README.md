# OADP LVMS IBU POC

## Overview

This repository contains sample code for restoring Applications using LVMS with OADP after an Image Based Upgrade (IBU). 
The manifests are organized into the following directories:

* `test-backup-setup` contains a sample `DataProtectionApplication` for setting up OADP for backing up to object storage
* `test-lvms-cluster` contains a sample `LVMCluster` configuration for setting up LVMS with a StorageClass
* `test-base` contains a sample application with a PVC using the storage class of `LVMCluster`
* `test-clone` and `test-snapshot` contain sample manifests for cloning and snapshotting the data in base
* `test-backup` contains the Backup Specification for backing up both LVMCluster as well as the Application. This is to be run before the Upgrade.
* `test-restore` contains the Restore Specification for restoring both LVMCluster as well as the Application. This is to be run after the Upgrade completed.

## Prerequisites

* OADP is installed into `openshift-adp` namespace.
* LVMS is installed into `openshift-storage` namespace and is also installed in the Seed Cluster used for the IBU Seed Image.

## Runbook

1. Create the `test-backup-setup` application to setup OADP for backing up to object storage.
2. Create the `test-lvms-cluster` application to setup LVMS with a StorageClass.
3. Create the `test-base` application to create a PVC using the storage class of `LVMCluster` and let the application use it. 
   Write arbitrary data into `/usr/share/nginx/html/index.html` of the original pod for later verification of the restoration
4. Create the `test-clone` application to clone the data in `test-base` into a new PVC. 
   Write arbitrary data into `/usr/share/nginx/html/index.html` of the cloned pod for later verification of the restoration
5. Create the `test-snapshot` application to snapshot the data in `test-base` into a new PVC. 
   Write arbitrary data into `/usr/share/nginx/html/index.html` of the snapshot pod for later verification of the restoration
6. IMPORTANT: Change all PersistentVolumes to `.spec.persistentVolumeReclaimPolicy=Retain`. 
   This will cause Velero to restore them instead of ignoring them during the Restore process
7. Create the `test-backup` resources to backup both LVMCluster as well as the Application. 
   This is to be run before the Upgrade.
8. Confirm that the Backups have been created
9. Perform the Upgrade in the IBU CR and wait for the Upgrade to reboot the node and complete
10. Create the `test-restore` resources to restore both LVMCluster as well as the Application. 
    This is to be run after the Upgrade.
11. Confirm that the Restores have been created and the resources are all recreated as well
12. Confirm that the data that you wrote to the nginx directories is still visible
13. Confirm that creating new PVCs still works
