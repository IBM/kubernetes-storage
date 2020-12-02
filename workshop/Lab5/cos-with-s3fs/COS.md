# Setup Object Storage

In this section, you will create an instance of IBM Cloud Object Storage (COS), create credentials and a bucket to store your persistent data for MongoDB.

Steps:
* Preparation
* Create an Object Storage Instance
* Add Credentials
* Create a Bucket
* Get Private Endpoint

## Preparation

1. Set the following environment variables:

    ```
    RESOURCEGROUP=Default
    COS_NAME_RANDOM=$(date | md5sum | head -c10)
    COS_NAME=$COS_NAME_RANDOM-cos-1
    COS_CREDENTIALS=$COS_NAME-credentials
    COS_PLAN=Lite
    COS_BUCKET_NAME=$(date | md5sum | head -c10)-bucket-1
    REGION=us-south
    COS_ENDPOINT=s3.private.$REGION.cloud-object-storage.appdomain.cloud
    ```

2. Create an instance of the `IBM Cloud Object Storage` service. For information, go to https://cloud.ibm.com/catalog/services/cloud-object-storage. You can only have 1 single free Lite instance per account. Login to your **personal account**,

    ```
    ibmcloud login -u $IBM_ID
    ```

    **Note:** if you use a single-sign-on provider, use the `-sso` flag.

