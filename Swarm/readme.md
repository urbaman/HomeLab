# Docker Swarm

## Using keepalived for node ingress and dns relaibility

This assumes you have [installed a docker swarm](/0a9b71f35d375d4dba6c1c5aba0045f3)

### Introduction

When one has a docker swarm a container running on any node in the swarm can be accessed using any IP address of any swarm member.

For example if you had a single web server running on port 80, on one node of a swarm you could access the web server with any of the following IP addresses:

- server1-ip:80
- server2-ip:80
- serverN-ip:80  

Because you want to get to the app even if one swarm node is down typically folks use round robin DNS to try each of the IP in sequence, this has the disadvantage of failed requests if the node fails.
This gist show how i chose to implement a single IP and DNS name to improve reachability and consistency

## EDITS 9/28/2023

- Added dockerd health check - keepalived will now move the vip to another node if the dockerd is stopped on a node
- Added swarm node health check - checks for node in Active, if not VIP won't start on node
- Made all weights equal - no idea why i had them unequal one wants the VIP to roam!
- Removed the SMTP stuff, just not needed unless you really really want notifications 

## Install keepalve on all nodes

run the following on each docker node

```bash
sudo apt-get install keepalived
```  

add user `sudo useradd -r -s /sbin/nologin -M keepalived_script` note this is not used yet (i need to figure out how to let it run docker)

## create health check scripts

### To ensure swarm node is active

execute `sudo nano /usr/local/bin/node_active_check.sh`

add the following contents

```bash
#!/bin/bash
docker node ls -f name=$(hostname) | grep Active > /dev/null 2>&1
```

save 

then `sudo chmod 755 /usr/local/bin/node_active_check.sh`

### To ensure swarm node is Ready

execute `sudo nano /usr/local/bin/node_ready_check.sh`

add the following contents

```bash
#!/bin/bash
docker node ls -f name=$(hostname) | grep Ready > /dev/null 2>&1
```

save

then `sudo chmod 755 /usr/local/bin/node_ready_check.sh`

## create the keepalived.conf file on all nodes

```bash
sudo nano /etc/keepalived/keepalived.conf
```

paste in the following

```bash
! Configuration File for keepalived
	
global_defs {
	vrrp_startup_delay 5
	enable_script_security
    	max_auto_priority
    	script_user root 
}

vrrp_track_process track_docker {
      process dockerd
      weight 10
}

vrrp_script node_active_check {
      script "/usr/local/bin/node_active_check.sh"
      interval 2
      timeout 5
      rise 3
      fall 3
}

vrrp_script node_ready_check {
      script "/usr/local/bin/node_ready_check.sh"
      interval 2
      timeout 5
      rise 3
      fall 3
}

vrrp_instance docker_swarm {
	state MASTER
	interface eth0
	virtual_router_id 10
	priority 100
	advert_int 1
	authentication {
		auth_type PASS
		auth_pass 1111
	}
	virtual_ipaddress {
		192.168.1.45/24
	}
	track_process {
        	track_docker
        }
	track_script {
    		node_active_check
   	}
	track_script {
    		node_ready_check
   	}
	
}
```

Note you may want to:

- change the PASS to your preferred password
- change the IP to the IP you want
- change eth0 if your adapter has a different name

Once you have created the file save and exit

Then start the service

```bash
sudo systemctl start keepalived 
sudo systemctl enable  keepalived 
```

## Add a local DNS entry to your internal DNS server

for example

```bash
swarm.mydomain.com A 192.168.1.45
```

use this name when you want to address any container in the swarm 

## Testing

if you want a simple test ping the vip (e.g. 192.168.1.45) and see what happens when you shutdown each of the nodes!
