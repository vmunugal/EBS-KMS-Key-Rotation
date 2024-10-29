# EBS-KMS-Key-Rotation
This repo holds the CFN template to deploy the EBS KMS key Rotation solution

# Steps to deploy the solution:
1. Open the AWS CloudFormation console at https://console.aws.amazon.com/cloudformation.
2. On the navigation bar at the top of the screen, choose the AWS Region to create the stack in.
3. On the Stacks page, choose Create stack at top right, and then choose With new resources (standard).
4. On the Create stack page, choose  **Template is ready**
5. choose **Choose File** to choose a template file from your local computer.
6. Choose Next to continue and to validate the template.
7. On the **Specify stack details** page, type a stack name in the **Stack name** box.
8. In the **Parameters** section, specify values for the parameters that were defined in the template.
    >> **RetentionInDays** - This parameter is defined as 14 days, modify this with the allowed values from drop down to set                                 desired retention for CloudWatch log groups.

    >> **LambdaTimeout** - This Parameter defines the timeout for lambda function in seconds, it is suggested to not to modify                             this parameter.
