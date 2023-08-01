# Firewall Setup and Configuration
This Documentation is for the installation and configuration of firewalld in Debian Linux

## Installation 
### Install nftables 
```
sudo apt install nftables
 ```
### Start nftables
```
sudo systemctl enable nftables         
sudo systemctl start nftables
```
### Install firewalld 
``` 
sudo apt install firewalld
```
### Start firewalld
```  
sudo systemctl enable firewalld       
sudo systemctl start firewalls
```

At this stage firewalld should be running on your device you can check it with ```sudo systemctl status firewalld``` which will show a output like the one below.   

<img width="384" alt="image" src="https://github.com/Krishks369/Luganoes_devops/assets/71367204/03f89245-c09f-49a5-87c8-672058fe044e">   


## Configure firewalld  
Edit the ``` /etc/firewalld/firewalld.conf``` file and set the **firewallbackend** parameter to nftables   

``` 
backend = nftables
 ```
This confirms that firewalld uses nftables instead of iptables or any others

## Configure the zones and firewall rules
First let us see what are the zones that are already present

```
firewall-cmd --get-zones
```
The zones that are already available are ```block dmz drop external home internal nm-shared public trusted work```  

### Let us configure the **trusted**, **internal**, **public** zones and test the firewall

***Trusted Zone**

Current configuration of the zone can be known by
```
sudo firewall-cmd --zone=trusted --list-all
```
Since trusted zone is the most protected one there is no permissions given to it. So only IPs of certain subnet that have permission to enter this zone are configured using the SSH
```
sudo firewall-cmd --zone=trusted --add-service=ssh
```
Now only certain computers can access the trusted zone  

**Internal Zone** 

To know the current configuration the command is  
```
sudo firewall-cmd --zone=internal --list-all
```
<img width="347" alt="image" src="https://github.com/Krishks369/Luganoes_devops/assets/71367204/ba49a94c-aa54-4c0d-bbeb-81babe3f707d">  

As you may see there are many services that has been given access to internal zone, I would like to remove samba-client from it, so for that the command is 
```
sudo firewall-cmd --zone=internal --remove-service=samba-client
```
<img width="296" alt="image" src="https://github.com/Krishks369/Luganoes_devops/assets/71367204/c7b8f248-a031-4d8b-8389-eedf1aaaa299">  

Now the internal zone is configured to allow only dhcp, dns, ssh

**Public Zone**

Current public zone config
```
sudo firewall-cmd --zone=public --list-all
```
<img width="367" alt="image" src="https://github.com/Krishks369/Luganoes_devops/assets/71367204/fc2c4aad-3983-4738-b681-a6c1f1a983c9">     

Blocking all access to public zone from outside is appropriate for now. So removing all the allowed services
```
sudo firewall-cmd --zone=public --remove-service=dhcpv6-client
```
```
sudo firewall-cmd --zone=public --remove-service=ssh
```
## Testing the Firewall 
## Add Rich Rules 

