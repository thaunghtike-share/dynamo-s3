<h2>Prerequite</h2>

Run the stack
```bash
aws  cloudformation create-stack --stack-name test --template-body file://datapipeline.yaml --capabilities CAPABILITY_NAMED_IAM
```
```bash
You have to change unique bucket name here.

<a href="href="https://github.com/thaunghtike-share/dynamo-s3/blob/main/datapipeline.yaml#L5">href="https://github.com/thaunghtike-share/dynamo-s3/blob/main/datapipeline.yaml#L5</a> 
<a href="href="https://github.com/thaunghtike-share/dynamo-s3/blob/main/datapipeline.yaml#L61">href="https://github.com/thaunghtike-share/dynamo-s3/blob/main/datapipeline.yaml#L61</a> 
```

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
```

<h2>Data Pipeline Role Permissions Policy</h2>

The example policy that follows is scoped to allow essential functions that AWS Data Pipeline requires to run a pipeline with Amazon EC2 and Amazon EMR resources. It also provides permissions to access other AWS resources, such as Amazon Simple Storage Service and Amazon Simple Notification Service, that many pipelines require.


```bash
MyTestRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service:
                - elasticmapreduce.amazonaws.com
                - datapipeline.amazonaws.com
            Action: sts:AssumeRole
      Description: Role to provide aws data pipeline
      Policies:
        - PolicyName: MyDataPipelineRolePolicy1
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - ec2:AuthorizeSecurityGroupEgress
                  - ec2:AuthorizeSecurityGroupIngress
                  - ec2:CancelSpotInstanceRequests
                  - ec2:CreateNetworkInterface
                  - ec2:CreateSecurityGroup
                  - ec2:CreateTags
                  - ec2:DeleteNetworkInterface
                  - ec2:DeleteSecurityGroup
                  - ec2:DeleteTags
                  - ec2:DescribeAvailabilityZones
                  - ec2:DescribeAccountAttributes
                  - ec2:DescribeDhcpOptions
                  - ec2:DescribeImages
                  - ec2:DescribeInstanceStatus
                  - ec2:DescribeInstances
                  - ec2:DescribeKeyPairs
                  - ec2:DescribeLaunchTemplates
                  - ec2:DescribeNetworkAcls
                  - ec2:DescribeNetworkInterfaces
                  - ec2:DescribePrefixLists
                  - ec2:DescribeRouteTables
                  - ec2:DescribeSecurityGroups
                  - ec2:DescribeSpotInstanceRequests
                  - ec2:DescribeSpotPriceHistory
                  - ec2:DescribeSubnets
                  - ec2:DescribeTags
                  - ec2:DescribeVpcAttribute
                  - ec2:DescribeVpcEndpoints
                  - ec2:DescribeVpcEndpointServices
                  - ec2:DescribeVpcs
                  - ec2:DetachNetworkInterface
                  - ec2:ModifyImageAttribute
                  - ec2:ModifyInstanceAttribute
                  - ec2:RequestSpotInstances
                  - ec2:RevokeSecurityGroupEgress
                  - ec2:RunInstances
                  - ec2:TerminateInstances
                  - ec2:DescribeVolumeStatus
                  - ec2:DescribeVolumes
                  - elasticmapreduce:TerminateJobFlows
                  - elasticmapreduce:ListSteps
                  - elasticmapreduce:ListClusters
                  - elasticmapreduce:RunJobFlow
                  - elasticmapreduce:DescribeCluster
                  - elasticmapreduce:AddTags
                  - elasticmapreduce:RemoveTags
                  - elasticmapreduce:ListInstanceGroups
                  - elasticmapreduce:ModifyInstanceGroups
                  - elasticmapreduce:*
                  - elasticmapreduce:DescribeStep
                  - elasticmapreduce:AddJobFlowSteps
                  - elasticmapreduce:ListInstances
                  - iam:ListInstanceProfiles
                  - redshift:DescribeClusters
                Resource:
                  - "*"
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonS3FullAccess
        - arn:aws:iam::aws:policy/IAMFullAccess
        - arn:aws:iam::aws:policy/AWSDataPipeline_PowerUser
        - arn:aws:iam::aws:policy/AWSDataPipeline_FullAccess
      RoleName: MyDataPipelineRole1
```
Here is the full IAM policies for data pipeline IAM role.

![s3](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-22%20at%2009.21.06.png)

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

Here is the full IAM policies for data pipeline IAM role.

![iam](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-22%20at%2009.22.55.png)

<h2> IAC for Data Pipeline </h2>

This is full IAC template for Datapipeline.

```yaml
Description: 'A CloudFormation template which shows how to provide multiple values
  to one StringValue when creating a DataPipeline definition, This template is entirely
  based on the example provided in the DataPipeline documentation here: http://docs.aws.amazon.com/datapipeline/latest/DeveloperGuide/dp-object-emrconfiguration.html
  - Written by Nishant Casey'
