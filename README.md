𝐂𝐎𝐍𝐓𝐀𝐈𝐍𝐄𝐑𝐒 𝐂𝐔𝐒𝐓𝐎𝐌 𝐍𝐄𝐓𝐖𝐎𝐑𝐊𝐈𝐍𝐆

𝑨𝑵𝑨𝑳𝑶𝑮𝒀: Given F1 and F2
Imagine  F1  lives in room 124 in a  hotel 🏨. Anytime F2 wants to play a game with F1, he visits the hotel and goes to room 124. It will be difficult for F2 to find F1 if he changes his hotel room, say F1 moves to room 289, or if F2 forgets the hotel room number for F1. The best way F2 do is to keep the name of F1, and ask from the receptionist every time he goes to the hotel rather than using his room number to trace him.

# Why We Need a Custom Network for Containers

𝑹𝑬𝑨𝑳𝑰𝑻𝒀: With the default bridge network, containers can communicate with each other using IP addresses but not with container names. Containers are stateless. That is, their ip addresses can be (are) changed when they die. To enable communication using container names, we need to create a custom (our own) network. In k8s we call it labels.

This works when your docker Dockerfile to build the docker image has ping utility as a dependency. 

Example:
```bash
FROM ubuntu/apache2:2.4-20.04_beta
RUN apt-get update \
    && apt-get install -y iputils-ping \
    && apt-get install -y net-tools\
    && apt-get install jq -y
# Ping utils allow containers to test communication between containers in default and custom networks.


#copy files into html directory 
COPY web/* /var/www/html/

ENTRYPOINT ["apachectl", "-D", "FOREGROUND"]
```
Containers that run in a default network run within a `bridge` network

-1) To see the list of network in your docker containers, run the command
```bash
~ $ docker network list
```
```bash
output:
NETWORK ID     NAME     DRIVER   SCOPE
...3efd6       bridge   bridge   local
...ad953       host     host     local
...71911       none     null     local
```
-2) Pull this image from docker hub 

```bash
$ docker pull wisdom2608/network:v1.0.0
```
-3) Run two containers 

-4) Run the first container called app1
```bash
~$ docker run --rm -d --name app1 -p 8001:80  wisdom2608/network:v1.0.0
```

Run the second container called app2. You can change the images depending on your need
```bash
~$ docker run --rm -d --name app2 -p 8001:80  wisdom2608/network:v1.0.0
``` 
-5) We inspect containers to get their ip addresses.
 
Inspect container *app1* 
```bash
~$ docker inspect app1 | grep -i IPAddress
```
*Output*:
```bash
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```
Inspect container *app2* 

```bash
~$ docker inspect app2 | grep -i IPAddress
```
*Output*:
```bash
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.3",
                    "IPAddress": "172.17.0.3",
```

-6) login to container “app1” and ping *app2*

```bash
~$ docker exec -it  app1 bash
```
Ping container “app2”
```bash
root@551e15def4a3:/# ping 172.17.0.3
```
#           NOTE:  If ping command is not found, it means the Dockerfile for container image does have ping utility 

# NB: 
 a) This container remains at the background after stopping it because it does not have `-d` flag only
 ```bash
~$ docker run -d --name app2 -p 8001:80 wisdom2608/network:v1.0.0
```
 b) This container gets removed from the background after stopping it because it has a `--rm` flag. This means it will be removed after it's been stopped
 
 ```bash
~$ docker run --rm -d --name app2 -p 8001:80 wisdom2608/network:v1.0.0
```
 c) This container will be exited because it does not have the `-d` flag. It's runs on the terminal.
 ```bash
~$ docker run --name app2 -p 8001:80 wisdom2608/network:v1.0.0
```

But this is not the recommended way to ping containers with their ip addresses. Containers are stateless, this means that if a container fails, its Ip address will change and pinging the container with old ip will keep on failing. The best way is to create a custom network for the containers so they can be pinged by their names.

-4) To create your own custom network where your containers can run, run the command
```bash
$ docker network create <your_networ_kname --driver bridge
Output:
...1079cc1864f2dd7ce0b
```
List the docker network after creating your own custom network
```bash
~ $ docker network list
```
*Output*:
```bash
NETWORK ID     NAME                    DRIVER    SCOPE
...3efd6       bridge                  bridge    local
...7c9836      <your_network_name>     bridge    local
...ead953      host                    host      local
...71911       none                    null      local

```
-5) Create containers within the custom network 

