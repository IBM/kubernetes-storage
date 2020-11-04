# Lab 2 File storage with kubernetes



The application is the [Guestbook App](https://github.com/IBM/guestbook), which is a sample multi-tier web application.

## Review the storage classes for file storage

```bash
kubectl get storageclasses | grep file
```

Expected output:
```
$ kubectl get storageclasses | grep file

default                    ibm.io/ibmc-file   Delete          Immediate           false                  27m
ibmc-file-bronze           ibm.io/ibmc-file   Delete          Immediate           false                  27m
ibmc-file-bronze-gid       ibm.io/ibmc-file   Delete          Immediate           false                  27m
ibmc-file-custom           ibm.io/ibmc-file   Delete          Immediate           false                  27m
ibmc-file-gold (default)   ibm.io/ibmc-file   Delete          Immediate           false                  27m
ibmc-file-gold-gid         ibm.io/ibmc-file   Delete          Immediate           false                  27m
ibmc-file-retain-bronze    ibm.io/ibmc-file   Retain          Immediate           false                  27m
ibmc-file-retain-custom    ibm.io/ibmc-file   Retain          Immediate           false                  27m
ibmc-file-retain-gold      ibm.io/ibmc-file   Retain          Immediate           false                  27m
ibmc-file-retain-silver    ibm.io/ibmc-file   Retain          Immediate           false                  27m
ibmc-file-silver           ibm.io/ibmc-file   Delete          Immediate           false                  27m
ibmc-file-silver-gid       ibm.io/ibmc-file   Delete          Immediate           false                  27m
```

This lab uses the default `ibmc-file-gold`

```
kubectl describe storageclass ibmc-file-gold
Name:            ibmc-file-gold
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"labels":{"kubernetes.io/cluster-service":"true"},"name":"ibmc-file-gold"},"parameters":{"billingType":"hourly","classVersion":"2","iopsPerGB":"10","sizeRange":"[20-4000]Gi","type":"Endurance"},"provisioner":"ibm.io/ibmc-file","reclaimPolicy":"Delete"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           ibm.io/ibmc-file
Parameters:            billingType=hourly,classVersion=2,iopsPerGB=10,sizeRange=[20-4000]Gi,type=Endurance
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     Immediate
Events:                <none>
```