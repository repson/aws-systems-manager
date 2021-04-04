# Creating an SSM IAM Role and Configuring an EC2 Instance with AWS Systems Manager via the CLI

## Introduction

In this hands-on lab, we'll be dissecting the IAM role required by an EC2 instance to be able to communicate with the Systems Manager service. We'll first locate the managed AWS policy required for this role and create an EC2 instance via the command line, assigning it the instance profile (container for role assigned). Finally, we'll verify that Systems Manager (SSM) can detect the instance and communicate with it.

> Note: This lab is focused on the AWS command line and all technical steps will be shown via the CLI.

## Solution

Open a terminal session and log in to the cloud server via SSH using the provided lab credentials:

``` 
ssh cloud_user@<PUBLIC_IP>
```

### Get the ARN of the SSM Policy for the IAM Role AmazonEC2RoleforSSM

1. Note the policy Arn and DefaultVersionId in the output from the following command:

``` 
aws iam list-policies --scope AWS --query "Policies[?PolicyName == 'AmazonEC2RoleforSSM']"
``` 

2. Optionally, you may also use the following command in addition to the value of DefaultVersionId from the above command to look at the JSON document of the policy in question via the CLI:

```
aws iam get-policy-version --policy-arn <POLICY-ARN> --version-id <VERSION-ID-STRING-FROM-PREV-COMMAND>
```

### Create an IAM Role for EC2 and Assign the SSM Policy to It

1. Create a new JSON file in your current directory:

```
vim EC2Trust.json
```

2. To avoid adding unnecessary spaces and hashes, enter:

```
:set paste
```

3. Enter insert mode:

```
i
```

4. Enter the following JSON:

```
{
  "Version": "2012-10-17",
  "Statement": {
    "Effect": "Allow",
    "Principal": {"Service": "ec2.amazonaws.com"},
    "Action": "sts:AssumeRole"
  }
}
```

5. Save and quit the file by pressing Escape followed by wq!.

6. Create the role and attach the trust policy JSON file:

```
aws iam create-role --role-name MyEC2SSMRole --assume-role-policy-document file://EC2Trust.json
```

7. Assign the policy we created earlier to the newly created role:

```
aws iam attach-role-policy --role-name MyEC2SSMRole --policy-arn <SSM-POLICY-ARN>
```

### Create and Attach an Instance Profile to the IAM EC2 Service Role

1. Create an instance profile:

```
aws iam create-instance-profile --instance-profile-name MyEC2InstanceProfile
```

2. Copy the instance profile name and ARN in the output, as we'll use it when we create an EC2 instance.

3. Add the role we created earlier to the instance profile:

```
aws iam add-role-to-instance-profile --instance-profile-name MyEC2InstanceProfile --role-name MyEC2SSMRole
```

### Gather Necessary Data for Plugging into EC2 Instance Creation Command

1. Get the subnet ID:

```
aws ec2 describe-subnets --query "Subnets[?Tags[?Value == 'SubnetA'] ].SubnetId | [0]"
```

2. Copy the subnet ID in the output into a text file (or save in a variable).

3. Get the security group ID:

```
aws ec2 describe-security-groups \
--filters Name=group-name,Values=SG \
--query "SecurityGroups[?GroupName == 'SG'].GroupId | [0]"
```

4. Copy the security group ID in the output into a text file (or save in a variable).

5. Get the AMI 2 ID (with SSM installed):

```
aws ec2 describe-images \
--filters "Name=architecture,Values=x86_64" "Name=description,Values=*Amazon Linux 2 AMI 2.0.2019*gp2" "Name=owner-id,Values=137112412989" "Name=image-type,Values=machine" \
--query "sort_by(Images, &CreationDate)[::-1].ImageId | [0]"
```

6. Copy the AMI 2 ID in the output into a text file (or save in a variable).

### Create an EC2 Instance Using Subnet, Security Group, and AMI ID

1. Paste the subnet ID, security group ID, AMI ID, and instance profile ARN into the following command:

```
aws ec2 run-instances --associate-public-ip-address \
--security-group-ids <SECURITY-GROUP-ID> \
--iam-instance-profile Arn=<INSTANCE-PROFILE-ARN> \
--instance-type t2.micro \
--image-id <AMI-ID> \
--subnet-id <SUBNET-ID> \
--tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=MyEC2}]"
```

### Confirm via SSM CLI API (or GUI) that the Newly Created EC2 Instance Is Visible to It

1. Get the instance ID of the newly created instance:

```
aws ec2 describe-instances --filters Name=tag:Name,Values=MyEC2 --query "Reservations[].Instances[].InstanceId"
```

2. Run the following SSM instance information API command to see if the same instance ID from the command above is showing up in SSM API's output:

```
aws ssm describe-instance-information --query "InstanceInformationList[?InstanceId == '<INSTANCE-ID>']"
```

The command should return an object with information about the newly configured EC2 instance with Systems Manager.

Note: It can take up to five minutes for the instance to show up in the SSM GUI or API command output.