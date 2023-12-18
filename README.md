# How to Deploy the cross-account encryption proccess

- This solution is a step functions workflow made up of 5 Lambda functions which carry out the process of encrypting EBS Volumes within an account based on a tag value. It is desgined to work in the following way.

- Within this directory you will find all the code files, the cloudformation template and a diagram of the solution.

# Prerequisites

1. Upload the 5 code files to an S3 bucket within a delegated administrator account / the master payer account and give it a bucket policy which allows it to be access by lambda functions across your org / accounts you want to specify.  See bucket-policy.json as an example.


- Add in your organization id and your bucket name in the corresponding places. This will ensure that each stack will be able to use the exact same code regardless of which account it is deployed in, ensuring consistency. 


2. Tag a maximum of 20 EC2 instances which you intend to encrypt with any of the following tags:

- Batch1:true
- Batch2:true
- Batch3:true
- Batch4:true
- Batch5:true
- Batch6:true
- Batch7:true
- Batch8:true

- The idea is that each time you deploy the cloudformation stack, you select a 'Batch' as a parameter when launching the stack, and this corresponds to a batch of of EC2 instances to encrypt. 

# Deployment steps

1. Deploy your cloudformation template as a Stack set within your delegated administrator account / in the master payer account using service-managed permissions - and choose to share the stack either across your entire organization or a particular OU - depending on how you want to deploy this solution. #

2. When you launch the stack you will provide the stack name, the bucket name of the bucket which you uploaded your code to, your tag value for the EC2 instances, and the email address you would like to be notified on when the process has completed via SNS. 

3. Once deployed, log into your corresponding account and run the step corresponding step functions state machine - and following the workflow, you should see the following process happen:

- Stop all instances with the corresponding tag
- Detach the EBS volumes (remembering the block device mapping of the volume)
- create a snapshot of the volume
- copy the snapshot and encrypt it using the default kms key
- restore the volume from the snapshot
- reattach the restored encrypted volume to the original instance (via the same block device mapping)
- run the instances 
