# 1. Setup

Execute the following steps:

1. Sign up for IBM Cloud](#1-sign-up-for-ibm-cloud), go [here](https://ibm.github.io/workshop-setup/NEWACCOUNT/)
2. Setup Client CLI using CognitiveLabs, go [here](https://ibm.github.io/workshop-setup/COGNITIVECLASS/), or using IBM Cloud Shell, go [here](https://ibm.github.io/workshop-setup/CLOUDSHELL/),
3. Connect to an OpenShift Cluster, go [here](https://ibm.github.io/workshop-setup/ROKS/)
4. [Setup Environment Variables](#setup-environment-variables)

## Setup Environment Variables

Create an environment variable for your IBM ID,

```console
IBM_ID=<your ibm id>
```

If completed, in your [terminal](https://labs.cognitiveclass.ai/), create a working directory named `cos-with-s3fs` to start the lab,

```console
NAMESPACE=cos-with-s3fs-lab
mkdir $NAMESPACE
cd $NAMESPACE

export WORKDIR=$(pwd)
echo $WORKDIR
```

In the CognitiveLabs terminal this should output the following directory `/home/project/cos-with-s3fs-lab`.

## Next

[3. Create Object Storage Instance](../cos-with-s3fs/COS.md).

Optionally you can first read more about what Object Storage is [here](../cos-with-s3fs/ABOUT-COS.md).
