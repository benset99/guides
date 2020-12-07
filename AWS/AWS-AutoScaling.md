# AWS Learning Series
Where I document my AWS learning progress

Prerequisities:
* Some working knowledge of Linux
* Followed the AWS-WebBasics guide

# Auto-Scaling Basics

## Creating a launch template

**Create a launch template**

Note: The security group and image was created using the AWS-WebBasics guide found in this repo.  

Create a launch template:  
`aws ec2 create-launch-template --launch-template-name benset-launch-template --version-description 0.1 --launch-template-data '{"NetworkInterfaces":[{"DeviceIndex":0,"Groups":["sg-0c28afaf2c775e906"],"DeleteOnTermination":true}],"ImageId":"ami-05b2a7a22fe19b0f0","InstanceType":"t2.micro"}'`

**Get list of subnets from your VPC**

Instead of using the AWS console, we are going to try to get a list of our subnets using AWS CLI:   
`aws ec2 describe-subnets | grep SubnetId`

Output:
```
"SubnetId": "subnet-74c0b97a",
"SubnetId": "subnet-48da5769",
"SubnetId": "subnet-d679feb0",
"SubnetId": "subnet-4bf3a406",
"SubnetId": "subnet-72ff3543",
"SubnetId": "subnet-4d018112",
```

**Create a load balancer**

Create a load balancer for the EC2 so that connections to the EC2 application will still be open if one of the EC2 instances go down:   
`aws elb create-load-balancer --load-balancer-name benset-elb --listeners "Protocol=HTTP,LoadBalancerPort=80,InstanceProtocol=HTTP,InstancePort=80" --subnets subnet-4d018112 subnet-d679feb0 --security-groups sg-0c28afaf2c775e906`

Write down the output of this command for later use:  
`aws elb describe-load-balancers | grep DNS`

**Create an Auto Scaling group and attach the load balancer**

We will use the launch template that we created to create an Auto Scaling group which will automatically configure the instances and we will attach the existing load balancer that we just created as well. 

`aws autoscaling create-auto-scaling-group --auto-scaling-group-name benset-asg --launch-template "LaunchTemplateName=benset-launch-template,Version=1" --min-size 1 --max-size 2 --vpc-zone-identifier "subnet-4d018112,subnet-d679feb0" --load-balancer-names "benset-elb" --max-size 3 --min-size 2 --desired-capacity 2` 

**Visit the site**

Check if your newly created load balancer and auto scaling group:  
`curl benset-elb-224395725.us-east-1.elb.amazonaws.com` 

Output:  
`<h1>Fortune-of-the-Day Coming Soon</h1>`

Also, you can stop the EC2 instances in the Auto Scaling group and it will automatically create a new instance

**Delete load balancer and Auto Scaling group**

Make sure you delete the load balancer and Auto Scaling group so it doesn't charge you.

```
aws elb delete-load-balancer --load-balancer-name benset-elb
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name benset-asg --force-delete
```

### You can view our simple HTML page served on both of our EC2 instances and if one of the instances goes down, our website is still accessible. 


