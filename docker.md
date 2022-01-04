# 1. Some basic management commnad:
## 1.1 running
docker run -it image:latest bash
docker exec -it 22130ae45700[container id] bash
## image delete
delete all images in this mechine:
```bash
docker rmi -f $(docker images -q)
```
delete images which name have a substring "clickhouse":
``` bash
docker images | grep clickhouse | tr -s ' ' | cut -d ' ' -f 3 | xargs docker image rm
```
# 2. Copy file to container(or reverse)
# 3. Start two terminal login to the same container
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
Docker swarm is a native container orchestrator within Docker runtime. 
It share some common CLI commands with *docker*. In this artical, let's 
take a journey to create a swarm cluster of 3 nodes, then:
* create service to check task schedule;
* scale service to add plasticity to the service;
* service rolling update;
*Fasten Seat Belt*, let's go!!
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
ssh login to worker node, execute the *docker swarm join* command above, then check nodes in mamager node:
``` bash
docker node ls

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

Also, you want to know which is your service's working node:
```
docker service ps helloworld

ID             NAME           IMAGE           NODE         DESIRED STATE   CURRENT STATE            ERROR     PORTS
2wxw6bb34qqw   helloworld.1   alpine:latest   jingle-246   Running         Running 17 minutes ago
mr2ia0snv75a   helloworld.2   alpine:latest   jingle-245   Running         Running 18 minutes ago
```
## 5.6 scale the service in the swarm:
The *replicas* of the *helloworld* service is 2. How about scale the *replicas* to 5? Now, let's have a try:
``` bash
sudo docker service scale helloworld=5

helloworld scaled to 5
overall progress: 5 out of 5 tasks
1/5: running   [==================================================>]
2/5: running   [==================================================>]
3/5: running   [==================================================>]
4/5: running   [==================================================>]
5/5: running   [==================================================>]
verify: Service converged
```
The message from the command tell us: 5 replias is running... :)
Now we do a small check:
``` bash
sudo docker service ps helloworld
ID             NAME           IMAGE           NODE             DESIRED STATE   CURRENT STATE            ERROR     PORTS
2wxw6bb34qqw   helloworld.1   alpine:latest   jingle-246       Running         Running 28 minutes ago
mr2ia0snv75a   helloworld.2   alpine:latest   jingle-245       Running         Running 28 minutes ago
8l2p0bqkklhk   helloworld.3   alpine:latest   jingle-nas-221   Running         Running 2 minutes ago
xev90iwym0h7   helloworld.4   alpine:latest   jingle-245       Running         Running 2 minutes ago
vtb5advbo1hh   helloworld.5   alpine:latest   jingle-nas-221   Running         Running 2 minutes ago
```
## 5.7 delete the service in the swarm:
```
sudo docker service rm helloworld
helloworld
```
Now inspect the *helloworld* service:
``` bash
 sudo docker service inspect  helloworld
[]
Status: Error: no such service: helloworld, Code: 1
```

## 5.8 do rolling updates to a service:
``` bash
docker service create \
>   --replicas 3 \
>   --name redis \
>   --update-delay 10s \
>   redis:3.0.6

rferm7cryv93y6bt01farwtn0
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged
```
*--update-delay* is the time iterval between two task update.

Now, let's inspect the redis service:
``` bash
 docker service inspect --pretty redis

ID:             rferm7cryv93y6bt01farwtn0
Name:           redis
Service Mode:   Replicated
 Replicas:      3
Placement:
UpdateConfig:
 Parallelism:   1
 Delay:         10s
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
 Image:         redis:3.0.6@sha256:6a692a76c2081888b589e26e6ec835743119fe453d67ecf03df7de5b73d69842
 Init:          false
Resources:
Endpoint Mode:  vip
```
We can find three messages here:
1. the *replcas* is 3;
2. delay time of *UpdateConfig* is 10s, and when update failed, the update process will be paused;
3. now the image is in 3.0.6 version

Now do a rolling update to v3.0.7:
```
 docker service update --image redis:3.0.7 redis

redis
overall progress: 3 out of 3 tasks
1/3: running   [==================================================>]
2/3: running   [==================================================>]
3/3: running   [==================================================>]
verify: Service converged
```
check the service again:
``` bash
docker service inspect --pretty redis

ID:             rferm7cryv93y6bt01farwtn0
Name:           redis
Service Mode:   Replicated
 Replicas:      3
UpdateStatus:
 State:         completed
 Started:       6 minutes ago
 Completed:     2 minutes ago
 Message:       update completed
Placement:
UpdateConfig:
 Parallelism:   1
 Delay:         10s
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
 Image:         redis:3.0.7@sha256:730b765df9fe96af414da64a2b67f3a5f70b8fd13a31e5096fee4807ed802e20
 Init:          false
Resources:
Endpoint Mode:  vip
```
Ha！Look at the *UpdateStatus*, they tell the update result!
Use service ps get the update trace:
``` bash
sudo docker service ps redis
ID             NAME          IMAGE         NODE             DESIRED STATE   CURRENT STATE            ERROR     PORTS
mwf4b47w6lgm   redis.1       redis:3.0.7   jingle-245       Running         Running 5 minutes ago
xh6j6yd2blsu    \_ redis.1   redis:3.0.6   jingle-245       Shutdown        Shutdown 5 minutes ago
wcv1n4d5trgk   redis.2       redis:3.0.7   jingle-246       Running         Running 5 minutes ago
skb5fstmv1x5    \_ redis.2   redis:3.0.6   jingle-246       Shutdown        Shutdown 7 minutes ago
jmfhokb3rbpl   redis.3       redis:3.0.7   jingle-nas-221   Running         Running 8 minutes ago
ru8ehvz3uzeb    \_ redis.3   redis:3.0.6   jingle-nas-221   Shutdown        Shutdown 8 minutes ago
```

## 5.9 drain a node in swarm:
docker node update --availability drain ca5a7h8zrfme0zrtmy588p2xo
# 6. Deploy a stack to a swarm
## 6.1 Setup a docker registry on the swarm:

# 7. How docker manage storage?
# 8. Build out a docker image then upload to Docker Hub.
Build and push a container image to Docker Hub from your computer
Start by creating a Dockerfile to specify your application as shown below:

```
# syntax=docker/dockerfile:1
FROM busybox
CMD echo "Hello world! This is my first Docker image."

```
* Run docker build -t <your_username>/my-private-repo . to build your Docker image.
* Run docker run <your_username>/my-private-repo to test your Docker image locally.
* Run docker push <your_username>/my-private-repo to push your Docker image to Docker Hub. You should see output similar to:
