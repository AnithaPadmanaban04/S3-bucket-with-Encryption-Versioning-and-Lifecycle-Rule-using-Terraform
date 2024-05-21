# Create-S3-bucket-with-Encryption-Versioning-enabled-and-Lifecycle Configuration

## Objective:

The goal of this project is to create an Amazon S3 bucket with server-side encryption and versioning enabled using Terraform. This ensures secure storage of data with automatic data version control, allowing for easy recovery of earlier versions in case of accidental deletion or corruption.

## Required Skills and Tools:

* Terraform and IaC experience.
  
* Basic knowledge of AWS services, specifically Amazon S3.
  
* Understanding of security best practices for AWS S3.
  
* Ability to implement and test infrastructure changes using Terraform.

## Step 1: Define [provider.tf](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/blob/main/provider.tf) file

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.48.0"
    }
  }
}

provider "aws" {
  # Configuration options
  region = var.aws_region
}
```

## Step 2: [Create S3 bucket](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/blob/main/main.tf)

```hcl
# Create a S3 bucket 
resource "aws_s3_bucket" "my-bucket" {
  bucket = var.bucket-name
}
```

Using the resource block, we've defined a new resource of type **aws_s3_bucket**. This tells Terraform that we want to create a new S3 bucket in our AWS account.

Inside the aws_s3_bucket block, we've specified the name of our bucket using the bucket field. The name is reference from the variable file

**[variable.tf](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/blob/main/variable.tf)** 

```hcl
# declare a variable to define the region name
variable "aws_region" {
    description = "Mention the region name"
    type = string
    default = "us-east-1"
}

