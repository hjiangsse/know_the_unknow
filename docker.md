# 1. run
docker run -it image:latest bash
docker exec -it 22130ae45700[container id] bash
# 2. copy file to container(or reverse)
# 3. start two terminal login to the same container
docker run -it ubuntu bash
docker exec -it {docker id} bash

# 4. Stop and remove all docker containers and images
## List all containers (only IDs)
docker ps -aq
## Stop all running containers
docker stop $(docker ps -aq)
## Remove all containers
docker rm $(docker ps -aq)
## Remove all images
docker rmi $(docker images -q)

# 5. Docker Swarm
## 5.1 create a swarm on manager node
``` bash
docker swarm init --advertise-addr 192.168.222.221
```
the following message will be generated:
``` bash
docker swarm join --token SWMTKN-1-2qsi0n0s6zz32elnwfbh3invdpguonw3u584z1mqgn601mgpev-30s9ekejekq06tmsjaw6ba5xf 192.168.222.221:2377
```
if you want to retrive the message again:
``` bash
docker swarm join-token worker
```
## 5.2 worker node join
ssh login to worker node, execute the *docker swarm join* command above
## 5.3 check nodes in mamager node
``` bash
docker node ls
```
will list all node in the swarm.
```
ID                            HOSTNAME         STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
uw8cpilef589arfaap272ewh9     jingle-245       Ready     Active                          20.10.7
t8or2nv1x8brtuo6oz71k3id4     jingle-246       Ready     Active                          20.10.7
ca5a7h8zrfme0zrtmy588p2xo *   jingle-nas-221   Ready     Active         Leader           20.10.7
```
## 5.4 deploy a service to the swarm
``` bash
docker service create --replicas 2 --name helloworld alpine ping docker.com
```
output:
``` bash
sx6ye6p8ldogf8muqy2w579kf
overall progress: 2 out of 2 tasks
1/2: running   [==================================================>]
2/2: running   [==================================================>]
verify: Service converged
```
* The docker service create command creates the service.
* The --name flag names the service helloworld.
* The --replicas flag specifies the desired state of 2 running instance.
* The arguments *alpine ping docker.com* define the service as an *Alpine Linux container* that executes the command *ping docker.com*.

check services status:
```bash
$docker service ls
ID             NAME         MODE         REPLICAS   IMAGE           PORTS
sx6ye6p8ldog   helloworld   replicated   2/2        alpine:latest
```
## 5.5 inspect the service you just created:
``` bash 
docker service inspect --pretty helloworld

ID:             sx6ye6p8ldogf8muqy2w579kf
Name:           helloworld
Service Mode:   Replicated
 Replicas:      2
Placement:
UpdateConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Update order:      stop-first
RollbackConfig:
 Parallelism:   1
 On failure:    pause
 Monitoring Period: 5s
 Max failure ratio: 0
 Rollback order:    stop-first
ContainerSpec:
 Image:         alpine:latest@sha256:21a3deaa0d32a8057914f36584b5288d2e5ecc984380bc0118285c70fa8c9300
 Args:          ping docker.com
 Init:          false
Resources:
Endpoint Mode:  vip
```
you can remove the *--pretty*, then the messages will be reported in a json format.
