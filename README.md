<h2>Run Cloudformation Stack</h2>

I used this cloudformation template to create aws resources - DynamoDB and S3 bucket.

```bash
AWSTemplateFormatVersion: "2010-09-09"
Description: Simple cloud formation for bucket creation and configuration

Parameters:
  BucketName: { Type: String, Default: "my-unique-bucket-111-test" }
  HashKeyElementName:
    Type: String
    Default: EmployeeId
    Description: Hash Key Name
  HashKeyElementType:
    Type: String
    Default: S
    Description: Hash Key Type

Resources:
  MainBucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: !Ref BucketName
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: AES256
  EmployeeTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Employee
      AttributeDefinitions:
        -
          AttributeName: !Ref HashKeyElementName
          AttributeType: !Ref HashKeyElementType
      KeySchema:
        -
          AttributeName: !Ref HashKeyElementName
          KeyType: HASH
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5

Outputs:
  MainBucketName:
    Description: Name of the main bucket
    Value: !Ref MainBucket
  Employee:
    Description: Table Created using this template.
    Value: !Ref EmployeeTable

```

Github action is used to run this template file. Some Env Variables are required to run github actions. Eg. ACCESS_KEY, ACCESS_KEY_SECRET, AWS_REGION

```bash
name: 'Deploy to AWS CloudFormation'

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code from master branch
        uses: actions/checkout@v2

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.ACCESS_KEY_SECRET }}
          aws-region: ${{ secrets.AWS_REGION }}

      - name: Deploy to AWS CloudFormation
        uses: aws-actions/aws-cloudformation-github-deploy@v1
        with:
          name: github
          template: s3-dynamodb.yaml
          no-fail-on-empty-changeset: "1"
```

<h2>Export Dynamodb To S3 </h2>

Let us now visit our AWS Console and come to Data Pipeline service. Create a new pipeline. Choose export DynamoDB template from the drop down.

![template](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-21%20at%2021.06.08.png)

Mention the output S3 bucket we have created and the DynamoDB table.

![bucket](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-21%20at%2021.06.45.png)

Schedule the pipeline job at 2AM everyday 

![cron](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-21%20at%2021.08.02.png)

Attach the data pipeline role and resource pipeline role.

![role](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-21%20at%2021.11.29.png)

The contents of the AmazonEC2RoleforDataPipelineRole is shown below. This is the managed policy attached to the default resource role for AWS Data Pipeline, DataPipelineDefaultResourceRole. When you define a resource role for your pipeline, we recommend that you begin with this permissions policy and then remove permissions for AWS service actions that are not required.

Version 3 of the policy is shown, which is the most recent version at the time of this writing. View the most recent version of the policy using the IAM console.

```bash
{
  "Version": "2012-10-17",
  "Statement": [{
      "Effect": "Allow",
      "Action": [
        "cloudwatch:*",
        "datapipeline:*",
        "dynamodb:*",
        "ec2:Describe*",
        "elasticmapreduce:AddJobFlowSteps",
        "elasticmapreduce:Describe*",
        "elasticmapreduce:ListInstance*",
        "elasticmapreduce:ModifyInstanceGroups",
        "rds:Describe*",
        "redshift:DescribeClusters",
        "redshift:DescribeClusterSecurityGroups",
        "s3:*",
        "sdb:*",
        "sns:*",
        "sqs:*"
      ],
      "Resource": ["*"]
    }]
}
```

Here is the visual representation of the pipeline definition.

![visual](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-21%20at%2021.27.31.png)

Once it reached 2:OOAM, the status changed to “Waiting for Runner” and job should bescheduled

![finish](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-22%20at%2009.11.51.png)

Go to s3 bucket. You will see one exported folder.

![s3](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-22%20at%2009.11.25.png)




