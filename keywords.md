# Managing Images and Containers 

>Can use --help to see all options when using a docker command

### Viewing Running Containers

docker ps = shows all currently running containers

docker ps -a = shows all containers (incl stopped)

### Stopping & Restarting Containers

docker run - creates a new container (attached mode is default)

docker start - Starts a stopped container (detached mode is default)

docker stop - Stops a container

### Interacting with Container

docker run -i (interactive mode)-t (allocates a pseudo TTY)

docker start -i -a 

### Deleting Images and Containers

docker rm - removes containers (stopped only)

docker images - lists images

docker rmi - removes images

Images belonging to a container cannot be removed

docker prune - will remove all unused images

docker run --rm Automatically removes the container when it exits

### Inspecting Images/Containers

docker logs - can see the previous console outputs

docker image inspect - gives lots of information about the image

### Copying files into and from a container

docker cp  (local_source) (container):(dest_path) 
or 
docker cp  (container):(source) (local_dest_path) 

### Naming Images and Containers

docker run --name  - gives custom name to the container

tagging consists of the name and a tag such that the format is name:tag
defines a group of images, tags can be used to specify a version. Combined gives a unique identifier

docker build -t goals:latest . 
