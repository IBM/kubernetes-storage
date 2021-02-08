# IBM Cloud Object Storage plugin

[IBM Cloud Object Storage plug-in](https://github.com/IBM/ibmcloud-object-storage-plugin/blob/master/README.md) is a Kubernetes Volume plug-in that enables Kubernetes pods to access IBM Cloud Object Storage buckets.

The plug-in has two components:

1. a dynamic provisioner (Object Storage Bucket Provisioner), and
1. a FlexVolume driver (Kube FlexDriver) for mounting the buckets using `s3fs-fuse` on a worker node. You can read more about Filesystems for User Spaces (FUSE) and `s3fs-fuse` in the [s3fs-fuse lab](../fuse/README.md).

![IBM Cloud Object Storage plugin architecture](../../images/ibmcos-plugin-arch.png)

See: [Fundamentals of IBM Cloud Object Storage](https://medium.com/ai-platforms-research/fundamentals-of-ibm-cloud-storage-solutions-8739f36f024e.)

The `IBM Cloud Object Storage` driver depends on `s3fs` binaries and deploys them by launching a daemonset that runs one pod on each worker node that will then open a tunnel into the worker itself (which requires privileged access) to copy its binaries.

A better approach in the near future will be using CSI drivers which will run completely containerized and thus not depend on advanced privileges. CSI is an independent standard that also applies to other cloud orchestrators (COs) like Docker and Mesos and it will be used through the same Kubernetes primitives mentioned above (PVs, PVCs and storage classes).

S3, the Simple Storage Service, originated as Amazon. the central storage component for Netflix (which developed S3mper to provide a consistent secondary index on top of an eventually consistent storage) as well as Reddit, Pinterest, Tumblr and others. IBM’s Cloud Object Storage is S3 compatible.

Instead of always providing all parameters via the API, it is more convenient to mount the bucket as a folder onto the existing file system. This can be done via s3fs or goofys.

## Using s3fs

Create a credentials file `~/.cos_creds` with:

```console
<ACCESS_KEY>:<SECRET_ACCESS_KEY>
```

Make sure neither your group nor others have access rights to this file, e.g. via `chmod o-rwx ~/.cos_creds`. You can then mount the bucket with,

```console
s3fs dlaas-ci-tf-training-data-us-standard ~/testmount -o passwd_file= ~/.cos_creds -o url=https://s3-api.us-geo.objectstorage.softlayer.net -o use_path_request_style
```

Note that s3fs can optionally provide extensive logging information:

```console
s3fs dlaas-ci-tf-training-data-us-standard ~/testmount -o passwd_file= ~/.cos_creds -o dbglevel=info -f -o curldbg -o url=https://s3-api.us-geo.objectstorage.softlayer.net -o use_path_request_style &
```

In simple test environments it might be sufficient to mount the folder as a host volume. you could achieve the same through a PVC. For Kubernetes clusters in production it is more desirable to properly mount volumes via drivers, using a Flex driver.

[s3fs](https://github.com/s3fs-fuse/s3fs-fuse) allows Linux and macOS to mount an S3 bucket via FUSE. With the s3fs mountpoint, you can access your objects as if they were local files, i.e. list them with ls, copy them with cp, and access them seamlessly from any application built to work with local files. Unlike many file-to-object solutions, s3fs maintains a one-to-one mapping of files to objects.

s3fs can yield good performance results when used with workloads reading or writing relatively large files (say, 20MB+) sequentially. On the other hand, you probably do not want to use s3fs with workloads accessing a database (as file locking is not supported), or workloads requiring random read or write access to files (because of the one-to-one file to object mapping). s3fs is not suitable for accessing data that is being mutated (other than by the s3fs instance itself).

## Next

[Lab 5: Add Object Storage to a Persistent Database](README.md)
