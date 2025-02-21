# AWS CloudFormation IAM User Management

## Overview
This project automates the creation and management of IAM users, groups, and permissions using AWS CloudFormation. It also integrates AWS Secrets Manager, AWS Lambda, and Amazon EventBridge to manage and log user credentials securely.

## Features
- **IAM User & Group Management**: Creates IAM users and assigns them to specific groups with predefined permissions.
- **Secrets Management**: Uses AWS Secrets Manager to store one-time passwords for IAM users securely.
- **Event-Driven Logging**: Listens to IAM user creation events via Amazon EventBridge and triggers an AWS Lambda function.
- **AWS Lambda Function**: Logs IAM user details and retrieves one-time passwords from AWS Secrets Manager.
- **CloudFormation Deployment**: Automates resource provisioning for IAM, Lambda, EventBridge, and logging infrastructure.

## Architecture
The stack includes:
- **IAM Groups**:
  - `S3UserGroupV3`: Users in this group get read-only access to the specified S3 bucket.
  - `EC2UserGroupV3`: Users in this group get read-only access to EC2 instances.
- **IAM Users**:
  - `moses-ec2`: Assigned to `EC2UserGroupV3`.
  - `moses-s3`: Assigned to `S3UserGroupV3`.
- **IAM Policies**:
  - `S3ReadOnlyAccessPolicy-v3`: Grants S3 read-only permissions.
  - `EC2ReadOnlyAccessPolicy-v3`: Grants EC2 describe-instance permissions.
- **Secrets Management**:
  - `OneTimePassword`: Stores a generated password for new users.
- **AWS Lambda**:
  - Logs new user creation events and retrieves credentials from AWS Secrets Manager.
- **EventBridge Rule**:
  - Triggers the Lambda function on IAM user creation events.
- **AWS CloudWatch Logging**:
  - Stores logs for user creation activities.

## Deployment
### Prerequisites
- AWS CLI installed and configured.
- An S3 bucket to store the Lambda JAR file.
- Java 17 for the Lambda function.

### Steps
1. **Upload Lambda JAR to S3**:
   ```sh
   aws s3 cp CloudFormationUser-1.0-SNAPSHOT.jar s3://moses-test-bucket-67899/
   ```

2. **Deploy CloudFormation Stack**:
   ```sh
   aws cloudformation deploy \
     --template-file template.yaml \
     --stack-name IAM-User-Management \
     --parameter-overrides Password="MySecurePassword" Environment="dev" \
     --capabilities CAPABILITY_NAMED_IAM
   ```

## Lambda Function
The Lambda function `LogUserCreationHandler`:
- Receives IAM user creation events from EventBridge.
- Retrieves the user’s email.
- Fetches the one-time password from AWS Secrets Manager.
- Logs the user details securely.

### Example Event Payload
```json
{
  "detail": {
    "eventName": "CreateUser",
    "requestParameters": {
      "userName": "moses-ec2"
    }
  }
}
```

## Cleanup
To delete the CloudFormation stack and all associated resources:
```sh
aws cloudformation delete-stack --stack-name IAM-User-Management
```

## Author
Moses Winbood Ayandau

