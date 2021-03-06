---
layout: post
title: Docker Commands Cheat Sheet
permalink: blog/docker-cheat-sheet/
excerpt_separator: <!--more-->
comments: True
---

A summary of commands I have come across while using Docker in my projects.

*Note: if you are using Boot2Docker, then do not type sudo otherwise you will get an error*

## Tips

- You can replace [containerID] with [containerName]. You only typically need 3 characters from [containerID]
- Append --help if you want more information about the command, like `docker run --help`
- Note that Containers are not the same as Images. Containers are like instances.
- If you are using boot2docker, it might cause some issues such as docker server being unresponsive. Restart boot2docker with `boot2docker restart` to fix issues such as DNS and etc.
- Containers are process-driven, so once the process is done, it will exit (this might take a while to get used to).
- to “enter” a container that is currently running in the background as a daemon, run `docker exec -it [container] /bin/bash`

## Commands

### General
---

#### Check Docker Version
`sudo docker version`

### Image Info
---

#### Search For Available Images
`sudo docker search [imageName]`

#### List Locally Saved Images
`sudo docker images`

#### Pull and Cache Images from Remote
`sudo pull [imageName:tag]`

- note that if you do `docker run ubuntu ...` it will run this command if you do not have ubuntu stored locally
- it is recommended to pull with a specific tag

Example

```
# search for ubuntu
sudo docker search ubuntu

# pull ubuntu v14.04
sudo pull ubuntu:14.04
```

<!--more-->

### Modifying an Image
---

Note that to modify an image, you have to create an instance of that image as a container. You must then run the container, then do your modifications then on. Similar to Git, you have to “commit” your changes. This will “save” the changes you made on that container into the origin image. Then you can push it to docker hub.

#### Build an Image
`sudo docker build -t [username/imagename:tag] .`

- note that `.` assumes there is an existing Dockerfile

Example

```
# create docker file
touch Dockerfile

# Sample Dockerfile Contents:
# More on: https://docs.docker.com/reference/builder/
# FROM ubuntu
# RUN \
#   apt-get update && \
#   apt-get install -y python python-dev python-pip python-virtualenv && \
#   rm -rf /var/lib/apt/lists/*
# # working dir
# WORKDIR /data
# # default command
# CMD [“bash”]

sudo docker build -t jancarloviray/python
```

#### Checking Diff
`sudo docker diff [containerID]`

- This shows file changes/additions/deletions

#### Commit Changes to an Image
`sudo docker commit -m ‘Added nodejs’ -a ‘John Doe’ [containerID] [username/new_name:custom_tag]`

#### Setting Tag on Existing Images
`sudo docker tag [containerID] [username/imagename:newtag]`

#### Push Image to Docker Hub
`sudo docker push [username/imagename]`

#### Remove Docker Image
`sudo docker rmi [imageName]`

- note that this is different than `sudo docker rm [containerName]` which removes a container

### Container Information
---

#### List Containers
```bash
# list only running containers
# ----------------------------
sudo docker ps

# list all container
# ------------------
sudo docker ps -a

# show recently created container
# ----------------------------
sudo docker ps -l

# display file sizes (of all containers)
# ----------------------------
sudo docker ps -a -s
```

#### View Container Logs/Outputs
```bash
# View Container Log
# ------------------
sudo docker logs [containerID]

# View and Tail Container Log
# ---------------------------
sudo docker logs -f [containerID]

# Also show Timestamps
# --------------------
sudo docker logs -f -t [containerID]
```

#### Inspect a Container
`sudo docker inspect [containerID]`

### Container Actions
---

#### Run a Command and Activate a Container
`sudo docker run ubuntu /bin/echo ‘Hello world’`

- first, we specified the **Docker** binary and the command we wanted to execute, **run**. Note that the **docker run** combination runs the container.
- Next, we specified an image. If it is not installed, it will download it.
- Next is the command we run inside the container.
- Note that docker containers are only active so long as the command you specify is active.

#### Interactive Container
`sudo docker run -t -i [imagename] /bin/bash`

- **-t** assigns a pseudo-tty or terminal inside our new container
- **-i** flag allows us to make an interactive connection by grabbing the standard in STDIN of the container
- **/bin/bash** also launches a bash shell inside our container

