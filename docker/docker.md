# Docker

Before going through this tutorial on how to install docker and dockerize an app, please read this [intro to docker](https://docs.docker.com/introduction/understanding-docker/). It's a very short read and does a very good job of explaining the what, the why, and the how, of docker.

## Installation

1) Download the latest version of the [Boot2Docker](https://github.com/boot2docker/osx-installer/releases/) Mac OSX client that runs a lightweight VM to run docker on.

2) From your terminal, run the following commands:

A) Initialize the client

	$ boot2docker init

B) Start the Docker daemon

	$ boot2docker start

If you get the follwing message:

	To connect the Docker client to the Docker daemon, please set:
    export DOCKER_HOST=tcp://192.168.59.103:2376
    export DOCKER_CERT_PATH=/Users/garciadanny/.boot2docker/certs/boot2docker-vm
    export DOCKER_TLS_VERIFY=1

Then set those environment variables on your local machine.

The latest version of Boot2Docker sets up two network adaptors. One allows the Boot2Docker virtual machine to download images and files from the internet via NAT;
the other exposes the Docker container’s port to the host-only network.

You may need to open a new terminal window and re-run the command from step B.

C) Boot2Docker runs Docker with TLS enabled. Because of this, we need to run the following command to generate certificates and store them in the VM.

	$(boot2docker shellinit)

**Side Note:** If you get a `title 8 parse error` it's not related to docker but related to `myzsh`. It won't affect the installation (but you may wanna upgrade myzsh... I didn't bother).

4) Confirm boot2docker installer ran successfully:

	$ boot2docker ssh

	                   ##        .
                  ## ## ##       ==
               ## ## ## ##      ===
           /""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
           \______ o          __/
             \    \        __/
              \____\______/
	 _                 _   ____     _            _
	| |__   ___   ___ | |_|___ \ __| | ___   ___| | _____ _ __
	| '_ \ / _ \ / _ \| __| __) / _` |/ _ \ / __| |/ / _ \ '__|
	| |_) | (_) | (_) | |_ / __/ (_| | (_) | (__|   <  __/ |
	|_.__/ \___/ \___/ \__|_____\__,_|\___/ \___|_|\_\___|_|
	Boot2Docker version 1.3.2, build master : 495c19a - Mon Nov 24 20:40:58 UTC 2014
	Docker version 1.3.2, build 39fa2fa
	docker@boot2docker:~$

If you see the above output, it worked!

## Port Forwarding

##### Alternative Solution 1

To access servers running in docker containers via our local machines, we can forward all of the ports in docker's port range from the VM to localhost.

1)

	$ boot2docker stop

2)

	for i in {49000..49900}; do \
		VBoxManage modifyvm "boot2docker-vm" --natpf1 "tcp-port$i,tcp,,$i,,$i";
		VBoxManage modifyvm "boot2docker-vm" --natpf1 "udp-port$i,udp,,$i,,$i";
	done

##### Alternative Solution 2

For some reason that I can't yet figure out, the above solution worked only temporarily for me. Another solution is as follows:

On your local machine add an entry to the `/etc/hosts` file that points to the `boot2docker ip` address

	$ echo $(boot2docker ip) dockerhost | sudo tee -a /etc/hosts

Now when you have a server running in a docker container, you can access it from your brower by using `dockerhost`:

	http://dockerhost:PORT

##### Alternative Solution 3

If you don't want to do any of the fancy things above, you can always access a running containter like this:

	http://boot2docker.host:PORT


## Dockerizing a node app that uses redis.


1) Create a `Dockerfile` in the root of the directory

	# Pull base image.
	FROM node:latest

	# Bundle app source
	# This creates a directory (mnt/app) in your container's file system and puts your app in this directory.
	# It also makes this directory the working directory.
	ADD . /mnt/app
	WORKDIR /mnt/app

	# Install app dependencies
	# This command assumes you have a make file with a buil command that installs your app's dependencies.
	RUN make build

	# Your app binds to port 8080 so you'll use the EXPOSE instruction to have it mapped by the docker daemon
	EXPOSE  8080

	# Last but not least, define the command to run your app
	# This command assumes you have a make file with a start command that starts your app.
	CMD ["make", "start"]

