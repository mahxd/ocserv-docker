# Create a docker based Ocserv server

## Server side configuration

0. This assume you are running in root

1. Install docker service on your Linux server
 
 See https://docs.docker.com/engine/installation/linux/
 
2. Download the source code. This is necessary as the source repo contains some admin tools under `tools/` for host machine.

  `cd ~;git clone https://github.com/mahxd/ocserv-docker.git`. 

  Before this, you need to install git first if you don't have git installed
  ```
  #debian based: 
  apt install git 
  ```
  ```
  #redhat based:
  yum install git  
  ```
  
3. Change current working directory to "ocserv-docker"
  `cd ocserv-docker`
  
4. Generate the root CA certifate if you don't have one by:
   ```
   #debian based:
   apt install gnutls-bin
   
   #redhat based:
   yum install gnutls-utils
   
   chmod +x tools/*
   
   tools/create-root-certificates
   ```

   It creates root certificate under etc/certs/
   
5. Generate the server certifate for current VPN server:

   `tools/create-server-certificates <Enter your VPN server IP address>`

6. Customize the configuration etc/ocserv.conf
   
7. open ports using your os firewall like firewalld (firewall-cmd redhat bases) or ufw (debian based) if you don't want to enable firewall ignore this part.
	```
	#debian based
	ufw allow 443
	```
   	```
	#redhat based
	firewall-cmd --add-port=443/tcp --permanent 
	firewall-cmd --add-port=443/udp --permanent
	firewall-cmd --reload
	firewall-cmd --state
	```
    
   OR Config ip tables (following open ssh,https port and icmp annd block other ports not recommended for regulare users) :
	```
	iptables -A INPUT -p tcp --dport 22 -j ACCEPT  
	iptables -A INPUT -i lo -j ACCEPT
	iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP
	iptables -A INPUT -p tcp ! --syn -m state --state NEW -j DROP
	iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP
	iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
	iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
	iptables -A INPUT -p tcp --dport 443 -j ACCEPT
	iptables -A INPUT -p udp --dport 443 -j ACCEPT
	iptables -P INPUT DROP
	iptables -P OUTPUT ACCEPT
	iptables -P FORWARD ACCEPT
	```
    After this, please restart the docker service to generate additional iptables rules for docker.
    
	`systemctl restart docker`
	
    For more info on iptables config, please check https://www.vultr.com/docs/setup-iptables-firewall-on-centos-6    
   
8. Start the ocserver using docker compose (recommended)

   ```
   docker compose pull
   
   docker compose up -d
   
   docker compose ps
   ```
   Or using docker command

   `docker run -d --privileged -v $(pwd)/etc:/etc/ocserv -p 443:443/tcp -p 443:443/udp seanzhong/ocserv-docker`
         
9. Check whether the service is running by:
   `docker logs <docker container id>`
   
   the docker container id can be found by `docker ps`

10. Check whether the port 443 is serving:
   `netstat -nap | grep 443`
   
  
11. All done.

## Client side configuration

1. Create a user

  This will add an user entry to etc/ocpasswd and save the password to username.password under etc/user/

  ```
  tools/create-user user_name password
  ``` 
    
  NOTE: Please change etc/certs/client.template to meet your demand.
  
2. Download the Cisco AnyConnect or open connect and connect to vpn
```
android:
	https://play.google.com/store/apps/details?id=com.github.digitalsoftwaresolutions.openconnect&hl=en&gl=US 
	https://play.google.com/store/apps/details?id=com.cisco.anyconnect.vpn.android.avf&hl=en&gl=US

IOS:	https://apps.apple.com/de/app/cisco-anyconnect/id1135064690

Windows:https://github.com/openconnect/openconnect-gui/releases/download/v1.5.3/openconnect-gui-1.5.3-win32.exe
	https://openconnect.github.io/openconnect-gui/
	OR any connect
	http://www.hostwaydcs.com/CISCO/AnyConnect/anyconnect-win-4.10.05095-predeploy-k9.zip

linux: ubuntu:  sudo apt install openconnect network-manager-openconnect network-manager-openconnect-gnome
		>> create new vpn connection in network manager 
		>> type cisco anyconnect 
		>> just enter gateway address 
		>> add >> then connect
		
	OR anyconnect
	http://www.hostwaydcs.com/CISCO/AnyConnect/anyconnect-linux64-4.10.05095-predeploy-deb-k9.tar.gz
	http://www.hostwaydcs.com/CISCO/AnyConnect/anyconnect-linux64-4.10.05095-predeploy-k9.tar.gz
	

MAC:	http://www.hostwaydcs.com/CISCO/AnyConnect/anyconnect-macos-4.9.04043-predeploy-k9.dmg
```
  
3. Enter the VPN server IP and make the connection
  
4. Done.  

## More notes:

Upstream doc: 

https://github.com/wppurking/ocserv-docker 

and

https://github.com/clockfly/ocserv-docker
