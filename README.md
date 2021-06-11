# How to setup a Linux VM with Apache and PHP using cloud-init on Azure

> ✔️ I've wrote two more articles covering the usage of cloud-init. You can see the other two here:
>
> 1. [https://github.com/ricmmartins/linuxvm-cloud-init-loadbalancer/](https://github.com/ricmmartins/linuxvm-cloud-init-loadbalancer/)
> 2. [https://github.com/ricmmartins/linuxvm-cloud-init-vmss](https://github.com/ricmmartins/linuxvm-cloud-init-vmss)

In this article let's see how to setup a Linux VM on Azure with Apache, PHP and a single application using Azure-CLI and cloud-init.

Cloud-init is a widely used approach to customize a Linux VM as it boots for the first time. You can use cloud-init to install packages and write files, or to configure users and security. As cloud-init runs during the initial boot process, there are no additional steps or required agents to apply your configuration.

In other clouds, this concept is often referred to as user data. In Microsoft Azure, we have a similar feature called custom data. Custom data is only made available to the VM during first boot/initial setup, we call this 'provisioning'. Provisioning is the process where VM Create parameters (for example, hostname, username, password, certificates, custom data, keys etc.) are made available to the VM and a provisioning agent processes them, such as the [Linux Agent](https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/agent-linux) and [cloud-init](https://cloudinit.readthedocs.io/).

I recommend to do the next steps using the Azure Cloud Shell directly from Azure Portal. 

![cloud-shell.png](/cloud-shell.png)

✔️ Remember to set Bash as environment.

## Creating the cloud-init.txt

First of all, create a file called cloud-init.txt and insert the following content:

```
#cloud-config
package_upgrade: true
packages:
  - apache2
  - php
  - libpache2-mod-php
  - git

runcmd:
  - cd "/usr/share" && git clone https://github.com/ricmmartins/simple-php-app.git
  - mv /var/www/html /var/www/html-bkp
  - ln -s /usr/share/simple-php-app/ /var/www/html
```
✔️ You can use vim, or simple type "code" to open a VS Code inside the cloud shell.

This will install the packages:

* Apache2
* PHP
* LibApache2-Mod-PHP
* Git

And after will:

* Clone the Github repo [https://github.com/ricmmartins/simple-php-app.git](https://github.com/ricmmartins/simple-php-app.git) under /usr/share
* Move the content of /var/www/html to /var/www/html-bkp
* Create a symbolic link from the content of /usr/share/simple-php-app on /var/www/html

## Create a resource group

```
az group create --name myResourceGroup --location eastus
```

## Create a virtual machine

```
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --custom-data cloud-init.txt \
    --generate-ssh-keys
```

## Open port 80 for web traffic

```
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

## Get the VM Public IP

```
az network public-ip show \
    --resource-group myResourceGroup \
    --name myVMPublicIP \
    --query ipAddress \
    --output tsv
 ```
 
## Test access 

![access.png](/access.png)

## References

-  [https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-cloud-init](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/using-cloud-init)
-  [https://docs.microsoft.com/en-us/azure/virtual-machines/custom-data](https://docs.microsoft.com/en-us/azure/virtual-machines/custom-data)
-  [https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-automate-vm-deployment](https://docs.microsoft.com/en-us/azure/virtual-machines/linux/tutorial-automate-vm-deployment)
