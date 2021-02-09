# Object Storage with Kubernetes

## About this Lab

This hands-on lab for object storage on Kubernetes steps you through the creation and configuration of persistent storage for MongoDB using an encrypted `IBM Cloud Object Storage` bucket on IBM Cloud Object Storage.

We use the [IBM Cloud Object Storage plugin](https://github.com/IBM/ibmcloud-object-storage-plugin) to enable Kubernetes pods to access `IBM Cloud Object Storage` buckets using the cloud native PersistentVolume (PV) and PersistentVolumeClaim (PVC) resources. The plugin has two components: a dynamic provisioner and a FlexVolume driver for mounting the buckets using `s3fs-fuse` on a worker node.

`FlexVolume` and the `Container Storage Interface (CSI)` are so-called `out-of-tree` volume plugins. `Out-of-tree` volume plugins enable storage developers to create custom storage plugins not included (`in-tree`) in the core Kubernetes APIs. For background information about `FlexVolume`, go to the [flexvolume](../flexvolume/README.md) readme.

The FlexVolume driver uses [`s3fs`](https://github.com/s3fs-fuse/s3fs-fuse) to allow Linux and macOS to mount an S3 bucket via FUSE. If you want to learn more about `s3fs-fuse` and how FUSE works, you can do the additional [s3fs lab](../fuse/README.md).

![Cloud Object Storage architecture](images/cos-plugin-architecture.png)

This Object Storage lab consists of the following steps:

1. To setup client CLI and Kubernetes cluster, go to [Setup](../setup/README.md),
2. To learn more about what Object Storage is, go to [About Object Storage](ABOUT-COS.md)
3. Create Object Storage instance, go [here](COS.md),
4. Configure your Kubernetes Cluster, go [here](CLUSTER.md),
5. To configure the `IBM Cloud Object Storage plugin`, go [here](COS-PLUGIN.md),
6. Create the `PersistentVolumeClaim` with dynamic provisioning using the `ibmc` plugin, go [here](PVC.md).
7. Deploy MongoDB using Object Storage, go [here](MONGODB.md).

Start with [Setup](setup/README.md).

## Other Labs

Related labs using Object Storage are:

* [Optional] (TBD) Deploy Guestbook with MongoDB and Object Storage.
* [Share Documents using Cloud Object Storage](share-docs-with-cos/README.md).
* [Mount a Remote Object Storage as Local Filesystem in Userspace (FUSE) with S3FS](../fuse/README.md)

## Next

[1. Setup](setup/README.md)
