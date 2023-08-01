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
