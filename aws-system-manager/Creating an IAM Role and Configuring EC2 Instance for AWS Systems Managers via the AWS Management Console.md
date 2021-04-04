# Creating an IAM Role and Configuring EC2 Instance for AWS Systems Manager via the AWS Management Console

## Introduction

In this hands-on lab, you will learn about the IAM role necessary for configuring an EC2 instance with AWS Systems Manager service. We'll create and attach a role to an EC2 instance via the AWS Management Console (GUI) and confirm that it is configured with SSM service by checking in the Systems Manager console as a managed instance.

## Solution

Log in to the live AWS environment using the credentials provided. Make sure you're in the N. Virginia (us-east-1) region throughout the lab.

### Create an EC2 IAM Role with SSM Policy for EC2

1. Navigate to IAM > Roles.
2. Click Create role.
3. Select AWS service as the type of trusted entity.
4. Select EC2 as the service that will use the role.
5. Click Next: Permissions.
6. On the permissions policies page, search for (in the Filter policies box) and select AmazonEC2RoleforSSM.
7. Click Next: Tags.
8. Leave as default, and click Next: Review.
9. Enter a Role name of "MyEC2SSMRole".
10. Click Create role.

### Launch an EC2 Instance

1. Navigate to EC2 > Instances.
2. Click Launch Instance.
3. On the AMI page, select the Amazon Linux 2 AMI.
4. Leave t2.micro selected, and click Next: Configure Instance Details.
5. On the Configure Instance Details page:
* Network: Leave default
* Subnet: SubnetA
* Auto-assign Public IP: Enable
* IAM role: MyEC2SSMRole
6. Click Next: Add Storage, and then click Next: Add Tags.
7. On the Add Tags page, add the following tag:
* Key: Name
* Value: MyEC2
8. Click Next: Configure Security Group.
9. Click to Select an existing security group.
10. Select SG from the table.
11. Click Review and Launch, and then Launch.
12. In the key pair dialog, select Proceed without a key pair.
13. Check the acknowledgement box.
14. Click Launch Instances.
15. Click View Instances, and give it a few minutes to enter the running state.

### Verify the EC2 Instance Is Configured with SSM Service Properly

1. Navigate to Systems Manager > Managed Instances.
* It can take up to five minutes for the instance to show up on this page.
2. The instance you created with the SSM role should be visible here, confirming you chose the correct AMI and set up the SSM IAM role properly.