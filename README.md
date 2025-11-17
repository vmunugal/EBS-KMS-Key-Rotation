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

## Architecture Components

### AWS Step Functions State Machine
The solution uses AWS Step Functions to orchestrate the entire key rotation workflow, providing:
- **Visual workflow management**: Clear state transitions and error handling paths
- **Parallel processing**: Handles multiple volumes simultaneously for efficiency
- **Built-in retry logic**: Automatic retry mechanisms for transient failures
- **State persistence**: Maintains execution state throughout the process

### Lambda Functions (16 Core + 4 Rollback)
The solution consists of 20 specialized Lambda functions:

#### Core Processing Functions:
1. **ValidateInput** - Validates KMS keys, instance IDs, and regions
2. **FetchVolumesEncryptedWithOldKey** - Identifies volumes using the target KMS key
3. **PreserveInputToSSM** - Stores original instance state in SSM Parameter Store
4. **StopInstance** - Safely stops EC2 instances
5. **FetchInstanceState** - Monitors instance state transitions
6. **DetachVolumes** - Detaches EBS volumes from instances
7. **FetchVolumeState** - Monitors volume state changes
8. **CreateSnapshot** - Creates snapshots of existing volumes
9. **FetchSnapshotState** - Monitors snapshot creation progress
10. **CopySnapshot** - Copies snapshots with new KMS key encryption
11. **FetchCopiedSnapshotState** - Monitors copied snapshot status
12. **CreateNewVolume** - Creates new volumes from encrypted snapshots
13. **FetchNewVolumeStatus** - Monitors new volume creation
14. **AttachNewVolToInstance** - Attaches new volumes to instances
15. **StartInstances** - Restarts EC2 instances
16. **ValidateIfInstanceRunning** - Confirms instance startup
17. **CleanupResources** - Removes old volumes and snapshots

#### Rollback Functions:
18. **RollbackDetachVolumes** - Reattaches original volumes if detachment fails
19. **RollbackCreateSnapshot** - Cleans up and restores state if snapshot creation fails
20. **RollbackCopySnapshot** - Handles failures during snapshot copying
21. **RollbackCreateVolume** - Manages failures during volume creation

### State Management
- **SSM Parameter Store**: Preserves original instance configuration for rollback scenarios
- **Status tracking**: Each step maintains detailed status information for monitoring
- **Error context**: Comprehensive error information for troubleshooting

## Current Features
- **Zero-data-loss rotation**: Uses snapshot-based approach to ensure data integrity
- **Cost optimization**: Automatic cleanup of old resources
- **Minimal downtime**: Optimized process flow to reduce service interruption
- **Comprehensive error handling**: Multi-level rollback capabilities with state restoration
- **Parallel processing**: Handles multiple volumes per instance simultaneously
- **State persistence**: Uses SSM Parameter Store to maintain original configurations
- **Detailed logging**: CloudWatch logging for audit and troubleshooting
- **Input validation**: Validates all inputs before processing begins

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

### CloudFormation Template Parameters
- **RetentionInDays**: CloudWatch Log group retention (1-3653 days, default: 14)
- **LambdaTimeout**: Lambda function timeout in seconds (60-900, default: 90)

## Security & Permissions

### IAM Roles Created
1. **EBSKMSKeyRotationLambdaRole**: Execution role for Lambda functions with permissions for:
   - EC2 operations (describe, stop, start, volume management, snapshot operations)
   - KMS operations (describe, generate data keys, re-encrypt, create grants)
   - SSM Parameter Store (put, get, delete parameters)
   - CloudWatch Logs (create log groups/streams, put log events)

2. **EBSKMSKeyRotationStateMachineRole**: Execution role for Step Functions with permissions to:
   - Invoke all Lambda functions in the workflow
   - Manage state machine execution

### Security Best Practices
- **Least privilege access**: Each role has minimal required permissions
- **Resource-specific permissions**: SSM parameters scoped to instance IDs
- **Encryption in transit**: All API calls use HTTPS
- **State preservation**: Original configurations stored securely in SSM

## Monitoring & Observability

