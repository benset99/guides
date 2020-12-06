# AWS Learning Series
Where I document my AWS learning adventure  
Prerequisites:  
* Some working knowledge of Linux 

# Web Hosting Basics
Make sure you correctly configured AWS CLI before starting with this guide.    
  
## Deploy a EC2 Instance 

**Create a security group which acts as a virtual firewall for the EC2 Instance:**   
`aws ec2 create-security-group --group-name EC2access --description "Allows for SSH and HTTP connections` 

This will output the GroupID and you will need to write it down somewhere for
later.

**Open ports to the newly created security group to allow SSH and HTTP connections into our EC2 instance:**  
```
aws ec2 authorize-security-group-ingress --group-name EC2access --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-name EC2access --protocol tcp --port 80 --cidr 0.0.0.0/0
```

**Create a key pair used to access our EC2 instance via SSH without a password:**  
`aws ec2 create-key-pair --key-name EC2-key --query 'KeyMaterial' --output text > EC2-key.pem`

Verify the keypair was successfully created.   
`cat E2-key.pem`

Change the permissions.  
`sudo chmod 600 E2-key.pem` 

## Create an EC2 instance  
**Create a t2.micro instance type:**  
`aws ec2 run-instances --image-id ami-0e6d2e8684d4ccb3e --security-group-ids sg-0c28afaf2c775e906 --instance-type t2.micro --key-name EC2-key`  

Note: This will take several minutes to create an instance.  

**Obtain public IP address of the instance and connect to the instance:**  
`aws ec2 describe-instances --instance-ids i-0e69629cb2642f8e7 --query 'Reservations[0].Instances[0].PublicIpAddress'`

**Use the public ip to connect to the newly created EC2 instance:**  
`ssh -i EC2-key.pem ec2-user@54.237.54.51`  

If the following text below is displayed, then successfully accessed the EC2 instance:  
```

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
55 package(s) needed for security, out of 143 available
Run "sudo yum update" to apply all updates.
```

Note: If you need to terminate your EC2 instance at any time, simply run these commands:  
```
aws ec2 stop-instances --instance-ids i-0e69629cb2642f8e7 
aws ec2 terminate-instances --instance-ids i-0e69629cb2642f8e7 
```

## Host a simple static page on our newly created EC2 instance.   
**Update packages and install NGINX:**  
```
sudo yum update
sudo amazon-linux-extras install nginx1
sudo systemctl start nginx
sudo systemctl enable nginx
```
Navigate to your web browser and type in the EC2 instance's public IP address to see if NGINX works and if it does, you should see the NGINX message.

**Set up NGINX server blocks:**  
```
sudo mkdir -p /var/www/example.com/public_html
sudo vi /var/www/example.com/public_html/index.html
```
Put placeholder HTML code:  
`<h1>Fortune-of-the-Day coming soon</h1>`

Change permissions of the newly created `index.html` file:   
`sudo chown -R nginx: /var/www/example.com`

**Create NGINX config file:**  
`sudo vi /etc/nginx/conf.d/example.com.conf`

You should input this in the file:
```
server {
    listen 80;
    listen [::]:80;

    root /var/www/example.com/public_html;

    index index.html;

    server_name ec2-18-234-134-192.compute-1.amazonaws.com;

    access_log /var/log/nginx/example-one.com.access.log;
    error_log /var/log/nginx/example-one.com.error.log;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

**Check configuration and restart NGINX:**
```
sudo nginx -t 
sudo systemctl restart nginx
```

If you did everything correctly, it should output your HTML file that you
created.

## Take a snapshot of VM, delete VM, and deploy new VM from snapshot

**Get ID of your volume:**  
`aws ec2 describe-instances --instance-id i-0e69629cb2642f8e7 | grep vol`

**Create the snapshot:**  
`aws ec2 create-snapshot --volume-id vol-0b75debd2e1a2bd3f --description "static NGINX server"`

Note: Make sure you write down the SnapshotID.

(Optional) Delete the old EC2 instance:  

```
aws ec2 stop-instances --instance-ids i-037f94e41eca40983 
aws ec2 terminate-instances --instance-ids i-037f94e41eca40983   
```

**Create a new image based on the snapshot we just created:**  
`aws ec2 register-image --name "nginxStatic" --region=us-east-1 --description "AMI from snapshot EBS" --block-device-mappings DeviceName="/dev/sda",Ebs={SnapshotId="snap-0753fa2dadbf4eb94"} --root-device-name "/dev/sda1"`

Note: Make sure you write down the ImageID

**Deploy a new EC2 instance from the volume that we just created:**  
`aws ec2 run-instances --image-id ami-05b2a7a22fe19b0f0 --security-group-ids sg-0c28afaf2c775e906 --instance-type t2.micro --key-name EC2-key`

Note: Make sure you write down the public IP address

Check if your newly created instance works:  
`curl ec2-18-234-134-192.compute-1.amazonaws.com`

### You now have a EC2 instance along with an image to deploy the same EC2 instance whenever you want 
