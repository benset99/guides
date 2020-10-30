# Part of the Learning Azure series
### My attempt at using azure to improve my cloud skills

## Setting up a resource group and creating a virtual machine

1. Create a resource group

`az group create --location eastus -g learn-azure`

2. Create a virtual machine

`az vm create -g learn-azure --admin-user azuser --image CentOS -n az-vm --generate-ssh-keys`

3. Open port for web traffic

`az vm open-port --port 80 -g learn-azure -n az-vm`

4. Retrieve IP address of newly created virtual machine

`az vm list-ip-addresses -g learn-azure -n az-vm | grep "ipAddress"`

Output should be this: 
`Output: "ipAddress": "104.45.136.140",`

5. Access virtual machine via SSH

`ssh -i ~/.ssh/id_rsa azuser@104.45.136.140`

## Setting up a webserver
1. Update and install necessary packages

```
sudo yum -y update
sudo yum install epel-release 
sudo yum install nginx git 
```

2. Start NGINX

```
sudo systemctl start nginx
sudo systemctl enable nginx
```

3. Check if NGINX works

`curl 104.45.136.140`

It should display a HTML document with text talking about CentOS

4. Make the necessary directory and permissions for our static webpage and config

```
sudo mkdir -p /var/www/azuser/public_html
sudo touch /var/www/azuser/public_html/index.html
sudo chown -R azuser:azuser /var/www/azuser/*
sudo chcon -v -R --type=httpd_sys_content_t /var/www/ 
sudo mkdir /etc/nginx/sites-available
sudo mkdir /etc/nginx/sites-enabled
```

5. Edit NGINX webpage and config files

`vim /var/www/azuser/public_html/index.html`

Input filler text into vim:
```
<h1>Hello World!</h1>

You successfully created a NGINX website!
```

Save & Exit

`sudo vim /etc/nginx/nginx.conf`

Add these lines to the end of the http {} block:
```
include /etc/nginx/sites-enabled/*;
server_names_hash_bucket_size 64;
``` 

**Make sure comment out the server block in this config file!**

Save & Exit

Create a NGINX server block:

`sudo vim /etc/nginx/sites-available/azuser.conf`

```
server {
   listen 80 default_server;
   server_name _;

   root /var/www/azuser/public_html;
   index index.html index.htm;

   location / {
      try_files $uri $uri/ =404;
   }

}
```

Save & Exit

6. Create a symbolic link

`sudo ln -s /etc/nginx/sites-available/azuser.conf /etc/nginx/sites-enabled/azuser.conf`

7. Check if nginx config is working correctly

`sudo nginx -t` 

Output should be this:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

8. Restart NGINX

`sudo service nginx restart`

9. Check if the static webpage works

`curl 104.45.136.140`

Output should be this: 

```
<h1>Hello World!</h1>

You successfully created a NGINX website!
```

10. **That's it! You now have a working static webpage!**
