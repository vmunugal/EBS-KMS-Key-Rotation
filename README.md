# EBS-KMS-Key-Rotation
This repo holds the CFN template to deploy the EBS KMS key Rotation solution

# Steps to deploy the solution:

# Step 1: Create IAM Role for CloudFormation
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


# Step 2: Create CloudFormation Stack to deploy the solution.
1. Open the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation.
2. On the navigation bar at the top of the screen, choose the AWS Region to create the stack in.
3. On the Stacks page, choose Create stack at top right, and then choose With new resources (standard).
4. On the Create stack page, choose  **Template is ready**
5. choose **Choose File** to choose **ebs-rotation.yaml** file from your local computer.
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
