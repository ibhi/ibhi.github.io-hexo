title: Stop and Start Amazon EC2 instances at regular intervals using AWS Lambda
comments: true
date: 2018-01-13 13:01:54
tags: 
    - ec2
    - lambda
categories: Cloud
---
{% asset_img amazon-ec2.jpg %}
### Requirement
Recently I was talking to a friend who was using AWS EC2 instances for their dev-test environment and complaining about the huge cost associated with running it. One of the suggestion I provided to reduce the cost was to stop the instances when not in use, as they were on-demand instances and AWS only charges for the running time. So I proposed a scheduled start and stopping of the instances during certain time of the day, for example, the dev-test instances will be automatically started every day at 9 in the morning and stopped at 9 in the night.

### Solution
We can use a CloudWatch Event to trigger a Lambda function to start and stop EC2 instances at scheduled intervals. We can configure Cloud watch to emit a event based on a predefined schedule, and this event can trigger a lambda function and within the lambda function we can use the aws-sdk for nodejs to start and stop the specified instances.

### Lambda functions and permissions
Lets create Lambda functions and configure necessary permissions step by step.

1. Open the AWS Lambda console, and choose Create function.
2. Choose Author from scratch.
3. Enter a Name for your function, such as "StopEC2Instances".
4. Expand the Role drop-down menu, and choose Create a custom role. This opens a new tab or window in your browser.
5. In the IAM Role drop-down menu, select Create a new IAM Role, and enter a Role Name, such as “lambda_start_stop_ec2."
6. Choose View Policy Document, Edit, and then choose Ok when prompted to read the documentation. Edit the policy as follows:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ec2:Start*",
        "ec2:Stop*"
      ],
      "Resource": "*"
    }
  ]
}
```
7. Choose Allow to finish creating the role and return to the AWS Lambda console.
8. To stop your instances, enter the following into the Function code editor:

```javascript
//  Enter the region your instances are in. Include only the region without specifying Availability Zone; e.g.; 'us-east-1'
const region = 'XX-XXXXX-X';
// Enter your instances here: ex. ['X-XXXXXXXX', 'X-XXXXXXXX']
const instances = ['X-XXXXXXXX'];

var AWS = require('aws-sdk');
AWS.config.update({region: region});

exports.handler = (event, context, callback) => {
    // TODO implement
    const ec2 = new AWS.EC2({apiVersion: '2016-11-15'});
    const params = {
        InstanceIds: instances
    };

    ec2.stopInstances(params, (err, data) => {
        if (err) {
            console.log("Error", err);
            callback(err);
        } else if (data) {
            console.log("Success", data.StoppingInstances);
            callback(null, data.StoppingInstances);
        }
    });
    
};
```
9. From the Runtime drop-down menu, choose nodejs.
10. In Basic settings, enter 10 seconds for the function Timeout.
11. Choose Save.
12. Repeat these steps to create another function that starts your instances again by using the following:

```javascript
//  Enter the region your instances are in. Include only the region without specifying Availability Zone; e.g.; 'us-east-1'
const region = 'XX-XXXXX-X';
// Enter your instances here: ex. ['X-XXXXXXXX', 'X-XXXXXXXX']
const instances = ['X-XXXXXXXX'];

var AWS = require('aws-sdk');
AWS.config.update({region: region});

exports.handler = (event, context, callback) => {
    // TODO implement
    const ec2 = new AWS.EC2({apiVersion: '2016-11-15'});
    const params = {
        InstanceIds: instances
    };

    ec2.startInstances(params, (err, data) => {
        if (err) {
            console.log("Error", err);
            callback(err);
        } else if (data) {
            console.log("Success", data.StartingInstances);
            callback(null, data.StartingInstances);
        }
    });
    
};
```

Note: Use a Name and Description that indicate this function is used to start instances. You can use the previously created role.

### Test your newly created functions

1. Open the AWS Lambda console, and choose Functions.
2. Select your function, and then choose Test.
3. In Event name, enter a name and then choose Create.

Note: The body of the test event doesn't impact your function because the function does not use it.

### Create a CloudWatch Event that triggers your Lambda function at night

1. Open the Amazon CloudWatch console.
2. Choose Events, and then choose Create rule.
3. Choose Schedule under Event Source.
4. Enter an interval of time or cron expression that tells Lambda when to stop your instances. For more information on the correct syntax, see [Schedule Expression Syntax for Rules](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/ScheduledEvents.html).
eg: `30 3 * * ? *`
**Note**: Cron expressions are evaluated in UTC. Be sure to adjust the expression for your preferred time zone.
5. Choose Add target, and then choose Lambda function.
6. For Function, choose the Lambda function that stops your instances.
7. Choose Configure details.
8. Enter the following information in the provided fields:
For Name, enter a meaningful name, such as "StopEC2Instances."
For Description, add a meaningful description, such as “stops EC2 instances every day at night.” 
For State, select Enabled.
9. Choose Create rule.

To restart your instances in the morning, repeat these steps and use your preferred start time.