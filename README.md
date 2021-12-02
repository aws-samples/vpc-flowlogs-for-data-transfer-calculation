## vpc-flowlogs-for-data-transfer-calculation

### Single Account Setup
The deployment for a single account setup is as follows:

1. Upload the code for lambda functions and the lambda layer to an Amazon Simple Storage Service (S3) bucket in the Region where you want to deploy this infrastructure. This includes calculator.zip, loadAZCidr.zip, CreateVpcFlowlogs.zip and UpdateDDBTable.zip.
2. Log on to the AWS Management Console. Select the CloudFormation service.
3. Launch the CloudFormation stack using the template single-account-deployment.yml.
4. Provide the S3 bucket name that you used in step 1 and the IDs of the VPCs that you want to watch, to see the data transferred between Availability Zones.
5. Next, acknowledge the permissions required to provision IAM roles, and cross account permissions granted via Organization policies and create the stack.
6. Select the Resources tab in the center pane. Validate that all the resources are CREATE_COMPLETE and the stack is in CREATE_COMPLETE state.
7. Next, configure CloudWatch Contributor Insights and its associated alarms. To do this, perform:
  a. Log on to the AWS Management Console. Select the CloudWatch service.
  b. Select Contributor Insights on the left pane. In the create rule wizard, choose custom rule.
  c. Provide an appropriate rule name.
  d. Select the log group of your calculator lambda function (rcalculatorlambda) that was created by cloudformation stack in step 3.
  e. Select log format as JSON, Contribution as event.srIp and event.destIp
  f. Aggregate on SUM and event.bytes.
  g. Select create rule in enabled state and then choose Create.
 
8. To create alarms for your Contributor Insights metrics:
  a. In the CloudWatch console, from the left navigation pane, choose Contributor Insights and then choose the rule.
  b. Choose Actions and then choose View in Metrics.
  c. Choose Unique Contributors. This metric will be graphed in CloudWatch metrics.
  d. Choose the alarm icon in the row of the metric. For example, create an alarm when there are more than 1GB of data transfer observed per minute between a particular source and destination.
  e. Choose Create. Figure 4 provides a Contributor Insight as a metric for data transfer between AZs.

### Multi Account Setup
The deployment comprises of three steps for a multi account setup. We have outlined the details of each step below:

#### Pre-requisites:

1. Setting up IAM roles and policies:   

All the stacks launched must be prefixed with DTAZ. For example, create stacks with names - DTAZ-pre-role, DTAZ-Data-Transfer-Calculator and DTAZ-Data-Transfer-Update etc.

2. Spoke account:

  a. Log on to the AWS Management Console of your spoke account. Upload loadAZCidr.zip, CreateVpcFlowlogs.zip and UpdateDDBTable.zip file to s3 bucket that stores your lambda code.
  b. Log on to the AWS Management Console of spoke account. Select CloudFormation service.
  c. Launch a CloudFormation stack using the template pre-roles.yml . This creates the execution roles required for the lambda functions.
  d. Next, acknowledge the permissions required to provision IAM roles, and cross account permissions granted via Organization policies and create the stack.
  e. Select the Resources tab in the center pane. Validate that all the resources are CREATE_COMPLETE and the stack is in CREATE_COMPLETE state. Figure 5 shows a pre-role stack along with its parameters.

### Step 1 : Set up the Hub Account

1. Upload the code for lambda function, calculator.zip, to the S3 bucket in the Region where you want to deploy the infrastructure.
2. Select CloudFormation service. CloudFormation stack using the template data-transfer-calculator.yml This creates the DynamoDB table and the central lambda function that calculates the cost.
3. Select the Resources tab in the center pane. Validate that all the resources are CREATE_COMPLETE and the stack is in CREATE_COMPLETE state.

### Step 2 : Set up the Spoke Account

1. Copy the outputs from the stack provisioned by the data transfer calculator template in the hub account. This will serve as inputs to the next step.
2. Select CloudFormation and launch a CloudFormation stack using the data-transfer-calculator-update.yml template. This creates a lambda function and a custom resource. The Lambda function is triggered by a AWS CloudTrail API for any subnet creation activity and keeps the DynamoDB up to date with subnet - AZ mapping. 3. The custom resource and the backend lambda function will push the initial data (current subnet AZ IDs and CICD blocks) to DynamoDB table.
4. Select the Resources tab in the center pane. Validate that all the resources are CREATE_COMPLETE and the stack is in CREATE_COMPLETE state.

### Step 3 : Set up the Contributor Insights and Alarms in CloudWatch

Next, configure CloudWatch Contributor Insights metrics for data transfer between Availability Zones, and its associated alarms as described in the single account setup.

### Cleanup

To avoid ongoing charges, delete the resources you created. Go to the AWS Management Console, identify the CloudFormation stack you launched. Select delete stack. This operation will delete the resources you created (the Dynamo DB, the lambda functions, Contributor Insights rules, CloudWatch Logs Insights rules, and alarms).

## Contributors
Shiva Vaidyanathan - vaidys@amazon.com

Stan Fan - fanhongy@amazon.com

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

