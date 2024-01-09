# Automating AWS Infrastructure with IaC using Terraform

## Prerequisites before you begin writing Terraform code
* Create an IAM user, name it `terraform` _(ensure that the user has only programatic access to your AWWS account)_ and grant this user `AdministratorAccess` permissions.
* Copy the **secret access key** and **acces key ID** and save them in a notepad temporarily.
* Configure programmatic access from your workstation to connect to AWS using thr access keys copied above and a [Python SDK (boto3)](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). Ensure you have [Python 3.6](https://www.python.org/downloads/) or higher on your workstation.


```sh
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)
```

## VPC | Subnets | Security Groups

Open your Visual Studio Code and:
* Create a folder called `PBL`
* Create a file in the folder and name it `main.tf`

The setup should look like the image shown below:

_Set up Terraform CLI as per [this instruction](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)_.

### Provider & VPC Resource Section
* Add `AWS` as a provider and a resource to create a VPC in the `main.tf` file. _(The **Provider** block informs Terraform that we intend to build infrastructure within AWS and the **Resource** block will create a VPC)_.

```sh
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
}
```

**Note**: You can change the configuration above to create your VPC in a region that is closer to you.

* Run the `terraform init` command to download the necessary plugins for Terraform to work. _(These plugins are used by `providers` and `provisioners`.)_ 

**Note**: There is a new directory created called `.terraform\...`, this is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means you must execute `terraform init` again to download them.

* Let us create the only resource we just defined `aws_vpc`. But before we do that, run `terraform plan` command to check what Terraform intends to create before we tell it to go ahead and create it.

* Run the `terraform apply` command if you are satisfied with the planned changes.


The following observations were made after executing the `terraform apply` command:

1. A new file is created `terraform.tfstate`. This is how terraform keeps itself up to date witht he exact state of the infrastructure. It reads this file to know what already exists, what should be added or destroyed based on the entire terraform code that is being developed.

2. If you also observed closely, you would realise that another file `terraform.tfstate.lock.info` gets created during planning and apply. But this file gets deleted immediately. This is what Terraform uses to track who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same. It prevents duplicates and conflicts.