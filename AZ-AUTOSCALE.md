# Part of the Learning Azure series
### My attempt at using azure to improve my cloud skills

## Deploying virtual machine image as an autoscaling group
### Note: This guide assumes that you followed the NGINX.md guide to make a virtual machine.

(Optional) Delete previous virtual machines

```
az vm delete -g learn-azure -n az-vm2
az vm delete -g learn-azure -n az-vm 
```

1. Create virtual machine using virtual machine image

`az vmss create -g learn-azure -n az-vmss --image az-image --upgrade-policy-mode automatic --instance-count 2 --admin-username azuser --ssh-key-values ~/.ssh/id_rsa.pub`

3. Create autoscale profiles and rules

```
az monitor autoscale create -g learn-azure --resource az-vmss --resource-type Microsoft.Compute/virtualMachineScaleSets -n autoscale --min-count 2 --max-count 4 --count 2 

az monitor autoscale rule create -g learn-azure --autoscale-name autoscale --condition "Percentage CPU > 70 avg 5m" --scale out 2

az monitor autoscale rule create -g learn-azure --autoscale-name autoscale --condition "Percentage CPU < 30 avg 5m" --scale in 0 
```

4. Retrieve IP of our virtual machine scale set.

`az vmss list-instance-connection-info -g learn-azure -n az-vmss`

Output should be this:
```
{
  "instance 1": "104.41.136.104:50001",
  "instance 2": "104.41.136.104:50002"
}
```

5. Connect to one of the virtual machine via SSH

`ssh -i ~/.ssh/id_rsa azuser@104.41.136.104 -p 50001`

6. Install and run stress utility to generate CPU load

```
sudo yum -y update
sudo yum -y install stress
sudo stress --cpu 10 --timeout 500 &
```

7. Use `top` to make sure it's working

8. Exit virtual machine and repeat steps `5-7` for the other virtual machine in the scale set

9. Monitor the virtual machine scale set to see if it automatically scales under heavy load.

`watch az vmss list-instances -g learn-azure -n az-vmss --output table`

After a few minutes, it should spin up new virtual machines based on the rules that we created.

Example: 
```
Every 2.0s: az vmss list-instances -g learn-azure -n az-vmss --output table                                                          Fri Oct 30 20:13:00 2020

InstanceId    LatestModelApplied    Location    ModelDefinitionApplied    Name       ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ------------------------  ---------  -------------------  ---------------  ----------------------------------
--
1             True                  eastus      VirtualMachineScaleSet    az-vmss_1  Succeeded            learn-azure      09456dbe-427f-4e44-ae3b-38bd447e72
4a
2             True                  eastus      VirtualMachineScaleSet    az-vmss_2  Succeeded            learn-azure      571fd7de-b5c8-4cb6-96f9-fb83658378
fd
4             True                  eastus      VirtualMachineScaleSet    az-vmss_4  Creating             learn-azure      638607d1-c8b4-49af-9970-331b67be62
db
5             True                  eastus      VirtualMachineScaleSet    az-vmss_5  Creating             learn-azure      c490d0a7-a45e-4130-b8b9-9d12df9b1f
a3
6             True                  eastus      VirtualMachineScaleSet    az-vmss_6  Creating             learn-azure      a6cd3d05-fbfa-4f6e-92ab-03029c1450
01
```

10. Find loadbalancer name and create a load balancer rule to test if our webpage works

```
az network lb list -g learn-azure | grep 'LB",'

az network lb rule create \
-g learn-azure \
--name HTTPrule \
--lb-name az-vmssLBBEPool \
--protocol tcp \
--frontend-port 80 \
--backend-port 80 \ 
--probe-name HealthProbe \ 
--disable-outbound-snat true \ 
--idle-timeout 15 
```

11. 
