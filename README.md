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
Please take a note of the repositoryUri and registryId, in the case above is **123456789000.dkr.ecr.eu-west-1.amazonaws.com/workshop-mls** and **123456789000**.

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
To create the task definition, first lets edit the file simple-task.json and replace the image value with our repositoryUri:
```shel
{
  "containerDefinitions": [
    {
      "name": "nginx",
      "image": "<repositoryUri>",
...
```
Once this is complete, we can create the task definition with the following command:
```shell
aws ecs register-task-definition --family workspace-<name>  --cli-input-json file://simple-task.json
```
>Replace <name> with your name.

This will produce an output similar to this one:
```shell
{
    "taskDefinition": {
        "taskDefinitionArn": "arn:aws:ecs:eu-west-1:123456789000:task-definition/workspace-mls:1",
        "containerDefinitions": [
            {
                "name": "nginx",
                "image": "123456789000.dkr.ecr.eu-west-1.amazonaws.com/workshop-mls",
                "cpu": 0,
                "memory": 128,
                "portMappings": [
                    {
                        "containerPort": 80,
                        "hostPort": 0,
                        "protocol": "tcp"
                    }
                ],
                "essential": true,
                "environment": [],
                "mountPoints": [],
                "volumesFrom": []
            }
        ],
        "family": "workspace-mls",
        "revision": 1,
        "volumes": [],
        "status": "ACTIVE",
        "placementConstraints": [],
        "compatibilities": [
            "EC2"
        ]
    }
}
```
Please take a note of the taskDefinitionArn, in the case above is **arn:aws:ecs:eu-west-1:123456789000:task-definition/workspace-mls:1**.

Now, lets run the task definition as a simple task in our cluster:
```shell
aws ecs run-task --cluster ecs-workshop --count 1 --task-definition <taskDefinitionArn>
```
Now we can access the ECS cluster and check our task under the tasks tab by typing your name in the search bar. When you access the task information page, you can expand the container to view which host port the container is mapped and after enable access to that port, you can try to access the ip:port provided.
### Add variables
With this run, lets check the example with a environment variable.

This time, lets edit the file withvars-task.json, once again replace the image value with our repositoryUri and replace the value for the variable SMWORKSHOP with your name:
```shell
{
  "containerDefinitions": [
    {
      "name": "nginx",
      "image": "<repositoryUri>",
      "cpu": 0,
      "memory": 128,
      "portMappings": [
        {
          "containerPort": 80,
          "hostPort": 0,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "environment": [
        {
          "name": "SMWORKSHOP",
          "value": "<name>"
        }
      ]
    }
  ]
}
```
Then we can register our new task, get the new taskDefinitionArn and execute the run-task command again with the new file:
```shell
aws ecs register-task-definition --family workspace-<name>  --cli-input-json file://simple-task.json
```
> Note that the task arn have a number increment

Execute the run-task using the new revision.
```shell
aws ecs run-task --cluster ecs-workshop --count 1 --task-definition <taskDefinitionArn>
```
This time we will be able to see a new task and in its information page we can see our environment variable 
## Create a Service
Now lets create an ecs service to manage our task lifecycle.

First step, lets create the task:
```shell
aws ecs create-service --cluster ecs-workshop --desired-count 1 --service-name workspace-<name> --task-definition <taskDefinitionArn>  
```
Here we can check and discuss the main differences from scheduling the container using Service x stand alone task.
### Attach to LB
In order to expose our services to the internet, we need to recreate them, but this time we will use a load balancer and direct traffic based on host-header.

First we need to create target group for our service:
```shell
aws elbv2 create-target-group --name workshop-<name>  --protocol HTTP --port 80 --target-type instance --health-check-interval-seconds 10 --health-check-timeout-seconds 5 --healthy-threshold-count 2 --vpc-id <vpc_id>
```
> *vpc_id* will be provided by the host


This will generate an output like the following:
```shell
{
    "TargetGroups": [
        {
            "TargetGroupArn": "arn:aws:elasticloadbalancing:eu-west-1:123456789000:targetgroup/workshop-mls2/15a9e0c1b1414df9",
            "TargetGroupName": "workshop-mls",
            "Protocol": "HTTP",
            "Port": 80,
            "VpcId": "vpc-123456789000",
            "HealthCheckProtocol": "HTTP",
            "HealthCheckPort": "traffic-port",
            "HealthCheckEnabled": true,
            "HealthCheckIntervalSeconds": 10,
            "HealthCheckTimeoutSeconds": 5,
            "HealthyThresholdCount": 2,
            "UnhealthyThresholdCount": 2,
            "HealthCheckPath": "/",
            "Matcher": {
                "HttpCode": "200"
            },
            "TargetType": "instance"
        }
    ]
}
```
> Keep note of the *TargetGroupArn*

Now lets create your listener rule which tells the Load balancer to forward traffic to the new target group. First lets edit the conditions-host.json and replace <REPLACE> with your name:
```shell
[
    {
        "Field": "host-header",
        "HostHeaderConfig": {
            "Values": ["<REPLACE>.ecs.src.hm"]
        }
    }
]
```
Then we can create the listener rule by running the command below:
```shell
aws elbv2 create-rule --listener-arn <listener_arn> --priority <priority> --conditions file://conditions-host.json --actions Type=forward,TargetGroupArn=<TargetGroupArn>
```
> *listener_arn* will be provided by the host
> *priority* should be different among the participants
> Replace <TargetGroupArn> with the one previously created

Once you have prepared the target group and the listener rule, we can recreate our service. First, lets delete the previous service:
```shell
aws ecs delete-service --cluster ecs-workshop --force --service-name workspace-<name>
```
Now lets recreate the service with loadbalancer support:
```shell
aws ecs create-service --cluster ecs-workshop --desired-count 1 --service-name workspace-<name> --task-definition <taskDefinitionArn> --load-balancers targetGroupArn=<TargetGroupArn>,loadBalancerName=<lb_name>,containerName=nginx,containerPort=80
```
> *lb_name* will be provided by the host
> Replace <TargetGroupArn> with the one previously created

Now you can access your service at http://name.ecs.src.hm
## Access the instance
## See logs
## Send logs to Datadog
