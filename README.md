# Create a docker based Ocserv server

## Server side configuration

0. This assume you are running in root

1. Install docker service on your Linux server
 
 See https://docs.docker.com/engine/installation/linux/
 
2. Download the source code. This is necessary as the source repo contains some admin tools under `tools/` for host machine.

  `cd ~;git clone https://github.com/clockfly/ocserv-docker.git`. 

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
   yum install gnutls-utils
   
   chmod +x tool/*
   
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
   
8. Start the ocserver daemon

   `docker run -d --privileged -v $(pwd)/etc:/etc/ocserv -p 443:443/tcp -p 443:443/udp seanzhong/ocserv-docker`
   
   you can also put your configuration inside docker compose file the run it
   
   `docker compose up -d`
   
   `docker compose ps`
      
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
  
2. Download the Cisco AnyConnect from https://github.com/clockfly/eduvpn_web/tree/master/files
  
3. Enter the VPN server IP and make the connection
  
4. Done.  

## More notes:

Upstream doc: 

https://github.com/wppurking/ocserv-docker 

and

https://github.com/clockfly/ocserv-docker