# declare a variable to define the name of the S3 bucket
variable "bucket-name" {
    description = "S3 name defined uniquely"
    type = string
    default = "mybucket-terraform2024"
  
}
```

The basic configuration to create a bucket is done. Run the ```terraform init``` command to initialize the working directory and download the required providers.

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/3f783a94-8881-4f76-9a7a-fe56e745153e)

Now, use ```terraform plan``` which creates an execution plan by analyzing the changes required to achieve the desired state of your infrastructure

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/ea804e5e-4295-43e3-b0bc-20b83b748b64)

Finally, use ```terraform apply ``` to apply the changes to create or update resources.

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/5347f250-a784-4215-9efb-4d8030d0401b)

S3 bucket successfully created in AWS Console

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/b60c51be-3e53-4ad5-8a61-85c2969f7f01)

## Step 3: Configure the bucket to allow public read access 

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/1e28fbd2-2c3f-459a-b123-67a9ba0fa95a)

This example explicitly disables the default S3 bucket security settings. This should be done with caution, as all bucket objects become publicly exposed.

```hcl
resource "aws_s3_bucket_ownership_controls" "example" {
  bucket = aws_s3_bucket.my-bucket.id
  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

resource "aws_s3_bucket_public_access_block" "example" {
  bucket = aws_s3_bucket.my-bucket.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_acl" "example" {
  depends_on = [
    aws_s3_bucket_ownership_controls.example,
    aws_s3_bucket_public_access_block.example,
  ]

  bucket = aws_s3_bucket.my-bucket.id
  acl    = "public-read"
}
```

Apply the changes to check the output in console

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/955a5a2c-5a17-4cce-ab01-91e33003b338)

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/3c70af7d-fa30-43d7-9ee2-36bfa1132f78)

## Step 4: Create an S3 bucket policy that allows read-only access to a specific IAM user or role

create a file **[iampolicy.tf](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/blob/main/iampolicy.tf)** and add the code below

```hcl
resource "aws_s3_bucket_policy" "allow_access_from_another_account" {
  bucket = aws_s3_bucket.my-bucket.id
  policy = data.aws_iam_policy_document.allow_access_from_another_account.json
}

data "aws_iam_policy_document" "allow_access_from_another_account" {
  statement {
    principals {
      type        = "AWS"
      identifiers = ["123456789012"] # add your AWS account number
    }

    actions = [
      "s3:GetObject",
      "s3:ListBucket",
    ]

    resources = [
      aws_s3_bucket.my-bucket.arn,
      "${aws_s3_bucket.my-bucket.arn}/*",
    ]
  }
}
```
Bucket Policy added

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/643f5da4-e014-453f-8a45-4bba62afb398)


## Step 5: Upload single file into S3 bucket

The below code uploads single file into the bucket. For example we have uploaded **[index.html](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/blob/main/index.html)** file 

```hcl
# Uploading index.html file to our bucket
resource "aws_s3_object" "index" {
  bucket = aws_s3_bucket.my-bucket.id
  key = "index.html"
  source = "index.html"
  acl = "public-read"
  content_type = "text/html"
}
```

run ```terraform apply -auto-approve``` to check the output

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/73f1e610-088a-4f29-8143-edcc6679cd05)

Check the AWS Console

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/dd699d5a-4bf8-49e0-9d51-dff6d6ee65c8)

## Step 6: Upload entire project folder into S3 bucket  (upload multiple files at a time)

Copy the folder into your local working directory

```hcl
# Using null resource to push all the files in one time instead of sending one by one
resource "null_resource" "upload-to-S3" {
  provisioner "local-exec" {
    command = "aws s3 sync ${path.module}/2109_the_card s3://${aws_s3_bucket.my-bucket.id}"
  }
}
```
![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/91c846b4-1258-417a-8d34-e7fac7992843)


## Step 7: Enable versioning on the S3 bucket **Resource: aws_s3_bucket_versioning**

Provides a resource for controlling versioning on an S3 bucket. Deleting this resource will either suspend versioning on the associated S3 bucket or simply remove the resource from Terraform state if the associated S3 bucket is unversioned.

```hcl
# Enable Versioning in Bucket

resource "aws_s3_bucket_versioning" "versioning_example" {
  bucket = aws_s3_bucket.my-bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}
```

Versioning Enabled

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/36e1ce13-1028-41ef-a7ba-50f6224c355c)

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/7d527363-88aa-49ec-8359-6d006d7f248a)

## Step 8: aws_s3_bucket_server_side_encryption_configuration

```hcl
resource "aws_kms_key" "mykey" {
  description             = "This key is used to encrypt bucket objects"
  deletion_window_in_days = 10
}


resource "aws_s3_bucket_server_side_encryption_configuration" "example" {
  bucket = aws_s3_bucket.my-bucket.id

  rule {
    apply_server_side_encryption_by_default {
      kms_master_key_id = aws_kms_key.mykey.arn
      sse_algorithm     = "aws:kms"
    }
  }
}
```

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/0c504040-25dd-4134-9ae4-819e140ceb63)


## Step 9: Add Lifecycle Configuration

* **Lifecycle Configuration:** Users can define or specify lifecycle rules for their own S3 buckets through lifecycle configuration settings. These principles indicate moves to be made on 
    objects as they age.
* **Transition Actions:** Transition Actions figure out what happens to objects as they arrive at determined stages in their lifecycle. For instance, items can be consequently changed to lower- 
    cost capacity classes like S3 Standard-IA or Icy mass to diminish storage costs.
* **Expiration Actions:** Expiration Actions characterize when objects should to be consequently deleted from the bucket. Items can be designed to lapse following a specific number of days since 
    creation or since the objects last modification.

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/09b59b87-e46d-4000-99ae-55c949fa79a3)


 ```hcl
  # Define lifecycle policy
  resource "aws_s3_bucket_lifecycle_configuration" "example_lifecycle" {
  bucket = aws_s3_bucket.my-bucket.id

  rule {
    id = "rule1"
    filter {
      prefix = "logs/" # You can set your prefix here
    }

    # Transition rule
    transition {
      days          = 30 # Update the transition days as per your requirement
      storage_class = "GLACIER"
    }

    # Expiration rule
    expiration {
      days = 60 # Update the expiration days as per your requirement
    }

    # Status
    status = "Enabled"
  }
}
```

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/be7b4550-db4d-41ce-810a-0d90e111dea2)


## Step 9: Clean the resources

Apply ```terraform destroy -auto-approve``` to clean all the resources created

When trying to delete a bucket with versioning enabled, you get the following error

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/176f4a88-41dc-4cdd-a2c0-74cb3f5bff31)

Fix this with the below code

```hcl
# Create a S3 bucket 

resource "aws_s3_bucket" "my-bucket" {
  bucket = var.bucket-name
  force_destroy = true
  lifecycle {
    prevent_destroy = false
  }
}
```

![image](https://github.com/AnithaPadmanaban04/S3-bucket-with-Encryption-Versioning-and-Lifecycle-Rule-using-Terraform/assets/170385807/128f5780-fd1a-4298-9717-3eb591af1cd6)

