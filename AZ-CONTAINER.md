# Part of the Learning Azure series
### My attempt at using azure to improve my cloud skills

## Creating a container in Azure

1. Create a container 

`az container create -g learn-azure -n nginx-cntr --image nginx --ip-address Public`

2. Acces the container 

`az container exec -g learn-azure -n nginx-cntr --exec-command /bin/bash`

3. Install some packages that will help us create a static webpage. 

`apt-get update && apt-get install vim`

4. Get IP of the container

`az container show -g learn-azure -n nginx-cntr --query ipAddress.ip --output tsv`


