# Lab 1. Non-pesistent storage with Kubernetes

Storing data in containers or worker nodes are considered as the [non-persistent](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#local-ephemeral-storage) forms of data storage.
In this lab, we will explore storage options on the IBM Kubernetes worker nodes. Follow this [lab](https://github.com/remkohdev/docker101/tree/master/workshop/lab-3) is you are interested in learning more about container based storage.

The lab covers the following topics:
- Create and claim persistent volumes based on the [primary]() and secondary storage available on the worker nodes.
- Make the volumes available in the `Guestbook` application.
- Use the volumes to stroage application cache and debug information.
- Access the data from the guestbook container using the Kubernetes CLI.
- Claim back the storage resources and clean up.


The primary storage maps to the volume type `hostPath` and the secondary storage maps to `emptyDir`. Learn more about Kubernetes volume types [here](https://kubernetes.io/docs/concepts/storage/volumes/).

## Reserve Persistent Volumes

From the cloud shell prompt, run the following commands to get the guestbook application and the kubernetes configuration needed for the storage labs.

```bash
cd $HOME
git clone --branch fs https://github.com/IBM/guestbook-nodejs.git
git clone --branch storage https://github.com/rojanjose/guestbook-config.git
cd $HOME/guestbook-config/storage
```

Let's start with reserving the Persistent volume from the primary storage.
Review the yaml file `pv-hostpath.yaml`. Note the values set for `type`, `storageClassName` and `hostPath`.

```console
apiVersion: v1
kind: PersistentVolume
metadata:
  name: guestbook-primary-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mnt/data"
```

Create the persistent volume as shown in the comamand below.
```
kubectl create -f pv-hostpath.yaml
persistentvolume/guestbook-primary-pv created

kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM                   STORAGECLASS       REASON   AGE
guestbook-primary-pv                       10Gi       RWO            Retain           Available                           manual                      13s
```

PVC yaml:
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: guestbook-local-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 3Gi
```

Create PVC:

```
kubectl create -f guestbook-local-pvc.yaml
persistentvolumeclaim/guestbook-local-pvc created
❯ kubectl get pvc
NAME                  STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
guestbook-local-pvc   Bound    guestbook-local-pv                         10Gi       RWX            manual             6s
```


## Guestbook application using storage

The application is the [Guestbook App](https://github.com/IBM/guestbook-nodejs), which is a sample multi-tier web application.


Build the image
Misc: (building v3 image)
```
docker build -t rojanjose/guestbook:v31 .
docker push rojanjose/guestbook:v31
https://hub.docker.com/repository/docker/rojanjose/guestbook/tags?page=1
```


Main.go file:
Line 186
```
//Setip data file
	f, err := os.OpenFile("data/datafile.txt", os.O_RDWR | os.O_CREATE | os.O_APPEND, 0666)
	
```

Deployment yaml:
```
apiVersion: apps/v1
kind: Deployment
...
    spec:
      containers:
        - name: guestbook
          image: rojanjose/guestbook:v31
          ports:
          - name: http-server
            containerPort: 3000
          volumeMounts:
          - name: guestbook-data-volume
            mountPath: /app/data
      volumes:
      - name: guestbook-data-volume
        persistentVolumeClaim:
          claimName: guestbook-local-pvc
```

Deploy Guestbook application:

```
❯ kubectl create -f guestbook-deployment.yaml
deployment.apps/guestbook-v1 created
❯ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
guestbook-v1-6f55cb54c5-jb89d   1/1     Running   0          14s
❯ kubectl create -f guestbook-service.yaml
service/guestbook created
```

Load data:

![Guestbook images](images/guestbook-local-data.png)


Log into the pod:

```
kubectl exec -it guestbook-v1-6f55cb54c5-jb89d --  busybox sh


BusyBox v1.21.1 (Ubuntu 1:1.21.0-1ubuntu1) built-in shell (ash)
Enter 'help' for a list of built-in commands.

/app # cat data/datafile.txt
2020/11/04 05:14:29 Logging guestbook data:
2020/11/04 05:20:14 One
2020/11/04 05:20:17 Two
2020/11/04 05:20:20 Three

/app # df -ah
Filesystem                Size      Used Available Use% Mounted on
overlay                  97.9G      1.4G     91.5G   2% /
proc                         0         0         0   0% /proc
tmpfs                    64.0M         0     64.0M   0% /dev
devpts                       0         0         0   0% /dev/pts
cgroup                       0         0         0   0% /sys/fs/cgroup/perf_event
...

/dev/xvda2               24.2G      2.2G     20.8G   9% /app/data
...
tmpfs                     7.8G         0      7.8G   0% /sys/firmware
```

Kill the pod:

```
kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
guestbook-v1-6f55cb54c5-jb89d   1/1     Running   0          12m
❯
❯
❯ kubectl delete pod guestbook-v1-6f55cb54c5-jb89d
pod "guestbook-v1-6f55cb54c5-jb89d" deleted
❯ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
guestbook-v1-6f55cb54c5-gctwt   1/1     Running   0          11s
```

![Guestbook images](images/guestbook-local-data-deleted.png)

Enter data:
![Guestbook images](images/guestbook-local-data-reload.png)


```
kubectl exec -it guestbook-v1-6f55cb54c5-gctwt --  busybox sh


BusyBox v1.21.1 (Ubuntu 1:1.21.0-1ubuntu1) built-in shell (ash)
Enter 'help' for a list of built-in commands.

/app # ls -alt
total 8972
drwxr-xr-x    1 root     root          4096 Nov  4 05:27 .
drwxr-xr-x    1 root     root          4096 Nov  4 05:27 ..
drwxr-xr-x    2 root     root          4096 Nov  4 05:14 data
drwxr-xr-x    1 root     root          4096 Nov  4 05:12 public
-rwxr-xr-x    1 root     root       9167339 Nov  4 05:12 guestbook
/app # cat data/datafile.txt
2020/11/04 05:14:29 Logging guestbook data:
2020/11/04 05:20:14 One
2020/11/04 05:20:17 Two
2020/11/04 05:20:20 Three
2020/11/04 05:27:20 Logging guestbook data:
2020/11/04 05:32:04 Four
2020/11/04 05:32:07 Five
2020/11/04 05:32:08 Six
```

