# Part of the Learning Azure series
### My attempt at using azure to improve my cloud skill guides

## Create an image of a virtual machine
### Note: This guide assumes that you followed the NGINX.md guide to make a virtual machine.

1. Connect to your virtual machine via SSH

`ssh -i ~/.ssh/id_rsa azuser@104.45.136.140`

2.  Deprovision the virtual machine 

`sudo waagent -deprovision+user`

After this is complete, exit the virtual machine.

3. Create the virtual machine image 

```
az vm deallocate -g learn-azure -n az-vm
az vm generalize -g learn-azure -n az-vm
az image create -g learn-azure -n az-image --source az-vm
```

4. Create a new virtual machine using the image that we just created

`az vm create -g learn-azure -n az-vm2 --image az-image --admin-username azuser2 --ssh-key-values ~/.ssh/id_rsa.pub`

5. Retrieve IP address of newly created virtual machine

`az vm list-ip-addresses -g learn-azure -n az-vm2 | grep "ipAddress"`

Output should be this: 

`Output: "ipAddress": "52.249.188.165",`

6. Open port for web traffic

`az vm open-port --port 80 -g learn-azure -n az-vm2`

7. Check if NGINX works on the virtual machine created from the image

`curl 52.249.188.165`

Output should be this:

```
<h1>Hello World!</h1>

You successfully created a NGINX website!
```

8. **That's it! You now have an image of a NGINX website that you can use to make static pages with ease** 






