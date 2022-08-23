# How to auto stop and auto start AWS EC2 instances using AWS Lambda

# Prerequisites
* You need an Amazon Web Services Account.
* You need a created & launched EC2 instance to connect to. Instructions for this section can found in here.
Tagging Instances
Note: Tag all the instance which needs to be auto stop and auto start.

Region = ap-south-1 (Mumbai)
Name = AutoStart ; Value = True (To start the instances)
Name = AutoStop ; Value = True (To stop the instances)

# Create IAM role for Lambda
First step is you need to create an IAM role for your Lambda function which will responsible for managing the EC2 instance's lifcycles like starting and stopping. To create an IAM role, you need to follow the steps below:

1.Navigate to Services in AWS console and click onIAM
2.Click on Roles in left side navigation panel
3.Click on Create role
4.Select Lambda from the list of AWS Service

Click Next:Permission
Now you need to click to Create policy. It will pop up a new window. (Here we’ll create new custom ploicy for our Lambda function)

Click on JSON tab. Remove default code and add paste below json.
	{
    	"Version": "2012-10-17",
    	"Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:StartInstances",
                "ec2:StopInstances"
            ],
            "Resource": "arn:aws:ec2:*:*:instance/*"
        },
        {
            "Sid": "VisualEditor1",
            "Action": [
                "logs:CreateLogStream",
                "ec2:DescribeInstances"
                "ec2:DescribeTags"
                "logs:PutLogEvents"
                "logs:CreateLogGroup"
                "ec2:DescribeInstanceStatus"
            ],
            "Resource": "*"
        }
    ]
	}

Click on Review Policy

Then add policy name as lambda_stop_start_ec2_policyand description of the policy and then click on create policy

Now you should come back to the previous tab where we creating the Role

Here you search and select the new policy which we just created
lambda_stop_start_ec2_policy. (click on Refresh button available on top right first).

 
Click Next:tags and then Next:review. Now on review page enter Role name as lambda_stop_start_ec2 And some description of the role.

Now click on Create role.

New role has been created with EC2 start and stop permission with some additional CloudWatch logs permission.

Create Lambda function to Stop Instance
Click on services then Lambda


Click on Create new function.


Then Select Author from scratch


Under Basic information tab enter Function Name as AutoStopEC2Instance and select runtime as Python 3.7.

Click on Choose or create an execution role to expand.

Under Execution role select Use an Existing Role and select the role lambda_stop_start_ec2 which we have created in previous step and click on Create function.


Lambda Designer will open scroll down where you can find Function Code.


Under Function code you will find an inline editor with lambda_function.py. Delete the content of the file and paste below code.

	import boto3
	import logging
	#setup simple logging for INFO
	logger = logging.getLogger()
	logger.setLevel(logging.INFO)
	#define the connection and set the region
	ec2 = boto3.resource('ec2', region_name='ap-south-1')
	def lambda_handler(event, context):
    	# all running EC2 instances.
    filters = [{
            'Name': 'tag:AutoStop',
            'Values': ['True']
        },
        {
            'Name': 'instance-state-name', 
            'Values': ['running']
        }
    ]
    
    #filter the instances which are stopped
    instances = ec2.instances.filter(Filters=filters)

    #locate all running instances
    RunningInstances = [instance.id for instance in instances]
    
    #print the instances for logging purposes
    #print RunningInstances 
    
    if len(RunningInstances) > 0:
        #perform the shutdown
        shuttingDown = ec2.instances.filter(InstanceIds=RunningInstances).stop()
        print(shuttingDown)
    else:
        print("Nothing to see here")


