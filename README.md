# synced-folder

Synchronizes a local folder to Amazon S3, Azure Blob Storage, or Google Cloud Storage.

Still very much 🚧 👷. 

## Usage 

```yaml
name: synced-folder-example-yaml
runtime: yaml
description: An example of using the synced-folder component in YAML.

resources:

  #
  # Amazon S3 example.
  #

  # Create an S3 bucket and configure it as a website.
  s3-bucket:
    type: aws:s3:Bucket
    properties:
      acl: public-read
      forceDestroy: true
      website:
        indexDocument: index.html
        errorDocument: error.html

  # Sync the contents of ./site to it, using Pulumi to manage each file as an aws.s3.BucketObject.
  synced-bucket-folder:
    type: synced-folder:index:S3BucketFolder
    properties:
      path: ./site
      bucketName: ${s3-bucket.bucket}
      acl: public-read

  #
  # Azure Blob Storage example.
  #

  # Create a resource group.
  resource-group:
    type: azure-native:resources:ResourceGroup

  # Create a storage account.
  storage:
    type: azure-native:storage:StorageAccount
    properties:
      resourceGroupName: ${resource-group.name}
      kind: StorageV2
      sku:
        name: Standard_LRS

  # Configure the storage account as a website.
  website:
    type: azure-native:storage:StorageAccountStaticWebsite
    properties:
      resourceGroupName: ${resource-group.name}
      accountName: ${storage.name}
      indexDocument: index.html
      error404Document: error.html

  # Sync the contents of ./site to the storage container, using the Azure CLI to manage the files outside of Pulumi.
  synced-azure-blob-folder:
    type: synced-folder:index:AzureBlobFolder
    properties:
      path: ./site
      resourceGroupName: ${resource-group.name}
      storageAccountName: ${storage.name}
      containerName: ${website.containerName}
      managedObjects: false

  #
  # Google Cloud Storage example.
  # 

  # Create a Google Cloud Storage bucket and configure it as a website.
  gcp-bucket:
    type: gcp:storage:Bucket
    properties:
      location: US
      forceDestroy: true
      website:
        mainPageSuffix: index.html
        notFoundPage: error.html

  # Configure the bucket for public access.
  gcp-bucket-iam-binding:
    type: gcp:storage:BucketIAMBinding
    properties: 
      bucket: ${gcp-bucket.name}
      role: roles/storage.objectViewer
      members: 
        - allUsers

  # Sync the contents of ./site to the bucket, using the Google Cloud CLI to manage the files outside of Pulumi.
  synced-google-cloud-folder:
    type: synced-folder:index:GoogleCloudFolder
    properties:
      path: ./site
      bucketName: ${gcp-bucket.name}
      managedObjects: false

outputs:
  s3_url: http://${s3-bucket.websiteEndpoint}
  azure_url: ${storage.primaryEndpoints.web}
  google_cloud_url: https://storage.googleapis.com/${gcp-bucket.name}/index.html
```
