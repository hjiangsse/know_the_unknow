# run
docker run -it image:latest bash
docker exec -it 22130ae45700[container id] bash
# copy file to container(or reverse)
# start two terminal login to the same container
docker run -it ubuntu bash
docker exec -it {docker id} bash

# Stop and remove all docker containers and images
## List all containers (only IDs)
docker ps -aq
## Stop all running containers
docker stop $(docker ps -aq)
## Remove all containers
docker rm $(docker ps -aq)
## Remove all images
docker rmi $(docker images -q)
