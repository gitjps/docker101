# Lab 1 - Running Your First Container

© Copyright IBM Corporation 2017

IBM, the IBM logo and ibm.com are trademarks of International Business Machines Corp., registered in many jurisdictions worldwide. Other product and service names might be trademarks of IBM or other companies. A current list of IBM trademarks is available on the Web at &quot;Copyright and trademark information&quot; at www.ibm.com/legal/copytrade.shtml.

This document is current as of the initial date of publication and may be changed by IBM at any time.

The information contained in these materials is provided for informational purposes only, and is provided AS IS without warranty of any kind, express or implied. IBM shall not be responsible for any damages arising out of the use of, or otherwise related to, these materials. Nothing contained in these materials is intended to, nor shall have the effect of, creating any warranties or representations from IBM or its suppliers or licensors, or altering the terms and conditions of the applicable license agreement governing the use of IBM software. References in these materials to IBM products, programs, or services do not imply that they will be available in all countries in which IBM operates. This information is based on current IBM product plans and strategy, which are subject to change by IBM without notice. Product release dates and/or capabilities referenced in these materials may change at any time at IBM&#39;s sole discretion based on market opportunities or other factors, and are not intended to be a commitment to future product or feature availability in any way.

# Overview

In this lab, you will run your first Docker container. 

Containers are just a process (or a group of processes) running in isolation. Isolation is achieved via linux namespaces and control groups. One thing to note, is that linux namespaces and control groups are features that are built into the linux kernel! Other than the linux kernel itself, there is nothing special about containers.

What makes containers useful is the tooling that surrounds it. For these labs, we will be using Docker, which has been the de facto standard tool for using containers to build applications. Docker provides developers and operators with a friendly interface to build, ship and run containers on any environment.

The first part of this lab, we will run our first container, and learn how to inspect it. We will be able to witness the namespace isolation that we acquire from the linux kernel.

After we run our first container, we will dive into other uses of docker containers. We will find many examples of these on the Docker Store, and we will run several different types of containers on the same host. This will allow us to see the benefit of isolation- where we can run multiple containers on the same host without conflicts.

We will be using a few Docker commands in this lab. For full documentation on available commands check out the [official documentation](https://docs.docker.com/).

## Prerequisites

Completed Lab 0: You must have docker installed,


# Step 1: Run your first container

We are going to use the Docker CLI to run our first container. You can use the [docker commandline reference](https://docs.docker.com/engine/reference/commandline/docker/), to review Docker commands.

1. Open a terminal on your local computer

2. Run the `docker ps -a` command to list all current containers. The `-a` flag shows not only the running containers, but all containers including exited containers. 
```sh
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

3. Run `docker container run -t ubuntu top`

Use the `docker container run` command to run a container with the ubuntu image using the `top` command. The `-t` flags allocate a pseudo-TTY which we need for the `top` to work correctly.

```sh
$ docker container run -it ubuntu top
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
aafe6b5e13de: Pull complete 
0a2b43a72660: Pull complete 
18bdd1e546d2: Pull complete 
8198342c3e05: Pull complete 
f56970a44fd4: Pull complete 
Digest: sha256:f3a61450ae43896c4332bda5e78b453f4a93179045f20c8181043b26b5e79028
Status: Downloaded newer image for ubuntu:latest
```

The `docker container` command is used to manage containers. The `docker container run` command will run a command in a new container.

The `docker container run` command will result first in a `docker pull` to download the ubuntu image onto your host if the image is not already found on your local repository. Once it is downloaded, it will start the container. The output for the running container should look like this:

```sh
top - 20:32:46 up 3 days, 17:40,  0 users,  load average: 0.00, 0.01, 0.00
Tasks:   1 total,   1 running,   0 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  2046768 total,   173308 free,   117248 used,  1756212 buff/cache
KiB Swap:  1048572 total,  1048572 free,        0 used.  1548356 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND    
    1 root      20   0   36636   3072   2640 R   0.3  0.2   0:00.04 top        

```

`top` is a linux utility that prints the processes on a system and orders them by resource consumption. Notice that there is only a single process in this output: it is the `top` process itself. We don't see other processes from our host in this list because of the PID namespace isolation.

Containers use linux namespaces to provide isolation of system resources from other containers or the host. The PID namespace provides isolation for process IDs. If you run `top` while inside the container, you will notice that it shows the processes within the PID namespace of the container, which is much different than what you can see if you ran `top` on the host.

Even though we are using the `ubuntu` image, it is important to note that our container does not have its own kernel. Its uses the kernel of the host and the `ubuntu` image is used only to provide the file system and tools available on an ubuntu system. 

Note: after the above command finishes, the container is stopped and removed, and is no longer available. Run the `docker ps -a` or `docker container ls -a` command to see all containers,

```sh
$ docker ps -a
```

# Step 2: Clean Up

Completing this lab results in a bunch of running containers on your host. Let's clean these up.


1. First get a list of the containers running using `docker container ls`. 


```sh
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
7c16c698acee        mongo:3.4           "docker-entrypoint.s…"   25 seconds ago      Up 24 seconds       0.0.0.0:8081->27017/tcp   mongo
510c778ca4fb        nginx               "nginx -g 'daemon of…"   5 minutes ago       Up 5 minutes        0.0.0.0:8080->80/tcp      nginx
ef30604ee5f4        ubuntu              "/bin/bash"              20 minutes ago      Up 20 minutes                                 ubuntu
```

2. Next,  run `docker container stop [container id]` or the shorter `docker stop [container id]` for each container in the list. You can also use the names of the containers that you specified before.
```sh
$ docker container stop 7c1 510 ef3
7c1
510
ef3
```

**Note**: You only have to reference enough digits of the ID to be unique. Three digits is almost always enough.


2. Remove the stopped containers

`docker system prune` is a really handy command to clean up your system. It will remove any stopped containers, unused volumes and networks, and dangling images.

```sh
$ docker system prune
WARNING! This will remove:
        - all stopped containers
        - all volumes not used by at least one container
        - all networks not used by at least one container
        - all dangling images
Are you sure you want to continue? [y/N] y
Deleted Containers:
7c16c698aceedf7857f6eaae025c9f89355b387cc3338a22fbb99c4ba55ea543
510c778ca4fb4af130a59c575b45e505cce6360ae63278e1d6b359b07a533e8e
ef30604ee5f4d7d6e34d2b99736e53d6c9e1e53271da65cf8ef7c241d7e8cbe2

Total reclaimed space: 94B
```

# Summary

In this lab, you created your first Ubuntu, Nginx and MongoDB containers.

Key Takeaways
- Containers are composed of linux namespaces and control groups that provide isolation from other containers and the host.
- Because of the isolation properties of containers, you can schedule many containers on a single host without worrying about conflicting dependencies. This makes it easier to run multiple containers on a single host: fully utilizing resources allocated to that host, and ultimately saving some money on server costs.
-  Avoid using unverified content from the Docker Store when developing your own images because these images may contain security vulnerabilities or possibly even malicious software.
- Containers include everything they need to run the processes within them, so there is no need to install additional dependencies directly on your host.

