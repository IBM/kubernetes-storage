# Mount a Remote Object Storage as Local Filesystem in Userspace (FUSE) with S3FS

## About FUSE

[Filesystem in Userspace (FUSE)](https://en.wikipedia.org/wiki/Filesystem_in_Userspace) lets non-privileged users create a file system in their user space. The FUSE project consists of two components: a FUSE kernel module that is part of the Linux kernel since version 2.6.14, and the `libfuse` userspace library. The [`libfuse` library](https://github.com/libfuse/libfuse) provides a reference implementation for communication with the FUSE kernel module, providing client functions to mount the file system, unmount it, read requests from the kernel, and send responses back. FUSE is particularly useful for writing Virtual File Systems (VFS).

### s3fs

`s3fs` or `s3fs-fuse` is an Amazon S3 (Services Simple Storage) and S3-based object stores compatible FUSE filesystem disk management utility that supports a subset of [Single UNIX Specification (SUS)](https://en.wikipedia.org/wiki/Single_UNIX_Specification) or [POSIX](https://en.wikipedia.org/wiki/POSIX) including reading/writing files, directories, symlinks, mode, uid/gid, and extended attributes, while preserving the original file format, e.g. a plain text or MS Word document

Applications that need to read and write to an NFS-style filesystem can use s3fs. It with easily integrates applications with S3 compatible storage like [IBM Cloud Object Storage](https://cloud.ibm.com/docs/cloud-object-storage). s3fs also allows you to interact with your cloud storage using familiar shell commands, like `ls` for listing or `cp` to copy files.

Performance is not equal to a true local filesystem, but you can use some advanced options to increase throughput. Object storage services have high-latency for time to first byte and lack `random write` access (requires rewriting the full object). Workloads that only read big files, like deep learning workloads, can achieve good throughput using s3fs.

s3fs on macOS uses [`osxfuse`](https://osxfuse.github.io/). osxfuse is FUSE for macOS.

## Lab

In this lab, you will [mount a bucket using s3fs](https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-s3fs). 

### Connect to IBM Cloud

You have to be logged in to your IBM Cloud account,

```
ibmcloud login -u <IBMId>
```

If you are using Single Sign-On (SSO) use the `-sso` flag to log in.

Then, set an environment variable with the bucket name, and upload a document to your bucket in the IBM Cloud Object Storage instance. 

```
COS_BUCKET_NAME=<cos-bucket-name>
```

### Upload an Object to Object Storage

Create a new document,

```
COS_OBJECT_KEY=helloworld.txt
echo "Hello World! Today is $(date)" > $COS_OBJECT_KEY
```

[Upload an object using S3Manager](https://cloud.ibm.com/docs/cli?topic=cloud-object-storage-cli-plugin-ic-cos-cli#ic-upload-s3manager),

```
ibmcloud cos upload --bucket $COS_BUCKET_NAME --key $COS_OBJECT_KEY --file ./helloworld.txt --content-language en-US --content-type "text/plain"

OK
Successfully uploaded object 'helloworld.txt' to bucket 'e59a327194-cos-1-bucket1'.
```

### Install s3fs

Install `s3fs`,
```
brew install s3fs
```

Create an IBM Cloud Object Storage instance with HMAC keys,

```
ibmcloud resource service-key-create $COS_CREDENTIALS Writer --instance-name $COS_NAME --parameters '{"HMAC":true}'
```

Will create credentials including among other HMAC keys,

```
"cos_hmac_keys": {
    "access_key_id": "c407e90c41c3463b8e7722048aa48edc",
    "secret_access_key": "0f8c2cb6ef82c63d8d1f935a8ef6a87fe1bc16ed1ba8483c"
},
```

Create environment variables with the HMAC keys, and create an S3FS password file,

```
COS_ACCESS_KEY=$(ibmcloud resource service-key $COS_CREDENTIALS --output json | jq -r '.[0].credentials.cos_hmac_keys.access_key_id')
COS_SECRET_KEY=$(ibmcloud resource service-key $COS_CREDENTIALS --output json | jq -r '.[0].credentials.cos_hmac_keys.secret_access_key')

COS_S3FS_PASSWORD_FILE=s3fs-passwd
echo $COS_ACCESS_KEY:$COS_SECRET_KEY > $COS_S3FS_PASSWORD_FILE
chmod 0600 $COS_S3FS_PASSWORD_FILE
```

### Mount Local File System

Mount the local directory using S3FS,

```
mkdir cos_data
LOCAL_MOUNTPOINT=$(pwd)/cos_data
COS_PUBLIC_ENDPOINT=s3.us-south.cloud-object-storage.appdomain.cloud

s3fs $COS_BUCKET_NAME -o passwd_file=$(pwd)/$COS_S3FS_PASSWORD_FILE -o url=https://$COS_PUBLIC_ENDPOINT $LOCAL_MOUNTPOINT
```

You should see the content of your IBM Cloud Object Storage, e.g using the Finder on macos

![S3FS mounted IBM Cloud Object Storage bucket](../.gitbook/images/s3fs/s3fs-mount-cos-bucket.png)

or using the cli on macos,

![S3FS mounted IBM Cloud Object Storage bucket CLI](../.gitbook/images/s3fs/s3fs-mount-cos-bucket-cli.png)

### References

* https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-s3fs