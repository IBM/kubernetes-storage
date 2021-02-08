# Share Documents with Cloud Object Storage

1. Login to your IBM Cloud account,

```console
ibmcloud login -u <username>
```

If you are using Single Sign-On (SSO) use the `-sso` flag to log in.

1. Create an IAM APIKEY for the Cloud Object Storage service, e.g. with service name `remkohdev-cos1`. Download and save the `iam_apikey` by adding the `--file` flag,

```console
COS_NAME=<service-name>
IAM_APIKEY_NAME=$COS_NAME-apikey1
ibmcloud iam api-key-create $IAM_APIKEY_NAME --file $IAM_APIKEY_NAME.txt
```

1. Set the IAM apikey environment variable,

    ```console
    IAM_APIKEY=$(cat $IAM_APIKEY_NAME.txt | jq -r '.apikey')
    echo $IAM_APIKEY
    ```

1. To create an object storage instance with a Lite plan, you need a `resource group`. Check if you already have a resource-group

    ```console
    ibmcloud resource groups
    ```

    outputs,

    ```console
    ibmcloud resource groups
    OK
    Name      ID                                 Default Group   State
    Default   282d2f25256540499cf99b43b34025bf   true            ACTIVE
    ```

    If you do not have a resource group yet, create one,

    ```console
    ibmcloud resource group-create Default
    ```

1. Create a new Object Storage instance with a `Lite` plan. If you prefer a paid plan, choose `Standard` plan.

    Set environment variables,

    ```console
    COS_PLAN=Lite
    RESOURCEGROUP=Default
    ```

    Then create the Cloud Object instance,

    ```console
    ibmcloud resource service-instance-create $COS_NAME cloud-object-storage $COS_PLAN global -g $RESOURCEGROUP
    ```

1. Get the GUID for the Cloud Object Storage service,

    ```console
    COS_GUID=$(ibmcloud resource service-instance $COS_NAME --output json | jq -r '.[0].guid')
    echo $COS_GUID
    ```

1. Create new service credentials with `Role: Reader` to share read-only access, and another service credentials with `Role: Writer` to upload documents,

    ```console
    COS_CREDENTIALS1=$COS_NAME-reader-credentials1
    COS_CREDENTIALS2=$COS_NAME-writer-credentials2
    ibmcloud resource service-key-create $COS_CREDENTIALS1 Reader --instance-name $COS_NAME
    ibmcloud resource service-key-create $COS_CREDENTIALS2 Writer --instance-name $COS_NAME
    ```

1. Create environment variables for the apikeys,

    ```console
    COS_READER_APIKEY=$(ibmcloud resource service-key $COS_CREDENTIALS1 --output json | jq -r '.[0].credentials.apikey')
    echo $COS_READER_APIKEY
    COS_WRITER_APIKEY=$(ibmcloud resource service-key $COS_CREDENTIALS2 --output json | jq -r '.[0].credentials.apikey')
    echo $COS_WRITER_APIKEY
    ```

1. Create a new bucket with a `Standard` storage class,

    ```console
    COS_BUCKET=$COS_NAME-bucket1
    COS_STORAGECLASS=Standard
    ibmcloud cos create-bucket --bucket $COS_BUCKET --ibm-service-instance-id $COS_GUID --class $COS_STORAGECLASS
    ```

1. Verify the new bucket was created successfully.

    ```console
    ibmcloud cos list-buckets --ibm-service-instance-id $COS_GUID
    ```

1. Retrieve the region of your object storage configuration,

    ```console
    ibmcloud cos config region list
    ```

    Or list your bucket's `LocationRestraint'

    ```console
    ibmcloud cos get-bucket-location --bucket $COS_BUCKET --json | jq -r '.LocationConstraint'
    ```

1. Set the environment variable for region, e.g. `us-south`,

    ```console
    COS_REGION=<region>
    ```

1. Create a new document,

    ```console
    COS_OBJECT_KEY=helloworld.txt
    echo "Hello World! Today is $(date)" > $COS_OBJECT_KEY
    ```

1. Upload a document using the S3Manager,

    ```console
    ibmcloud cos upload --bucket $COS_BUCKET --key $COS_OBJECT_KEY --file ./helloworld.txt --content-language en-US --content-type "text/plain"

    OK
    Successfully uploaded object 'helloworld.txt' to bucket 'e59a327194-cos-1-bucket1'.
    ```

1. Get IAM Token using the IAM Apikey:

    ```console
    curl --location --request POST "https://iam.cloud.ibm.com/identity/token" --header "Accept: application/json" --header "Content-Type: application/x-www-form-urlencoded" --header "apikey: $COS_READER_APIKEY" --data-urlencode "apikey=$IAM_APIKEY" --data-urlencode "response_type=cloud_iam" --data-urlencode "grant_type=urn:ibm:params:oauth:grant-type:apikey"
    ```

1. Set the response <access_token>

    ```console
    ACCESS_TOKEN=<access_token>
    ```

    Or using the curl statement above,

    ```console
    ACCESS_TOKEN=$(curl --location --request POST "https://iam.cloud.ibm.com/identity/token" --header "Accept: application/json" --header "Content-Type: application/x-www-form-urlencoded" --header "apikey: $COS_READER_APIKEY" --data-urlencode "apikey=$IAM_APIKEY" --data-urlencode "response_type=cloud_iam" --data-urlencode "grant_type=urn:ibm:params:oauth:grant-type:apikey" | jq -r '.access_token')
    echo $ACCESS_TOKEN
    ```

1. get bucket: <cos_bucket_name>

    ```console
    curl --location --request GET "https://s3.$COS_REGION.cloud-object-storage.appdomain.cloud/$COS_BUCKET" --header "Authorization: Bearer $ACCESS_TOKEN" --header "Accept: application/json"
    ```

1. Get object key: <cos_object_key>

    ```console
    ibmcloud cos list-objects --bucket $COS_BUCKET
    ```

    ```console
    curl --location --request GET "https://s3.$COS_REGION.cloud-object-storage.appdomain.cloud/$COS_BUCKET/$COS_OBJECT_KEY" --header "Authorization: Bearer $ACCESS_TOKEN"
    ```

1. Get document
