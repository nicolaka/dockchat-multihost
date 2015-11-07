#Dockchat + Interlock On Swarm with Multihost Networking

This repo demonstrates Dockchat ( the simple Python+Mongo chat app) with the new multi-host networking. 
Multi-host networking is officially supported with Docker Engine 1.9 and does require vxlan kernel feature.
By using the new multi-host networking, you'll be able to scale your application dynamically across your Swarm cluster.




#Requirements
- Linux Kernel 3.16+
- Docker Swarm Cluster configured with a k/v backend. Detailed example can be found [here](https://github.com/docker/swarm/blob/master/docs/discovery.md#using-consul) 
- Docker Compose >= 1.5



# Deploy

####Step 0: Ensure to set the DOCKER_HOST to point at Swarm master:

`export DOCKER_HOST=<SWARM_MASTER_IP>:<SWARM_MASTER_PORT>`

Note: Specify the IP explicitly (e.g don't use localhost or 127.0.0.1)

####Step 1: Create an Overlay Network called dockchat. Here we're also providing the specific subnet to use.

```
docker network create -d overlay --subnet=10.10.10.0/24 dockchat
234c8de9605136625752c983449dc6582fed4c292e73b28c35d6abfdac871ae0
```

Check that the overlay network was created:

```
docker network ls | grep dockchat
234c8de96051        dockchat                        overlay
```

####Step 2: Deploy with docker-compose.

```
# docker-compose --x-networking up -d
Creating dockchatinterlock_web_1
Creating db
Creating dockchatinterlock_interlock_1
```

Check status:

```
# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                            NAMES
85654f46b432        ehazlett/interlock:latest   "/usr/local/bin/inter"   30 seconds ago      Up 29 seconds       443/tcp, 10.0.0.165:80->80/tcp   ip-10-0-0-165/dockchat-multihost_interlock_1
42cd23b6327a        mongo                       "/entrypoint.sh --sma"   30 seconds ago      Up 30 seconds       27017/tcp                        ip-10-0-0-163/db
91106f5a239e        nicolaka/dockchat           "python webapp.py"       31 seconds ago      Up 30 seconds       10.0.0.164:32770->5000/tcp       ip-10-0-0-164/dockchat-multihost_web_1
```


####Step 3: Ensure your laptop's DNS name can resolve demo.dockchat.com.
Create a new /etc/hosts  on your laptop entry and set it to the public IP address of the instance that `dockchat-multihost_interlock_1` was deployed on. Ensure that that instance/vm can accept traffic on TCP port 80. 

```
vim /etc/hosts

(+) <dockchat-multihost_interlock_1 IP>   demo.dockchat.com
```

####Step 4: Go to `demo.dockchat.com` and check out the app :)



#### Step 5: Scale the Web App.

```
docker-compose --x-networking scale web=8
```

Using Docker Compose `scale` feature you can scale the web app **horizontally** across all the Swarm nodes!

Note: you can also go to demo.dockchat.com/haproxy?stats and check out active instances that are part of the HAProxy.

