# Part of the Learning Azure series
### My attempt at using azure to improve my cloud skills

## Creating a container in Azure

1. Create a container 

`az container create -g learn-azure -n nginx-cntr --image nginx --ip-address Public`

2. Get IP of the container

`az container show -g learn-azure -n nginx-cntr --query ipAddress.ip --output tsv`

3. Acces the container 

`az container exec -g learn-azure -n nginx-cntr --exec-command /bin/bash`

4. Install some packages that will help us create a static webpage. 

`apt-get update && apt-get install vim`

5. Make static test page. 

`vim ~/index.html`

```
<h1>Hello World!</h1>

You successfully created a NGINX website!
```

6. Replace it with the default NGINX test page.

`cp ~/index.html /usr/share/nginx/html`

7. Check if the static page works 

`curl 52.226.99.4`

Output should be this: 
```
<h1>Hello World!</h1>

You successfully created a NGINX website!
```

10. **That's it! You now have a working static webpage running in a container**