```bash
~ $ docker run --rm -d --name app3 -p 8003:80 --network custom_network wisdom2608/network:v1.0.0

~ $ docker run --rm -d --name app4 -p 8004:80 --network custom_network wisdom2608/network:v1.0.0
```
-6) Inspect containers created in *<your_network>* (custom network)

*Inspect container app3*
```bash
~$ docker inspect app3 | grep -i IPAddress
```
*Output*:
```bash
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.4",
                    "IPAddress": "172.17.0.4",
```

*Inspect container app4*
```bash
~$ docker inspect app2 | grep -i IPAddress
```
*Output*:
```bash
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.5",
                    "IPAddress": "172.17.0.5",
```

-7) login to container *app3* and ping *app4*

```bash
~$ docker exec -it  app3 bash
```
Ping container *“app4”*. Here we ping by container's name and not by ip addresses as we saw in previous steps.
```bash
root@551e15def4a3:/# ping app4
```

8) Connet to container(s) using a custom networks. Here you want to connect to a container runing in a *default* network from *custom* network

i) Connect to *app1* container
```bash
~$ docker network connect <your_custom_network_name> app1

# That is ~$ docker network connect <your_custom_network_name> <container_name>
```
ii) Connect to *app2* container
```bash
~$ docker network connect <your_custom_network_name> app2

# That is ~$ docker network connect <your_custom_network_name> <container_name>
```
That is your custom network has successfully allowed bridge network containers.

9) Inspect *custom* network: Here we'll see different networks and the various containers running within them.
```bash
~$ docker network inspect <your_custom_network_name> 
```
*Output*:

```bash
"Name"
"Id":
"<your_custom_network_name>",
"46b19be696ac236f8edf{61ae6c7ca484e2a04508a0afa49dd95eff654dd1033*,
"Created":
*2024-06-24T04:39:34.138861889z",
"Scope": "local",
"Driver": "bridge",
"EnableIPv6"; false,
"IPAM": 1
"Driver": "default",
"Options": 0),
"Config": [
"Subnet": "172.18.0.0/16",
"Gateway": "172.18.0.1"
* 14 Essential DevOps...
** Continuous Delivery-
4
N. Virginia +
SaikiranPI +
"Internal": false,
"Attachable": false,
"Ingress":
false,
"ConfigFrom":
"Network":
[Alt+S]
"Configonly": false,
"Containers": (
"b3d2cd681977fb356dee2335659196ffda6ada508816b15f8c602d402b53be50*: (
"Name"': "aPP!"4g53741229ecf£74{5838d49eb16b3152a38ed949e9da4caabb570£d457620ef1",
"EndpointID":
"MacAddress":
"02:42:ac:12:00:04",
"IPv4Address":
"172.18.0.4/16",
"IPv6Address":
wn
"e8a0198£092e67caa292a834d3dfd70787481de3ed658c0d4e15d86499171c7c": (
"Name":
"EndpointID":
"1D"1"*Od1 650e5f798debddFb6ed4a58a79661df957c9b3Fac008587608a9f7de5c21a™,
"MacAddress" : "02:42 :c:12:00:03",
"IPv4Address":
"172.18.0.3/16",
"IPv6Address":
"ed19d569ddd09c1b26078cb68ba4547d43757fdd0b75b98130a29230c9a3e2ce*: (
"EndpointID"
"Name": "app3":2512adaa29546d01a4e358a6720c9089440£c2e4e918dfa5970c079F01e5e227™,
"MacAddress":
"02:42 :ac:12:00:02",
"IPv4Address": "172.18.0.2/16",
"IPv6Address":
之
** Continuous Delivery-
4
N. Virginia +
SaikiranPl y
"Options":
"Labels": ()
```
-10) login to container “app3” and ping *app1*

```bash
~$ docker exec -it  app3 bash
```
Ping container *app4*. Here we ping by container's since we already conneted to containers in a default network using custom network
```bash
root@551e15def4a3:/# ping app1
```
We can also log to *app4* and ping app1 or *app2*. It will also work.

# Link youtube: https://youtu.be/dQHWmIoTs4k?si=k_GK1ro7N-Ciz9hj

# PORTAINER
Portainer is the worlds' most popular universal container management platform with more than 650,000 active monthly users. Portainer can be used to manage Docker Standalone, Kubernetes and Docker Swarm environments through a single common interface. It includes a simple GitOps automation engine and a Kube API.

Create a portainer to manage your containers

```bash
docker run -d -p 9000:9000 --name portainer \
  --restart unless-stopped \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```
# Link to portainer documentation: https://hub.docker.com/extensions/portainer/portainer-docker-extension
