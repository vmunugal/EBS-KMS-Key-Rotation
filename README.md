# AWS KMS Key Rotation Solution
This repository provides an automated solution for rotating KMS keys across AWS services, starting with EBS volume encryption key rotation. The solution is designed to be extensible to support other KMS-integrated AWS services in future releases.

## Solution Overview
The KMS Key Rotation solution automates the process of rotating encryption keys for AWS resources with minimal downtime. The current implementation focuses on EBS volumes attached to EC2 instances, with plans to extend support to other AWS services that integrate with KMS.

### EBS Volume Key Rotation Process
The solution performs the following automated steps:
1. **Input Validation**: Accepts target KMS key and EC2 instance IDs as input parameters
2. **Instance Management**: Safely stops the target EC2 instance
3. **Volume Operations**: Detaches all EBS volumes from the instance
4. **Snapshot Creation**: Creates snapshots of existing volumes for backup
5. **Encryption Migration**: Copies snapshots to the same region with new KMS key encryption
6. **Volume Recreation**: Creates new EBS volumes from the re-encrypted snapshots
7. **Reattachment**: Attaches the new volumes to the EC2 instance
8. **Instance Restart**: Starts the EC2 instance with updated encryption
9. **Cleanup**: Removes old volumes and snapshots to optimize costs

### Future Extensibility
This solution serves as the foundation for a comprehensive KMS key rotation platform that will be extended to support:
- RDS encrypted databases
- S3 bucket encryption
- EFS file systems
- Lambda environment variables
- Secrets Manager secrets
- Parameter Store secure strings
- And other KMS-integrated AWS services

## Current Features
- **Zero-data-loss rotation**: Uses snapshot-based approach to ensure data integrity
- **Cost optimization**: Automatic cleanup of old resources
- **Minimal downtime**: Optimized process flow to reduce service interruption
- **Error handling**: Comprehensive error handling and rollback capabilities
- **Logging**: Detailed CloudWatch logging for audit and troubleshooting

# Deployment steps
# Step 1: Create an IAM Role for CloudFormation
1. Sign in to the **AWS Management Console** and open the IAM console at https://console.aws.amazon.com/iam/.
2. In the navigation pane of the IAM console, choose **Roles**, and then choose **Create role**.
3. For **Trusted entity type**, choose **AWS service**.
4. For Service or use case, choose **CloudFormation** as service and Choose **Next**.
5. For Permissions policies, **Select no permissions policies, create the policies after the role is created, and then attach the policies to the role.**
6. Choose **Next**.
7. For **Role name** provide a role name.
8. Review the role, and then choose **Create role**.

# Step 2: Add permissions to the role:
1. Sign in to the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/.
2. In the navigation pane, choose **Roles**.
3. In the list, choose the name of the role created in **Step 1**.
4. Choose the **Permissions tab**.
5. Choose **Add permissions** and then choose **Create inline policy**.
6. In the **JSON editor** copy and paste the below policy:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "lambda:CreateFunction",
                "lambda:GetFunction",
                "logs:DeleteLogGroup",
                "lambda:DeleteFunction",
                "logs:PutRetentionPolicy",
                "logs:CreateLogGroup",
                "states:DescribeStateMachine",
                "lambda:UpdateFunctionCode"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole",
                "iam:DeleteRolePolicy",
                "states:DeleteStateMachine",
                "iam:CreateRole",
                "iam:DeleteRole",
                "iam:PutRolePolicy",
                "iam:GetRole",
                "iam:GetRolePolicy",
                "states:CreateStateMachine",
                "states:TagResource",
                "states:UpdateStateMachine"
            ],
            "Resource": [
                "arn:aws:states:*:*:stateMachine:EBSKMSKeyRotationStateMachine*",
                "arn:aws:iam::*:role/EBSKMSKeyRotationStateMachineRole*",
                "arn:aws:iam::*:role/EBSKMSKeyRotationLambdaRole*"
            ]
        }
    ]
}
```


## Input Parameters
The solution accepts the following input parameters:
- **Target KMS Key**: The new KMS key ARN or ID to encrypt volumes with
- **Instance IDs**: List of EC2 instance IDs whose EBS volumes need key rotation
- **Retention Settings**: Snapshot retention period for backup purposes

# Step 3: Create CloudFormation Stack to deploy the solution.
1. Open the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation.
2. On the navigation bar at the top of the screen, choose the AWS Region to create the stack in.
3. On the Stacks page, choose Create stack at top right, and then choose With new resources (standard).
4. On the Create stack page, choose  **Template is ready**
5. choose **Choose File** to choose **ebs-key-rotation.yaml** file from your local computer.
6. Choose Next to continue and to validate the template.
7. On the **Specify stack details** page, type a stack name in the **Stack name** box.
8. In the **Parameters** section, specify values for the parameters that were defined in the template.
    >> **RetentionInDays** - This parameter is defined as 14 days, modify this with the allowed values from drop down to set                                 desired retention for CloudWatch log groups.

    >> **LambdaTimeout** - This Parameter defines the timeout for lambda function in seconds, it is suggested to not to                                 modify this parameter.
9. Choose **Next** to continue creating the stack.
10.  On the **Configure stack options** page, choose the IAM role created in **Step 1**
11.  Choose **I acknowledge that this template may create IAM resources** to specify that you want to use IAM resources in 
     the template.
12.  Choose **Next** to continue. Choose **Submit** to launch your stack.
