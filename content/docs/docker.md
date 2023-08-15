---
title:  "docker"
date: 2023-08-15
---

# docker

The kernel features that enabled Docker are:

* *namespaces* allow execution isolation
* *cgroups* (control groups) allow resource limit control
* *capabilities* allow permission management (deny all by default)

Docker needs an *execution driver*. The execution driver is the interface with the kernel features Docker requires, that is *namespaces*, *cgroups*, *capabilities*. *LXC* was the initial execution driver but now *libcontainer* is used, that is owned and controlled by Docker and may be installed in Windows as well. The *Docker engine* consists of a *client* and a *daemon*.


## Starting with docker

### Installing

#### On debian systems
```bash
$ apt-get install -y docker.io  <1>
$ service docker.io status      <2>
```
- <1> Install docker
- <2> Ensure that the daemon is running

#### Alternative installation method
```bash
$ curl https://get.docker.com | sh
```


### Checking installation

```bash
$ docker -v <1>
$ docker version <2>
$ docker info <3>
$ docker run hello-world <4>
```
- <1> the docker client asking the docker daemon for the version
- <2> more detail
- <3> information on the docker installation, images, etc
- <4> runs a sample container


### Running containers

#### Run a container based on ubuntu interactively
```bash
$ docker run -it ubuntu /bin/bash
```
* `run`: starts a new container
* `-it`: make it interactive, assign a tty
* `ubuntu`: base it on the ubuntu image
* `/bin/bash`: inside the container launch the bash process

**CAUTION**: Issuing `exit`, or entering `ctrl+d` will terminate the container.

#### Run a container based on ubuntu interactively
```bash
$ docker run --name <container_name> -p 3306:3306 -d image_name
```

* `--name <container_name>`: name the container
* `-d`: detached
* `-p 3306:3306`: map network port from host to container

#### Example: run a short lived command in detached mode
```bash
$ docker run ubuntu /bin/bash -c "echo 'cool content' > /tmp/cool-file"
```

#### Example: execute a command in a running container
```bash
$ docker exec -it <container_name> /bin/bash      <1>
$ docker exec -d <container_name> touch /tmp/log  <2>
```
- <1> Executes interactively a shell in a running container with name `container_name`
- <2> Creates a file, detached, in a running container


### Running a container from a network port

In the previous example the container was started using the local socket. To launch a container from a network socket, the docker daemon must be stopped and restarted to listen on a network port.

```bash
$ service docker stop
$ docker -H 192.168.1.3:55555 -d &
```

* `-H host:port`: The host and port to start docker
* `-d`: daemonize
* `&`: Get the terminal back

#### Useful commands
```bash
$ netstat -t4lp <1>
$ docker info <2>
$ export DOCKER_HOST="tcp://192.168.1.3:55555" <3>
$ docker info <4>
```
- <1> check that docker listens
- <2> returns an error, since by default docker used the local socket
- <3> set the environment variable
- <4> now it works! (check firewalls if it does not)


## Digging into docker

```bash
$ docker ps <1>
$ docker ps -a <2>
$ ls -l /var/lib/docker/aufs/diff <3>
$ ls -l /var/lib/docker/aufs/diff/$hash <4>
$ docker start $hash <5>
$ docker attach $hash <6>
$ docker pull $image_name <7>
$ docker images $image_name <8>
$ docker stop $hash <9>
```
- <1> show running docker containers
- <2> all docker containers on this machine
- <3> the images
- <4> the directory structure of `$hash`
- <5> starts docker container with id `$hash`
- <6> attach on process with PID1 on the running container with id `$hash`
- <7> pulls `$image_name` from docker hub (latest version)
- <8> shows all images for `$image_name` (all versions)
- <9> stops running container `$hash`

**TIP**: [ctrl] + [p] + [q] exits (detaches from) a container without killing it. Use `docker attach $hash` to reattach.


## Registries and repositories

A Registry (*hub.docker.com*) contains many Repositories (*fedora*, *ubuntu*, *redis*, etc).


## Images

Images are stacked from more elementary images called layers. For example:

* Base Image
* Application (e.g redis)
* Patches