#### Run a Container in Background
`sudo docker run -d [imagename] /bin/sh -c “while true;do echo hello world; sleep 1; done”`

- **-d** tells Docker to run the container and put it in the background, to “daemonize” it
- note that after you run this, you will not see the output but Docker will rather give you the **Container ID**.
- to see running Containers, run `docker ps`
- to peek inside a specific container, run `sudo docker logs [containerIDorName]`
- to stop running the container, `sudo docker stop [containerIDorName]`. Note that there is a default timer of 10 seconds before the command will actually execute - a grace period just in case you want to change your mind. To override this, use `sudo docker stop -t 0 [containerID]`

#### Run and Name a Container
`sudo docker run --name [containerName] [imageName] /bin/echo ‘Hello World’`

#### Execute a Command in a Running Container
`sudo docker exec [containerID] /bin/bash`

#### Attach a Terminal in a Running Container
`sudo docker exec -it [containerID] /bin/bash`

#### Stop a Container Running in Background
`sudo docker stop [containerID]`

#### Restarting a Container
`sudo docker restart [containerID]`

#### Removing a Container
`sudo docker rm [containerID]`

### Container Networking
---

#### Expose Container Ports to Host (ie: run a web app)
`sudo docker run -d -P training/webapp python app.py`

- **-d** runs the container in background
- **-P** tells Docker to map network ports inside our container to our host

When you run `docker ps` you will see a column like this:
```
PORTS
0.0.0.0:49155->5000/tcp
```

This means that Docker has exposed port 5000 (default python flask port) on port 49155.

**NOTE: if you are using boot2docker, you will need to get the ip of boot2docker. Instead of running localhost:49155, you must get the port by running `boot2docker ip` or `open $(boot2docker ip):49155`**

#### Expose Container Port to Host Manually
`sudo docker run -d -p 5000:5000 training/webapp python app.py`

- This maps container port 5000 to host port 5000.
- Note that you can run the **-p** flag multiple times to configure multiple ports

### Managing Container Data

Note that containers persist until you remove them with `docker rm [container]`. Until then, the container (an ‘instance’ of an ‘image’ per se) will continue to exist, and can be started, stopped, restarted and etc.

#### Data Volumes

A *data volume* is a specially-designated directory within one or more containers that bypasses the Union File System to provide the following features:

- Volumes are initialized when a container is created
- Data volumes can be shared and reused between containers
- Changes to a data volume are made directly
- **Changes to a data volume will not be included when you update an image**
- **Volumes persist until no containers use them**

```bash
# Adding a Data Volume
# --------------------
#
# sudo docker run -d -P --name [newName] \
# -v [volumeName] [imageName] [commandToExecute]
#
# Use the `-v` flag with `docker create`
# or `docker run` command
# Note that you can use it multiple times
# to mount multiple data volumes
# The following code will create a new volume
# inside a container at `/webapp`

sudo docker run -d -P --name web -v /webapp training/webapp python appy.py

# Mount a Host Directory as a Data Volume
# ---------------------------------------
#
# sudo docker run -d -P --name [newName] \
# -v [/dir/from/host]:[/dir/in/cont] \
# [imageName] [commandToExecute]
#
# `-v` flag is also used to mount a directory from
# your Docker daemon’s host into a container.
# Note that if you are using Boot2Docker, it only
# has limited access to your /Users directory
# so, you mount it like this:
# `sudo docker run -v /Users/<path>:/<containerPath>`
# The following will mount the host directory,
# /src/webapp into the container at /opt/webapp.
# NOTE: if the path /opt/webapp already exists
# inside the container’s image, its contents will
# be replaced by the contens of /src/webapp on
# the host to stay consistent with the expected
# behavior of `mount`

sudo docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py
```

### Misc Commands

```bash
# quick install docker in linux
curl -sSL https://get.docker.com/ | sh

# kill all running containers
# note that -q stands for quiet, just show ids
sudo docker kill $(docker ps -q)

# delete stopped container
sudo docker rm -v `docker ps -a -q -f status=exited`
```