2) Assuming you have a config file in your app that establishes settings for redis, ensure that the `host` for redis is set to `redis` and not `127.0.0.1`. This is because when we run the container hosting redis, it'll create a named host called `redis`.

	session:
	  store: redis
	  host: redis
	  port: 6379
	  secret: some-secret

3) Build a docker image for your node app

	$ docker build -t <your username>/<whatever-you-wanna-name-the-image> /Users/path/to/repo

4) Pull down the redis image from the Docker registry

	$ docker pull redis

5) Run the redis image in detached state (`-d`)

	$ docker run --name <whatever-you-wanna-name-the-container> -d redis

6) Run the image container your node app

	$ docker run -p 8080:8080 --name <whatever-you-wanna-name-this-container> --link <whatever-you-named-the-redis-container>:redis <your username>/<whatever-you-named-the-node-image>

Because we didn't add the `-d` option in step 6, the server won't run in detached state. If you go to your browser and hit `http://dockerhost:8080`, you'll see your app's server logs flow down your terminal like the [waters of lake minnetonka](https://www.youtube.com/watch?v=TnWdnD0oyBk).


## Notes (commands)

##### Setting Environment Variables

You need the -it and -e options to set environment variables

	$ docker run -it -e FOO=bar ubuntu bash

##### Accessing a Container

Run `bash` at the end to get into the container

	$ docker run -it -p 8080:8080 --link <whatever-you-named-your-redis-container>:redis <your username>/<whatever-you-named-your-node-container> bash

## Things I learned

In the docker file, each instruction runs on a seperate process unless you explicitly tell it not to. Think of it as opening a new shell window before running each instruction. Docker uses an union file system. Each instruction you run from the docker file is it's own layer. So for example, if you change the version of an app in the dockerfile, only that layer gets updated rather than having to rebuild the whole image. Individual layers are added/updated.

### Taken from docker docs (say 'docker docs' 5 times really fast)

What happens when you run a container? Either by using the docker binary or via the API, the Docker client tells the Docker daemon to run a container.

	$ sudo docker run -i -t ubuntu /bin/bash

Let's break down this command. The Docker client is launched using the docker binary with the run option telling it to launch a new container. The bare minimum the Docker client needs to tell the Docker daemon to run the container is:

* What Docker image to build the container from, here ubuntu, a base Ubuntu image;

* The command you want to run inside the container when it is launched, here /bin/bash, to start the Bash shell inside the new container.
So what happens under the hood when we run this command?

In order, Docker does the following:

1) Pulls the ubuntu image: Docker checks for the presence of the ubuntu image and, if it doesn't exist locally on the host, then Docker downloads it from Docker Hub. If the )image already exists, then Docker uses it for the new container.

2) Creates a new container: Once Docker has the image, it uses it to create a container.

3) Allocates a filesystem and mounts a read-write layer: The container is created in the file system and a read-write layer is added to the image.

4) Allocates a network / bridge interface: Creates a network interface that allows the Docker container to talk to the local host.

5) Sets up an IP address: Finds and attaches an available IP address from a pool.

6) Executes a process that you specify: Runs your application, and;

7) Captures and provides application output: Connects and logs standard input, outputs and errors for you to see how your application is running.

### Underlying technologies

1) namespaces

2) Control groups (cgroups)

3) Union file systems (UnionFS)

4) Container format (`libcontainer`, LXC)

### Commiting

After you make changes (by running a command inside a container), you probably want to save those changes. This will enable you to start from this point later. With Docker, the process of saving the state is called committing. Commit basically saves the difference between the old image and the new state.

	$ docker commit ID-of-container <image or repository name i.e.(learn/ping)>