The top layer overrides the previous ones. This is accomplished through union mounts (how to mount multiple file systems on-top of each other). All these layers are mounted as read-only with a top layer as read/write. Only the very top level is writable. The bottom layer is bootfs, that is short lived and no one will ever have to deal with it.

```bash
$ docker images <1>
```
- <1> All locally stored images


## About repositories

The default docker repository is dockerhub. Nevertheless, one might want to use other repositories, and typically not HTTPS ones e.g. in the case when a valid certificate is not available and you want to proxy DockerHub through Nexus.

Provided that one has already configured Nexus accordingly, add the following on the client machines:

#### `/etc/docker/daemon.json`
```json
{
    "insecure-registries": [
        "my-nexus:8082",
        "my-nexus:8083"
    ],
    "disable-legacy-registry": true
}
```

#### Restart docker
```bash
$ sudo systemctl restart docker
```

#### Authenticate (once)
```bash
$ docker login -u admin -p admin123 my-nexus:8082
$ docker login -u admin -p admin123 my-nexus:8083
```

#### Run a container
```bash
$ docker run -it my-nexus:8082/ubuntu /bin/bash
```

#### Resources
* http://codeheaven.io/using-nexus-3-as-your-repository-part-3-docker-images/


## Exporting a docker container

The procedure in high level steps is:

* Save the container to a tar file
* Export it
* Import it

```bash
$ docker commit $hash alias <1>
$ docker images <2>
$ docker history alias <3>
$ docker save -o /tmp/export.tar alias <4>
$ docker load -i /tmp/export.tar <5>
$ docker images <6>
```
- <1> creates a docker images from $hash
- <2> the previously committed image is displayed
- <3> shows the commands that created the image
- <4> exports alias image as tar
- <5> imports it, in another instance
- <6> shows up!

#### When the process inside the container exits, so does the container.
```bash
$ docker run -d ubuntu /bin/bash  -c "ping 8.8.8.8 -c 30" <1>
$ docker top $hash <2>
$ docker run ubuntu:version <3>
```
- <1> ping host from an ubuntu based container
- <2> lets us see top running processes into a container
- <3> be explicit!


## docker run

#### Options
* `-i`: interactive
* `-t`: assign a tty
* `-d`: detached
* `--rm`: remove container when it exits
* `--cpu-shares`: how much CPU (1024 being 100%)
* `--memory`: how much memory to use
* `--restart (no|on-failure|unless-stopped|always)`: restart container on daemon restart according to policy

#### Useful commands on running containers
```bash
$ docker inspect $hash  <1>
$ docker ps -l <2>
$ docker rm $hash <3>
$ docker rm -f $hash <4>
$ docker log $hash <5>
```
- <1> get detailed information on the container
- <2> shows the last container to have run
- <3> remove a stopped container
- <4> remove a running container
- <5> what happens in the container (add -f to follow)


### Entering a container

`docker attach`, attaches to PID1 of a running container. If PID1 is not a shell, then attaching wont be of much use. In that case:

```bash
$ docker inspect $hash | grep Pid <1>
$ nsenter -m -u -n -p -i -t pid /bin/bash <2>
$ docker-enter $hash <3>
$ docker exec -it $hash /bin/bash <4>
```
- <1> find the PID of the running container
- <2> logs into the container
- <3> same thing as above
- <4> same thing as above

**NOTE**: Logging out of this container will not stop it!


## Networking

It is possible to set up docker container networks.

#### Host networking

Start containers with the `--network="host"` option. In that case, `-p` and `-P` options do not take effect, but the container will be able to access all
localhost endpoints, even ones that are started as docker containers.


## Volumes

#### What are volumes for
- To store mutable container data externally
- To share data between host and containers
- To share data between containers

#### Run a container, mapping a host directory into the container
```bash
$ docker run -v $from_host_dir:$to_guest_dir $image_name
```

#### Run a container that creates and mounts a volume
```bash
$ docker run -it -v /tmp ubuntu /bin/bash
```

#### Create a volume and mount it later on
```bash
$ docker volume create --name vol
$ docker run -it -v vol:/data ubuntu /bin/bash
```

