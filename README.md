# Docker

## What is Docker? 

Docker is a platform which allows you to containerise your applications into small standardised bundles with everything that a software application needs to operate. Containerisation allows for much faster deployment compared to VM's and can be deployed onto any environment.

## Virtual Machines vs Containers

Virtual machines (VM) can also be used to create identical environments however they come with a much larger overhead of needing to spin up a new operating system with each VM. Containers share the same operating system as the host machine allowing for a much lower impact on the host operating system (compared to hosting a whole new operating system for each application). It essentially replaces each VM OS with the 'Docker engine'.If a container needs an OS kernel different to the host operating system then a VM will be required to run the container. 

## How can we use Docker?

Definitions:
- Images are templates/blueprints storing all of the configuration containing the code and tools/runtimes required to run an application.
- Containers are the running unit of software itself.

We can utilise Docker by creating images and then running containers based off of images. 

### Images

Images are read only and utilise layers when building images. It does this by detecting changes in the code which is being built (if no changes then it utilises a caching feature to allow a faster build). Each instruction in a Dockerfile adds a layer. We construct images using a Dockerfile. 

Below is a dummy Dockerfile used for creating a basic node application.

```
FROM node:latest          # builds on top of the latest node image 

WORKDIR /app              # sets the working directory within the container

COPY package.json .       # copies the package.json into the container's working directory

RUN npm install           # runs npm install (used to install node packages)

COPY . .                  # copies the rest of the files

EXPOSE 3000               # used for documentation purposes so the correct port can be exposed (doesn't actually do anything)

CMD ["node","app.js"]     # the command needed to start the container (is only executed when starting a container based on this image)
```

You should place this Dockerfile into the same directory as your app files on your host machine such that your file structure looks like this:

Dockerfile
package.json
public/
app.js

The structure can be different to this however the paths to the various resources will need to be different within your Dockerfile.

Once you have this Dockerfile then you can run `docker build .`, ensuring that you are in the same directory as the Dockerfile. We can use the `-t` flag to tag the image, since if we do not do this then Docker will assign a random name to the image resulting in our command looking like `docker build -t IMAGE-NAME .`

>The first thing that running Docker build will do is grab the latest node image from Dockerhub (if you do not already have an image of node on your host machine). This image will be a 'bare-bones' image which will just be a software package where node is pre-installed. The `:latest` means that it grabs the latest image. If a different version is needed then you can search Dockerhub for the relevant tag and pull that down instead. 

>It is also important to note the reason for copying package.json into the image before copying all of the files in. The 'package.json' file is the minimum required dependency to run npm install (given that node is pre-installed on the container). This means that if we need to rebuild the image due to a change in the code then the image will not run `npm install` again because of the caching layers mentioned earlier.

Dockerfiles can also take environment variables and arguments, see container section for environment variables.

We can specify arguments within the Dockerfile with the ARG command. This takes a key-value pair such that we can say in our Dockerfile

```
ARG DEFAULT_PORT=80
EXPOSE $DEFAULT_PORT
```

### Containers

Now that we have our image, we can now create a container based on that image. This can be done with the command `docker run IMAGE-NAME`. This will use the base image that we previously defined and then run the command highlighted in the CMD section, aka `node app.js`. It is worth noting that the CMD command is presented as a comma separated list. 

By default running a container will attach to standard input/output/error. So to run in the background we need to use the `-d` detach flag

The containers are also self contained and cannot communicate with your host machine or other machines unless it is exposed and mapped to a port on your host machine. We can read what port is needed to be exposed in the Dockerfile. The flag `-p` can be used to expose a port. In the above example we could do `-p 80:3000` to map port 80 on the host machine to port 3000 on the container.

Alternatively, some containers need to be in interactive mode for them to work. For example when developing a React app if it cannot take stdin/stdout then the container will crash. This means that we need to enter the `-it` to allow this to work. Other use cases would include you needing access to a shell of the container.

Some applications will also need to take environment variables. This can be done either with an ENV line in the Dockerfile or an -e flag within your docker run command. The syntax should be `ENV key=value` or `-e key=value` depending on the approach you take.

Useful commands can be [found here]()

### Volumes

Upon container destruction, all data which may have been stored on the container is destroyed. This is why Volumes are useful.

Volumes can be used to persist data across different containers.

Docker supports 3 different types of container: 
1. Anonymous
2. Named
3. Bind Mounts

Bind mounts are used to map a location from your machine onto the container, this can be good for keeping containers running with the most up to date source code (since the data from your localhost is constantly being streamed). This has its main benefit in development environments. The syntax for a bind mount is `-v /path/to/code:/mount/point`

Named volumes can be used to persist data, for example if we wanted to persist /app/logs directory we could use `-v logs:/app/logs`. This will create a volume on your host machine (in a location abstracted from you) and save these files into this location.

Anonymous volues can be created using `-v /app/temp` where /app/temp is a dummy file endpoint. This data is not saved on your host machine so does not persist. They can be made useful by utilising Docker's order of precedence such that if a volume is specified with a longer file path than a different volume then the longer path takes precedence. 

It can be recognised in the example where a bind mount is targetted at /app however we do not wish to overwrite the /app/node_modules file. Therefore we can attach an anonymous volume to /app/node_modules so that the anonymous volume takes precedence and the bind mount does not alter the target path.

It is also worth noting that it is not neccessary to create volumes in advance since Docker will create them as they are called.

### Networking 

Containers need to be able to communicate with different endpoints to run succesfully (in most cases). Here are some different cases to connect containers to consider.

Case 1: Container to WWW communication (example communicate to an API)

Case 2: Container to Local Host Machine Communication 

Case 3: Container to Container Communication


1. Containers automatically can send requests to the WWW (don't need any special configuration)

2. Replace `localhost` with `host.docker.internal` to use localhost within source code

3. Can use the name of the container within the source code to reference a different container. Docker will resolve it internally.

Unlikes volumes networks need to be created in advance before they can be used. `docker network create NETWORK-NAME`

We can use container networks by adding ``docker run --network NETWORK-NAME`` where all containers can communicate with each other.
Within a Docker network , all containers can communicate with each other and IPs are automatically resolved.

Docker does not replace source code. It simply detects outgoing reqeusts and resolves IP for such requests

### Docker Network Drivers

Docker Networks actually support different kinds of "Drivers" which influence the behavior of the Network.

The default driver is the "bridge" driver - it provides the behavior shown in this module (i.e. Containers can find each other by name if they are in the same Network).

The driver can be set when a Network is created, simply by adding the --driver option.

docker network create --driver bridge my-net
Of course, if you want to use the "bridge" driver, you can simply omit the entire option since "bridge" is the default anyways.

Docker also supports these alternative drivers - though you will use the "bridge" driver in most cases:

host: For standalone containers, isolation between container and host system is removed (i.e. they share localhost as a network)

overlay: Multiple Docker daemons (i.e. Docker running on different machines) are able to connect with each other. Only works in "Swarm" mode which is a dated / almost deprecated way of connecting multiple containers

macvlan: You can set a custom MAC address to a container - this address can then be used for communication with that container

none: All networking is disabled.

Third-party plugins: You can install third-party plugins which then may add all kinds of behaviors and functionalities

The "bridge" driver makes most sense in the vast majority of scenarios.
