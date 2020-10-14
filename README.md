# ECS WORKSHOP
This workshop aims to provide an overview for basic ECS operation. For a full and more complete workshop, please check https://ecsworkshop.com/
## Requirements
AWS CLI: It can be installed here: https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html

Docker: https://docs.docker.com/get-docker/
## Credentials
Use aws-azure-login cli or the provided credentials
## Overview on ECS
Add docs
### Checkout the running cluster
Brief on ASG and Spot.io options
## Create ECR
To create an ECR repository we can run the following command:
```shel
aws ecr create-repository --repository-name workshop-<name>
```
>Replace <name> with your name.

This will produce an output similar to this one:
```shell
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:eu-west-1:123456789000:repository/workshop-mls",
        "registryId": "123456789000",
        "repositoryName": "workshop-mls",
        "repositoryUri": "123456789000.dkr.ecr.eu-west-1.amazonaws.com/workshop-mls",
        "createdAt": 1502712474.0,
        "imageTagMutability": "MUTABLE",
        "imageScanningConfiguration": {
            "scanOnPush": false
        }
    }
}
```
Please take a note of the repositoryUri and registryId, in the case above is **123456789000.dkr.ecr.eu-west-1.amazonaws.com/workshop-mls**.

### Push an image
Once we create the ECR repository, we can build an example image and push it to the new repository, but first, lets login to the AWS ECR:
```shell
aws ecr get-login-password --region eu-west-1 | docker login --username AWS --password-stdin <registryId>.dkr.ecr.eu-west-1.amazonaws.com
```
> Replace <registryId> with the newly created repository Id
After login, we can build the image:
```shell
docker build -t <repositoryUri> -f Dockerfile .
```
>Replace <repositoryUri> with the newly created repository URI.

Once the build is done, we can push the image:
```shell
docker push <repositoryUri>
```
>Replace <repositoryUri> with the newly created repository URI.

## Create a Task definition
### Add variables
## Create a Service
### Attach to LB
## Login to the instance
## See logs
## Send logs to Datadog