#### Create and share a volume
```bash
$ docker create -v /tmp --name datacontainer ubuntu <1>
$ docker run -it --volumes-from datacontainer ubuntu /bin/bash <2>
```
- <1> Creates a volume based on `ubuntu`, named `datacontainer` to be mounted on `/tmp` to containers that will want to use it.
- <2> Runs a container that mounts the previously created volume. Now even after destroying the container, data stored on the volume will remain. Volume can only be mounted on the directory declared during volume creation time, that is `/tmp`.


## Login

Access to registries require login.

```bash
$ docker login docker_registry:port
```

Login in creates or updates a file in `$HOME/.docker/config.json`, that looks like:

```json
{
    "auths": {
        "192.168.56.1:18000": {
            "auth": "YWRtaW46YWRtaW4xMjM="
        }
    }
}
```

**TIP**: The authentication token is base64 encoded and can be decoded like so, `$ echo "token" | base64 -d`

To logout:

```bash
$ docker logout docker_registry:port
```

## Dockerfile

A Dockerfile contains simple instructions to create a docker image.

* Plain text
* Simple format
* Instructions to build an image
* All other subdirectories will be included in the build

```bash
$ docker build <1>
$ docker build -t mytag:version . <2>
```
- <1> builds an image with instructions from the Dockerfile
- <2> tag and build from local directory

#### Sample Dockerfile
```dockerfile
# comment
FROM image:version
MAINTAINER email
RUN runCommand1  # creates a new layer!
RUN runCommand2  # creates a new layer!
CMD ["echo", "Hello", "World"]
```

#### Other commands
* `RUN`: Executes only during build time
* `CMD`: Executes only during run time and can only be one
* `EXPOSE`: Exposes a network port
* `ENTRYPOINT`: The command to run, cannot be overridden and anything appended from the command line is passed as argument

**NOTE**: `ENTRYPOINT` specifies a command that will always be executed when the container starts. `CMD` specifies the default arguments that will be fed to the `ENTRYPOINT` which can be overwritten from command line when docker container runs.

**TIP**: `ENTRYPOINT` can actually be overridden by the `--entrypoint` command line argument

#### ENTRYPOINT vs CMD

Entrypoint is the top level command that is executed when a container starts. 
Even when not defined, the default `sh -c` entrypoint is used. 
When running a container with an entrypoint and additional parameters, 
those parameters are passed to the entrypoint:

```bash
$ docker run <image> arg1 arg2
```
In the command above, the entrypoint of the image is used and `arg1`, `arg2` are passed as parameters. 
If an entrypoint was not defined, then `arg1` would have been the command to execute 
and `arg2` would have been a param to the command.
Obvioulsy, if `arg1` was not an executable, the docker command would have resulted to an error.

If `ENTRYPOINT` and `CMD` are defined in the dockerfile, and no parameters in the docker run,
then the `CMD` array is passed to the entrypoint as default parameters. 
If there are parameters in the command line, they override the `CMD` array from dockerfile.


## Futher commands

```bash
$ docker tag $hash repo/$image_name:version <1>
$ docker push repo/$image_name:version <2>
$ docker rmi $image_name <3>
$ docker system prune --volumes <4>
```
- <1> tags `$hash`
- <2> pushes `$image_name` to repository
- <3> removes `$image_name`
- <4> remove all stopped containers, all dangling images all unused networks and volumes


## Running Oracle 11g XE in a container

```bash
$ docker pull wnameless/oracle-xe-11g
$ docker run -d -p 1522:22 -p 1521:1521 wnameless/oracle-xe-11g
```

#### Connect to database

hostname:: localhost
port:: 1521
sid:: xe
username:: system
password:: oracle

#### Login with SSH (password: admin)
```bash
$ ssh root@localhost -p 1522
```


## Recipies

### Bulk remove containers

```bash
$ docker rm $(docker ps -a -q -f "ancestor=openjdk:8-alpine")
```


## Resources
* https://www.digitalocean.com/community/tutorials/how-to-work-with-docker-data-volumes-on-ubuntu-14-04[How To Work with Docker Data Volumes on Ubuntu]
* http://container-solutions.com/understanding-volumes-docker/[Understanding Volumes in Docker]
