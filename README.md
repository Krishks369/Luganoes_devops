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
sudo systemctl start firewalld
```

At this stage firewalld should be running on your device you can check it with ```sudo systemctl status firewalld``` which will show a output like the one below.  


<img width="384" alt="image" src="https://github.com/Krishks369/Luganoes_devops/assets/71367204/03f89245-c09f-49a5-87c8-672058fe044e">   


## Configure firewalld  
Edit the ``` /etc/firewalld/firewalld.conf``` file and set the **firewallbackend** parameter to nftables   

``` 
backend = nftables
 ```
- This confirms that firewalld uses nftables instead of iptables or any others

## Configure the zones and firewall rules
First, let us see what are the zones that are already present

```
firewall-cmd --get-zones
```
- The zones that are already available are ```block dmz drop external home internal nm-shared public trusted work```  

### Let us configure the **trusted**, **internal**, **public** **DMZ** zones ,and test the firewall

## Trusted Zone

Current configuration of the zone can be known by
```
sudo firewall-cmd --zone=trusted --list-all
```
- Since trusted zone is the most protected one there is no permissions given to it. let us assume that only one person is allowed to access via ssh the trusted zone, then that person can be added as source to the zone
```
sudo firewall-cmd --zone=trusted --add-service=ssh
```
Adding the IP of a single computer as source
```  
sudo firewall-cmd --zone=trusted --add-source=192.168.146.1
```

- This is the final config of the trusted zone  


<img width="331" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/9a19cf97-9e02-4171-ae87-6feda1e1a160">   



### Internal Zone

- To know the current configuration the command is  
```
sudo firewall-cmd --zone=internal --list-all
```
<img width="347" alt="image" src="https://github.com/Krishks369/Luganoes_devops/assets/71367204/ba49a94c-aa54-4c0d-bbeb-81babe3f707d"> 


- As you may see many services has been given access to internal zone, I would like to remove samba-client from it, so for that the command is 
```
sudo firewall-cmd --zone=internal --remove-service=samba-client
```
- Adding the subnet of internal network computers as the source so that no intrusion from outside the network happens   
```
sudo firewall-cmd --zone=trusted --add-source=192.168.55.0/24
```
 
<img width="313" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/f986989b-a66a-49c0-8275-c97fe43adbf4">   


Now the internal zone is configured to allow only DHCP, DNS, ssh, and source as the subnet 192.168.55.0/24


### Public Zone

Current public zone config
```
sudo firewall-cmd --zone=public --list-all
```
<img width="367" alt="image" src="https://github.com/Krishks369/Luganoes_devops/assets/71367204/fc2c4aad-3983-4738-b681-a6c1f1a983c9">     

- Blocking all access to the public zone from outside is appropriate for now. So removing all the allowed services
```
sudo firewall-cmd --zone=public --remove-service=dhcpv6-client
```
```
sudo firewall-cmd --zone=public --remove-service=ssh
```

### Active zones
```
sudo firewall-cmd --get-active-zones
```


<img width="317" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/4f0de3cd-3d83-4309-b69e-3fe0d2aaf13e">  

- Now with this basic firewall configuration let us check the trusted network if it allows only the particular IP or not

## Add Rich Rules 
Rich rules are custom-made rules for specific purposes, in our case let us make an alert and drop the request when an unauthorized user SSH into the protected device

```
sudo firewall-cmd --add-rich-rule='rule family=ipv4 service name=ssh log prefix="Unauthorised SSH dropped" level="notice" drop
```
- The above command will be added to the default zone which is public in our case  

<img width="644" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/b2b1ed25-b947-4c5f-bdf3-03762c0e99ff">

### Adding DMZ zone
- DMZ zone is where the network is open to the public, so we can give access to HTTP and HTTPS and add rule to log entry of every user.

```
sudo firewall-cmd --zone=dmz --remove-service=ssh          // remove the SSH service
sudo firewall-cmd --zone=dmz --add-service=http            // add http and https
sudo firewall-cmd --zone=dmz --add-service=https
sudo firewall-cmd --zone=dmz --add-source=0.0.0.0/0         // add source as everything
```
- Rich rule for logging the entry of user
```
sudo firewall-cmd --zone=dmz --add-rich-rule='rule family=ipv4 service name=http log prefix="HTTP Entry" level="notice" limit value="1/m" accept'
```

- Final configuration of DMZ   

<img width="463" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/b19f54ec-00eb-42ad-8696-16fcd5457265">   


## Testing the Firewall 

The reference pictures below may not be clear, please feel free to click on it and see

- Succesfull SSH into the trusted zone with the permitted IP 


<img width="713" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/343477a9-e910-47e1-be83-e856ed549515">  


- Unsuccesfull SSH into the trusted zone with denied IP  


<img width="378" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/f08d7c66-9bd0-473a-92b3-1b98264c282a">

- Denied log is added to the ```/var/log/syslog```  


<img width="466" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/6012cabd-25cf-4c8e-bf83-02e882cb0dc5">

- Performing curl to check if DMZ logs the user entry  


<img width="738" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/1060baa3-0ce0-43f1-8a13-267897a102fe">  

- log 

<img width="470" alt="image" src="https://github.com/Krishks369/Luganodes_devops/assets/71367204/04ed225f-3c67-41df-9745-3318af482077">

## To apply the changes permanently
```
sudo firewall-cmd --runtime-to-permanent
```

Now all the configuration are applied and will be running even after reboot

## Conclusion
This Documentation will help beginners to install and configure firewalld and also will give an idea about how a firewall works.
