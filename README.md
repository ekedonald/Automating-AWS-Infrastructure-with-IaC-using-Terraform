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

* Let us create the only resource we just defined `aws_vpc`. But before we do that, run the `terraform plan` command to check what Terraform intends to create before we tell it to go ahead and create it.

* Run the `terraform apply` command if you are satisfied with the planned changes.


The following observations were made after executing the `terraform apply` command:

1. A new file is created `terraform.tfstate`. This is how terraform keeps itself up to date witht he exact state of the infrastructure. It reads this file to know what already exists, what should be added or destroyed based on the entire terraform code that is being developed.

2. If you also observed closely, you would realise that another file `terraform.tfstate.lock.info` gets created during planning and apply. But this file gets deleted immediately. This is what Terraform uses to track who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same. It prevents duplicates and conflicts.

Its content is usually in the format shown below:

It is a `json` format file that stores information about a user: user's `ID`, what operation he/she is doing, timestamp and location of the `state` file.

## Subnets Resource Section
According to our architectural design, we require 6 subnets:
* 2 Public Subnets
* 2 Private Subnets for Webservers
* 2 Private Subnets for Data Layer

### Creating the first 2 Public Subnets
* Copy and paste the configuration below to the `main.tf` file:

```sh
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"
}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}
```

_In order to create 2 subnets, we must declare 2 **resource blocks** (i.e. one for each subnet). The `vpc_id` argument is used to reference the value of the VPC `id` by setting it to `aws_vpc.main.id`. This way, Terraform knows inside which VPC to create the subnet._

* Run the `terraform plan` command.

* Run the `terraform apply` command.

The following observations were made after executing the `terraform apply` command:

1. **Hard Coded Values**: Rememeber our best practice hint from the beginning? Both the `availability_zone` and `cidr_block` arguments are hard coded. We should always endeavour to make our configurations dynamic and reusable.

2. **Multiple Resource Block**: Notice that we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resoure that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So we need to optimize this by introducing a `count` argument.

#### Fixing The Problems By Code Refactoring
The following steps are taken to improve our code by refactoring it:

Run the `terrafom destroy` command and type `yes` to destroy to the current infrastructure. _**Note:** Do not destroy an infrastructure that has been deployed to production._

**Fixing Hard Coded Values**: Variables are introduced to remove hard coding. 

* Starting with the provider block, declare a variable named `region`, give it a default value and update the provider section by referring to the declared variable.

```sh
    variable "region" {
        default = "us-east-1"
    }

    provider "aws" {
        region = var.region
    }
```

* Do the same to the `cidr` value in the `vpc` block and all the other arguments.

```sh
    variable "region" {
        default = "us-east-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.region
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    }
```

**Fixing Multiple Resource Blocks**: This is where concepts like **Loops & Data Sources** are introduced.

Terrafrom has a functionality that allows us to pull data which exposes information to us. For example, every region has Availability Zones (AZ). Different regions have from 2 t0 4 Availability Zones. With over 20 geographic regions and over 70 Availability Zones served by AWS, it is impossible to keep up with the latest information by hard coding the names of Availability Zones. Hence, we will explore the use of Terraforms's **Data Sources** to fetch the information outside of Terraform. In this case, from **AWS**.
