𝐂𝐎𝐍𝐓𝐀𝐈𝐍𝐄𝐑𝐒 𝐂𝐔𝐒𝐓𝐎𝐌 𝐍𝐄𝐓𝐖𝐎𝐑𝐊𝐈𝐍𝐆

𝑨𝑵𝑨𝑳𝑶𝑮𝒀: Given F1 and F2
Imagine  F1  lives in room 124 in a  hotel 🏨. Anytime F2 wants to play a game with F1, he visits the hotel and goes to room 124. It will be difficult for F2 to find F1 if he changes his hotel room, say F1 moves to room 289, or if F2 forgets the hotel room number for F1. The best way F2 do is to keep the name of F1, and ask from the receptionist every time he goes to the hotel rather than using his room number to trace him.

𝑹𝑬𝑨𝑳𝑰𝑻𝒀: We do not use IP addresses to ping containers because containers are stateless. Rather we use containers names. In k8s we call it labels.

Containers that run in a default network run within a `bridge` network
To see the list of network in your docker containers, run the command
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
-1) Run two containers 

Run the first container called app1
```bash
~$ docker run --rm -d --name app1 -p 8001:80  <image:v1>
```

Run the second container called app2. You can change the images depending on your need
```bash
~$ docker run --rm -d --name app2 -p 8001:80  <image:v1>
``` 
We inspect containers to get their zip addresses.

-2) inspect containers 

Inspect container “app1” 
```bash
~$ docker inspect app1 | grep -i IPAddress
```
Output
```bash
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.2",
                    "IPAddress": "172.17.0.2",
```

# Inspect container “app2” 
```bash
~$ docker inspect app2 | grep -i IPAddress
```
Output:
```bash
            "SecondaryIPAddresses": null,
            "IPAddress": "172.17.0.3",
                    "IPAddress": "172.17.0.3",
```

-3) login to container “app1” and ping "app2"

```bash
~$ docker exec -it  app1 bash
```
Ping container “app2”
```bash
root@551e15def4a3:/# ping 172.17.0.3
```
# NB: 
 a) This container remains at the background after stopping it because it does not have `-d` flag only
 ```bash
~$ docker run -d --name app2 -p 8001:80  <image:v1>
```
 b) This container gets removed from the background after stopping it because it has a `--rm` flag. This means it will be removed after it's been stopped
 ```bash
~$ docker run --rm -d --name app2 -p 8001:80 <image:v1>
```
 c) This container will be exited because it does not have the `-d` flag. It's runs on the terminal.
 ```bash
~$ docker run --name app2 -p 8001:80  <image:v1>
```

But this is not the recommended way to ping containers with their ip addresses. Containers are stateless, this means that if a container fails, its Ip address will change and pinging the container with old ip will keep on failing. The best way is to create a custom network for the containers so they can be pinged by their names.

-4) To create your own custom network where your containers can run, run the command
```bash
$ docker network create <your_networ_kname> --driver bridge
Output:
...1079cc1864f2dd7ce0b
```
List the docker network after creating your own custom network
```bash
~ $ docker network list
```
Output:
```bash
NETWORK ID     NAME                    DRIVER    SCOPE
...3efd6       bridge                  bridge    local
...7c9836      <your_network_name>     bridge    local
...ead953      host                    host      local
...71911       none                    null      local

```
-5) Create containers within the custom network 

```bash
~ $ docker run --rm -d --name app3 -p 8003:80 --network custom_network <image:v1>

~ $ docker run --rm -d --name app4 -p 8004:80 --network custom_network <image:v1>
```

# Link youtube: https://youtu.be/dQHWmIoTs4k?si=k_GK1ro7N-Ciz9hj