Resources:
  DynamoDBInputS3OutputHive:
    Type: AWS::DataPipeline::Pipeline
    Properties:
      Name: DynamoDBInputS3OutputHive
      Description: Pipeline to backup DynamoDB data to S3
      Activate: 'true'
      ParameterObjects:
      - Id: "myDDBReadThroughputRatio"
        Attributes:
          - Key: "description"
            StringValue: "DynamoDB read throughput ratio"
          - Key: "type"
            StringValue: "Double"
          - Key: "default"
            StringValue: "0.2"
      - Id: "myOutputS3Loc"
        Attributes:
          - Key: "description"
            StringValue: "S3 output bucket"
          - Key: "type"
            StringValue: "AWS::S3::ObjectKey"
          - Key: "default"
            StringValue: "s3://my-unique-bucket-111-test/data/"
      - Id: "myDDBTableName"
        Attributes:
          - Key: "description"
            StringValue: "DynamoDB Table Name "
          - Key: "type"
            StringValue: "String"
      - Id: "myDDBRegion"
        Attributes:
          - Key: "description"
            StringValue: "DynamoDB Table Region"
          - Key: "type"
            StringValue: "String"
          - Key: "default"
            StringValue: "ap-northeast-1"
      ParameterValues:
      - Id: "myDDBTableName"
        StringValue: "Employee"
      PipelineObjects:
      - Id: "S3BackupLocation"
        Name: "Copy data to this S3 location"
        Fields:
          - Key: "type"
            StringValue: "S3DataNode"
          - Key: "dataFormat"
            RefValue: "DDBExportFormat"
          - Key: "directoryPath"
            StringValue: "#{myOutputS3Loc}/#{format(@scheduledStartTime, 'YYYY-MM-dd-HH-mm-ss')}"
      - Id: "DDBSourceTable"
        Name: "DDBSourceTable"
        Fields:
          - Key: "tableName"
            StringValue: "#{myDDBTableName}"
          - Key: "type"
            StringValue: "DynamoDBDataNode"
          - Key: "dataFormat"
            RefValue: "DDBExportFormat"
          - Key: "readThroughputPercent"
            StringValue: "#{myDDBReadThroughputRatio}"
      - Id: "DDBExportFormat"
        Name: "DDBExportFormat"
        Fields:
          - Key: "type"
            StringValue: "DynamoDBExportDataFormat"
      - Id: "TableBackupActivity"
        Name: "TableBackupActivity"
        Fields:
          - Key: "resizeClusterBeforeRunning"
            StringValue: "true"
          - Key: "type"
            StringValue: "HiveCopyActivity"
          - Key: "input"
            RefValue: "DDBSourceTable"
          - Key: "runsOn"
            RefValue: "EmrClusterForBackup"
          - Key: "output"
            RefValue: "S3BackupLocation"
      - Id: "DefaultSchedule"
        Name: "RunOnce"
        Fields:
          - Key: "occurrences"
            StringValue: "1"
          - Key: "startDateTime" 
            StringValue: "2023-11-21T02:00:00"
          - Key: "type"
            StringValue: "Schedule"
          - Key: "period"
            StringValue: "1 Day"
      - Id: "Default"
        Name: "Default"
        Fields:
          - Key: "type"
            StringValue: "Default"
          - Key: "scheduleType"
            StringValue: "cron"
          - Key: "failureAndRerunMode"
            StringValue: "CASCADE"
          - Key: "role"
            StringValue: "MyDataPipelineRole"
          - Key: "resourceRole"
            StringValue: "aws-elasticbeanstalk-ec2-role"
          - Key: "schedule"
            RefValue: "DefaultSchedule"
      - Id: "EmrClusterForBackup"
        Name: "EmrClusterForBackup"
        Fields:
          - Key: "terminateAfter"
            StringValue: "2 Hours"
          - Key: "amiVersion"
            StringValue: "3.3.2"
          - Key: "masterInstanceType"
            StringValue: "m1.medium"
          - Key: "coreInstanceType"
            StringValue: "m1.medium"
          - Key: "coreInstanceCount"
            StringValue: "1"
          - Key: "type"
            StringValue: "EmrCluster"
```

Here is the visual representation of the pipeline definition.

![visual](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-21%20at%2021.27.31.png)

Once it reached 2:OOAM, the status changed to “Waiting for Runner” and job should bescheduled

![finish](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-22%20at%2009.11.51.png)

Go to s3 bucket. You will see one exported folder.

![s3](https://raw.githubusercontent.com/thaunghtike-share/dynamo-s3/main/images/Screen%20Shot%202022-11-22%20at%2009.11.25.png)

