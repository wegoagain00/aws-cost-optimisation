
(image coming) architecture is lambda function (which will have python code (boto3)), it will talk to the AWS API, - to fetch EBS snapshots and to filter out snapshots that are 'still' then delete them

**Identifying Stale EBS Snapshots**

In this example, we'll create a Lambda function that identifies EBS snapshots that are no longer associated with any active EC2 instance and deletes them to save on storage costs.

**Description:**

The Lambda function fetches all EBS snapshots owned by the same account ('self') and also retrieves a list of active EC2 instances (running and stopped). For each snapshot, it checks if the associated volume (if exists) is not associated with any active instance. If it finds a stale snapshot, it deletes it, effectively optimising storage costs.

**STEPS:**

Create a simple EC2 instance

When creating an instance, it comes with a volume. When you want to save information of the volume at that specific moment, you create a snapshot, you select the volume of EC2 instance and then give it a name.

![alt text](images/img-1.png)

Now search LAMBDA

Create a lambda function (this code will test to see if an instance has a volume and also if it has a snapshot attached to it.)

![alt text](images/img-2.png)

Now scroll down to code source and paste the .py script in the repository, and press deploy to save it.

Then create test event, then test it. (there will be an error in OUTPUT as the script takes more than 3 seconds to run and this needs to be changed in CONFIGURATION → EDIT → TIMEOUT → 10secs.)

![alt text](images/img-3.png)

You will also notice an error when you read the output and this is because the code requires permission to ‘DescribeSnapshots’ and few others. 

When a Lambda function needs to interact with other AWS services, it must be granted the appropriate IAM permissions through its execution role. By default, a new Lambda function does not have permissions to perform actions like creating a snapshot. You need to attach an IAM role with the necessary policies to allow the Lambda to perform these operations.

Now in CONFIGURATION → PERMISSION → role name | click on the name and it will take you to IAM where now we will give the policies using ‘create inline policy’, choose json format and paste this script (or find it manually).

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "EBSSnapshotCleanupPermissions",
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeSnapshots",
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "ec2:DeleteSnapshot"
      ],
      "Resource": "*"
    }
  ]
}
```

We are giving the bare minimum permissions that we need based on the python script as this is **good security practice**.

![alt text](images/img-4.png)

Now atatch the created policy and lets **test** run the script and it should work fine, however the snapshot wont be deleted as we still have the instance with volume and snapshot attached.

![alt text](images/img-5.png)

Now lets delete our instance and you should see the volume deleted but not the snapshot. (wait few minutes to fully delete)

![alt text](images/img-6.png)

![alt text](images/img-7.png)

Lets go back to our lambda function and run it, now we should see the snapshot deleted.

![alt text](images/img-8.png)

you can now go to EC2 → snapshot and it has been deleted

---
We have been running this manually as a test - to have this running every day or week, we can use CloudWatch. Open EventBridge → rules → create rule. Following this will allow you to create an automatic flow for the Lambda.

This is something we can do for our next project.