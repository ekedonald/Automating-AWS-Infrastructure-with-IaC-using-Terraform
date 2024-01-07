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