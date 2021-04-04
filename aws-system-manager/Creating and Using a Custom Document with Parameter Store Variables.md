# Creating and Using a Custom Document with Parameter Store Variables

## Introduction

Systems Manager documents are an integral part of the Systems Manager service. They are at the heart of all the automation possible through SSM via JSON or YAML runbooks, which define steps to perform on a managed instance. In this lab, we'll create a document that carries out some tasks on a managed instance and will also use an SSM parameter, which offers scalable, hierarchal storage for configurations and secrets, allowing encryption.

## Log in to the AWS Management Console and Navigate to Systems Manager

1. Log in to the AWS Management Console using the credentials provided.
2. Navigate to the Systems Manager > Parameter Store.

## Create SSM Parameter to Use in SSM Document

1. Click Create parameter, and set the following values:

* Name: mysql-pass
* Description: Password for MySQL DB
* Tier: Standard
* Types: String
* Value: linuxacademy123456

2. Leave the Tags field as its default.
3. Click Create parameter.

## Create SSM Command Document
1. In the left-hand menu, click Documents.
2. Click Create command or session.
3. Give your document the name "MySQLDocument".
4. Leave the Target type dropdown field blank.
5. Set the Document type to Command document.

## Enter the Provided SSM Command Document Schema

1. In the Content section, choose the radio button for JSON.
2. Delete the existing code.
3. Paste in the following SSM Command document schema (also provided on the lab page):

``` 
{
  "schemaVersion": "2.2",
  "description": "Command Document Example JSON Template",
  "parameters": {
    "Packages": {
      "type": "String",
      "description": "MySQL package",
      "default": "mariadb-server"
    },
    "Password": {
      "type": "String",
      "description": "MySQL Password",
      "default": "{{ ssm:mysql-pass }}"
    }
  },
  "mainSteps": [
    {
      "action": "aws:runShellScript",
      "name": "Install_Configure_MariaDB",
      "inputs": {
        "runCommand": [
          "yum -y install {{ Packages }}",
          "systemctl start mariadb",
          "sleep 5;mysqladmin password {{ Password }};",
          "mysql -uroot -p{{ Password }} -e 'show databases;' > /root/db_output.txt"
        ]
      }
    }
  ]
}
```

4. Leave the Document tags section as its default.
5. Click Create document.

## Execute the SSM Document

1. Select the Owned by me tab.
2. Click the document you created.
3. Click Run command to execute your document.
4. Leave Document version as Default.
5. For Targets, select Choose instances manually.
6. In the Instances table, select the listed AmazonLinux-Instance.
7. In the Output options section, uncheck the Enable writing to an S3 bucket option.
8. Leave everything else as default.
9. Don't click Run just yet.

## Use SSM Session to Connect to the Managed Instance and Verify

1. In the left-hand menu, open Session Manager in a new browser tab.
2. Click Start Session.
3. Select AmazonLinux-Instance.
4. Click Start session.
5. In the shell session, become root:

```
sudo su - root
```

6. View the SSM Agent log file:

```
tail -f /var/log/amazon/ssm/amazon-ssm-agent.log
```

7. Back on the Run Command page in the AWS console, click Run.
8. In the shell session browser tab, note the many commands that have gone through.
9. On the Run Command page, click the refresh icon until the status changes to Success.
10. Click the listed instance ID.
11. Expand the Step 1 - Output section to see what the document did.
12. In the Session Manager shell session, press Ctrl+C to quit the process.
13. Make sure the /root/db_output.txt file exists:

```
ls -ltrh /root/db_output.txt
```
We should see that it's there.
14. View its contents:

```
cat /root/db_output.txt
```
We should see a list of database names.