3. You will be prompted to select an account. You must choose your own account under your own name. In the example below, account 1 is my own account under my own name, account 2 is where my Kubernetes cluster is located and that was provisioned to me, but on that second account I have no permission to create new resources. I have to select account **1** under my own name, e.g. `B Newell's Account',

    ```
    Select an account:
      1. B Newell's Account (31296e3a285f)
      2. IBM Client Developer Advocacy (e65910fa61) <-> 1234567
      Enter a number> **1**
      Targeted account B Newell's Account (31296e3a285f)
    ```

4. You also need a resource group. Check if a resource-group exists,

    ```
    ibmcloud resource groups
    ```

    outputs,

    ```
    ibmcloud resource groups
    OK
    Name    ID    Default Group   State
    Default   282d2f25256540499cf99b43b34025bf   true    ACTIVE
    ```

    If you have an existing resource group that is different than the default value `Default`, change the environment variable `$RESOURCEGROUP`. For example, if you have an existing resource group called `default` with lowercase `d`, change the environment variable,

    ```
    RESOURCEGROUP=default
    ```

    or use the following command to set the environment variable automatically,

    ```
    RESOURCEGROUP=$(ibmcloud resource groups --output json | jq -r '.[0].name')
    echo $RESOURCEGROUP
    ```

    If you do not have a resource group, create one,

    ```
    ibmcloud resource group-create $RESOURCEGROUP
    ```

    outputs,

    ```
    $ ibmcloud resource group-create $RESOURCEGROUP
    Creating resource group Default under account 5081ea1988f14a66a3ddf9d7fb3c6b29 as remko@remkoh.dev...
    OK
    Resource group Default was created.
    Resource Group ID: 93f7a4cd3c824c0cbe90d8f21b46f758
    ```

1. Set the environment variable $RESOURCEGROUP,

    ```
    RESOURCEGROUP=$(ibmcloud resource groups --output json | jq -r '.[0].name')
    echo $RESOURCEGROUP
    ```

## Create an Object Storage Instance

The Lite service plan for Cloud Object Storage includes Regional and Cross Regional resiliency, flexible data classes, and built in security. For the sample application, I will choose the `standard` and `regional` options in the `ibmc-s3fs-standard-regional` storageclass that is typical for web or mobile apps and we don't need cross-regional resilience beyond resilience per zones for our workshop app, but the options to choose for usage strategies and therefor the pricing of storageclasses for the bucket is very granular.

1. Create a new Object Storage instance via CLI command, for the lab you can use a `Lite` plan. If you prefer a paid plan, choose `Standard` plan.

    ```
    ibmcloud resource service-instance-create $COS_NAME cloud-object-storage $COS_PLAN global -g $RESOURCEGROUP
    ```

    For example, outputs,

    ```
    $ ibmcloud resource service-instance-create cef84ff5ff-cos-1 cloud-object-storage Lite global -g Default

    OK
    Service instance cef84ff5ff-cos-1  was created.                 
    Name:             cef84ff5ff-cos-1    
    ID:               crn:v1:bluemix:public:cloud-object-storage:global:a/          
                        e65910fa61ce9072d64902d03f3d4774:fef2d369-5f88-4dcc-bbf1-9afffcd9ccc7::   
    GUID:             fef2d369-5f88-4dcc-bbf1-9afffcd9ccc7   
    Location:         global   
    State:            active   
    Type:             service_instance   
    Sub Type:            
    Allow Cleanup:    false   
    Locked:           false   
    Created at:       2020-05-29T15:55:26Z   
    Updated at:       2020-05-29T15:55:26Z   
    Last Operation:                   
         Status    create succeeded      
         Message   Completed create instance operation   
    ```

1. List the object storage instance you created,

    ```
    ibmcloud resource service-instance $COS_NAME
    ```

1. Set the GUID of the object storage instance,

    ```
    COS_GUID=$(ibmcloud resource service-instance $COS_NAME --output json | jq -r '.[0].guid')
    echo $COS_GUID
    ```

## Add Credentials

1.  Now add credentials for authentication method IAM,

    ```
    ibmcloud resource service-key-create $COS_CREDENTIALS Writer --instance-name $COS_NAME --parameters '{"HMAC":true}'
    ```

    List the created credentials as json,

    ```
    COS_APIKEY=$(ibmcloud resource service-key $COS_CREDENTIALS --output json | jq -r '.[0].credentials.apikey')
    echo $COS_APIKEY
    ```

    **Congratulations**: you have now a free instance of IBM Cloud Object Storage. Next: create a bucket.

### Create a Bucket

Data in `IBM Cloud Object Storage` is stored and organized in so-called `buckets`. To create a new `bucket` in your `IBM Cloud Object Storage` service instance,

1. Retrieve the service instance id or `Cloud Resource Name (CRN)` from the credentials, and set the CRN on the Object Storage,

    ```
    COS_CRN=$(ibmcloud resource service-key $COS_CREDENTIALS --output json | jq -r '.[0].credentials.resource_instance_id')
    echo $COS_CRN
    ```

1. Check the Object Storage configuration,

    ```
    ibmcloud cos config list
    ```

    Review the CRN property. 
    
    ```
    $ ibmcloud cos config list
    Key                     Value   
    Last Updated               
    Default Region          us-south   
    Download Location       /home/theia/Downloads   
    CRN                        
    AccessKeyID                
    SecretAccessKey            
    Authentication Method   IAM   
    URL Style               VHost   
    Service Endpoint    
    ```
    
    If the CRN is not set as in the example above, you can set it explicitly as follows,

    ```
    ibmcloud cos config crn --crn $COS_CRN
    ```

2. Create a new bucket.

    ```
    ibmcloud cos bucket-create --bucket $COS_BUCKET_NAME --class Standard --ibm-service-instance-id $COS_CRN 
    ```

    outputs,

    ```
    ibmcloud cos bucket-create --bucket $COS_BUCKET_NAME --class Standard --ibm-service-instance-id $COS_CRN 

    OK
    Details about bucket 726ebedfcb-bucket-1:
    Region: us-south
    Class: Standard
    ```

3. Verify the new bucket was created successfully.

    ```
    ibmcloud cos list-buckets --ibm-service-instance-id $COS_CRN
    ```

## Get Private Endpoint

1. Get your object storage configurations with the CRN set,

    ```
    ibmcloud cos config list
    ```

    outputs,

    ```
    $ ibmcloud cos config list
    Key                     Value
    Last Updated            Saturday, November 07 2020 at 22:47:38
    Default Region          us-south
    Download Location       /home/theia/Downloads
    CRN                     crn:v1:bluemix:public:cloud-object-storage:global:a/31296e3a285f42fdadd51ce14beba65e:5260b21c-80eb-4d72-a019-d11112079e85::
    AccessKeyID
    SecretAccessKey
    Authentication Method   IAM
    URL Style               VHost
    Service Endpoint
    ```

    This will list your default region, e.g. `us-south` in the example above.

    To list your bucket's location use

    ```
    ibmcloud cos get-bucket-location --bucket $COS_BUCKET_NAME
    ```

    outputs,

    ```
    $ ibmcloud cos get-bucket-location --bucket $COS_BUCKET_NAME
    OK
    Details about bucket 726ebedfcb-bucket-1:
    Region: us-south
    Class: Standard
    ```

    or

    ```
    ibmcloud cos get-bucket-location --bucket $COS_BUCKET_NAME --output json
    {
    "LocationConstraint": "us-south-standard"
    }
    ```

1. Find the service default endpoint:

    ```
    ibmcloud cos config endpoint-url --list
    ```

    If the `ServiceEndpointURL` is empty as int he example below, you can find the service endpoint manually.

    ```
    $ ibmcloud cos config endpoint-url --list
    Key                  Value
    ServiceEndpointURL
    ```

    With your bucket's location, e.g. `us-south`, you can find your bucket's private endpoint here https://cloud.ibm.com/docs/cloud-object-storage?topic=cloud-object-storage-endpoints#advanced-endpoint-types, OR in the following steps you find it in your Cloud Object Storage's bucket configuration.

    If your region is `us-south` the private endpoint is `s3.private.us-south.cloud-object-storage.appdomain.cloud`. Set an environment variable `$REGION` with the found region, and construct the service endpoint as follows.

    ```
    REGION=us-south
    COS_ENDPOINT=s3.private.$REGION.cloud-object-storage.appdomain.cloud
    echo $COS_ENDPOINT
    ```

2. **In a browser**, you can verify the private endpoint for your region by navigating to https://cloud.ibm.com/resources.

   1. Expand the Storage section.
   2. Locate and select your IBM Cloud Object Storage service instance.
   3. In the left menu, select the `buckets` section Select your new `bucket` in the `Buckets` tab.
   4.  Select the `Configuration` tab under `Buckets` iin the left pane.

       ![](../.gitbook/images/cos-04.png)

   5.  Take note of the `Private` endpoint. It should match your environment variable $COS_ENDPOINT

## Next

[Configure your Kubernetes Cluster](CLUSTER.md)