### CloudWatch Integration
- **Log Groups**: Separate log groups for each Lambda function
- **Configurable retention**: Log retention from 1 day to 10 years
- **Execution tracking**: Step Functions provides visual execution monitoring
- **Error alerting**: Built-in error handling with detailed error messages

### Operational Metrics
- **Execution duration**: Track total rotation time per instance
- **Success/failure rates**: Monitor rotation success across instances
- **Resource cleanup**: Verify old resources are properly removed
- **Cost tracking**: Monitor snapshot and volume costs during rotation

# Step 3: Create CloudFormation Stack to deploy the solution.
1. Open the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation.
2. On the navigation bar at the top of the screen, choose the AWS Region to create the stack in.
3. On the Stacks page, choose Create stack at top right, and then choose With new resources (standard).
4. On the Create stack page, choose  **Template is ready**
5. choose **Choose File** to choose **ebs-key-rotation.yaml** file from your local computer.
6. Choose Next to continue and to validate the template.
7. On the **Specify stack details** page, type a stack name in the **Stack name** box.
8. In the **Parameters** section, specify values for the parameters that were defined in the template.
    >> **RetentionInDays** - CloudWatch log retention period (default: 14 days)
       - Allowed values: 1, 3, 5, 7, 14, 30, 60, 90, 120, 150, 180, 365, 400, 545, 731, 1096, 1827, 2192, 2557, 2922, 3288, 3653 days
       - Choose based on your compliance and troubleshooting requirements

    >> **LambdaTimeout** - Lambda function timeout (default: 90 seconds, range: 60-900 seconds)
       - Recommended to keep default unless you have specific performance requirements
       - Increase for large volume operations or slower storage performance
9. Choose **Next** to continue creating the stack.
10.  On the **Configure stack options** page, choose the IAM role created in **Step 1**
11.  Choose **I acknowledge that this template may create IAM resources** to specify that you want to use IAM resources in 
     the template.
12.  Choose **Next** to continue. Choose **Submit** to launch your stack.

## Post-Deployment

### Execution
After successful deployment, the solution creates:
- **Step Functions State Machine**: `EBSKMSKeyRotationStateMachine`
- **20 Lambda Functions**: All functions prefixed with their specific purpose
- **IAM Roles**: Execution roles for Lambda and Step Functions
- **CloudWatch Log Groups**: Separate log groups for each Lambda function

### Usage
1. Navigate to AWS Step Functions console
2. Find the `EBSKMSKeyRotationStateMachine`
3. Start execution with input JSON:
```json
[
  {
    "instance_id": "i-1234567890abcdef0",
    "kms_key_arn": "arn:aws:kms:region:account:key/old-key-id",
    "new_kms_key_arn": "arn:aws:kms:region:account:key/new-key-id",
    "region": "us-east-1"
  }
]
```

### Monitoring Execution
- **Step Functions Console**: Visual workflow progress
- **CloudWatch Logs**: Detailed execution logs per function
- **EC2 Console**: Monitor instance and volume states
- **Cost Explorer**: Track resource costs during rotation

## Troubleshooting

### Common Issues
1. **Permission Errors**: Verify IAM roles have required permissions
2. **KMS Key Access**: Ensure new KMS key allows EC2 service access
3. **Instance State**: Confirm instances are in stoppable state
4. **Volume Constraints**: Check for volumes with delete-on-termination settings

### Rollback Scenarios
The solution automatically handles rollbacks for:
- Volume detachment failures
- Snapshot creation errors
- Volume creation issues
- Attachment failures

### Manual Recovery
If manual intervention is required:
1. Check SSM Parameter Store for preserved instance states
2. Use rollback Lambda functions individually if needed
3. Verify instance and volume states in EC2 console
4. Clean up any orphaned snapshots or volumes

## Cost Considerations

### Resource Costs
- **Snapshots**: Temporary storage costs during rotation
- **Lambda Executions**: Minimal cost for function invocations
- **Step Functions**: State transitions and execution time
- **CloudWatch Logs**: Log storage based on retention settings

### Cost Optimization
- Automatic cleanup removes old snapshots and volumes
- Configurable log retention reduces long-term storage costs
- Parallel processing minimizes execution time
- Efficient state management reduces unnecessary API calls
