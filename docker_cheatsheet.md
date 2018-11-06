# Docker Cheat Sheet
Kerry Gougeon's Docker cheat sheet

- [Docker Cheat Sheet](#docker-cheat-sheet)
  - [Orientation](#orientation)
  - [Containers](#containers)
    - [Dockerfile](#dockerfile)
    - [List of Commands I](#list-of-commands-i)
  - [Services](#services)
    - [Docker-Compose](#docker-compose)
    - [List of Commands II](#list-of-commands-ii)
  - [Swarm](#swarm)
    - [List of Commands III](#list-of-commands-iii)

## Orientation

```sh
# List Docker CLI commands
docker
docker container --help

# Display Docker version and info
docker --version
docker version
docker info
```

## Containers

### Dockerfile 

You first create an empty directory, and then create a file called Dockerfile. 

Here's an example of a Dockerfile

```dockerfile
FROM python:2.7-slim        # Use an official Python runtime as a parent image

WORKDIR /app                               # Set the working directory to /app

COPY . /app   # Copy the current directory contents into the container at /app

RUN {commands}                             # Run any commands in the container

EXPOSE 80         # Make port 80 available to the world outside this container

ENV NAME World                                   # Define environment variable

CMD ["python", "app.py"]              # Run app.py when the container launches
```


### List of Commands I

```bash
docker build -t friendlyhello . # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello        # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

## Services

### Docker-Compose

docker-compose needs a docker-compose.yml file that defines how Docker containers should behaves in prod. 

Here's an example of a docker-compose.yml:

```yml
version: "3"
services:
  web:
    image: username/repo:tag
    deploy:
      replicas: 5                   # Run 5 instances of that images as "web"
      resources:
        limits:                     # Limit at most 10% of CPU and 50MB of RAM
          cpus: "0.1"
          memory: 50M
      restart_policy:               # Immediately restart containers if one fails
        condition: on-failure
    ports:
      - "4000:80"                   # Map port 4000 on the host to web's port 80
    networks:                       
      - webnet                      # Web containers should share prot 80 via load-balanced network called webnet
networks:                           
  webnet:                           # Define the webnet network with the default settings
```

### List of Commands II

```sh
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
docker swarm leave --force      # Take down a single node swarm from the manager
```

## Swarm 

A swarm is a group of machines that are running Docker and joined into a cluster. After that has happened, you continue to run the Docker commands you’re used to, but now they are executed on a cluster by a swarm manager. The machines in a swarm can be physical or virtual. After joining a swarm, they are referred to as nodes. 

A swarm can have different strategies to run containers. 

Swarm mode can be enabled, and enabling it will instantly switch the current machine to a swam manager

### List of Commands III

```sh
docker-machine create --driver virtualbox myvm1 # Create a VM (Mac, Win7, Linux)
docker-machine create -d hyperv --hyperv-virtual-switch "myswitch" myvm1 # Win10
docker-machine env myvm1                # View basic information about your node
docker-machine ssh myvm1 "docker node ls"         # List the nodes in your swarm
docker-machine ssh myvm1 "docker node inspect <node ID>"        # Inspect a node
docker-machine ssh myvm1 "docker swarm join-token -q worker"   # View join token
docker-machine ssh myvm1   # Open an SSH session with the VM; type "exit" to end
docker node ls                # View nodes in swarm (while logged on to manager)
docker-machine ssh myvm2 "docker swarm leave"  # Make the worker leave the swarm
docker-machine ssh myvm1 "docker swarm leave -f" # Make master leave, kill swarm
docker-machine ls # list VMs, asterisk shows which VM this shell is talking to
docker-machine start myvm1            # Start a VM that is currently not running
docker-machine env myvm1      # show environment variables and command for myvm1
eval $(docker-machine env myvm1)         # Mac command to connect shell to myvm1
& "C:\Program Files\Docker\Docker\Resources\bin\docker-machine.exe" env myvm1 | Invoke-Expression   # Windows command to connect shell to myvm1
docker stack deploy -c <file> <app>  # Deploy an app; command shell must be set to talk to manager (myvm1), uses local Compose file
docker-machine scp docker-compose.yml myvm1:~ # Copy file to node's home dir (only required if you use ssh to connect to manager and deploy the app)
docker-machine ssh myvm1 "docker stack deploy -c <file> <app>"   # Deploy an app using ssh (you must have first copied the Compose file to myvm1)
eval $(docker-machine env -u)     # Disconnect shell from VMs, use native docker
docker-machine stop $(docker-machine ls -q)               # Stop all running VMs
docker-machine rm $(docker-machine ls -q) # Delete all VMs and their disk images
```